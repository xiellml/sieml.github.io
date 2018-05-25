---
title: (Translation) A Small Leak Will Sink a Great Ship
date: 2018-01-21 21:27:02
tags: translation
---

**小泄露大翻船**

Android Lollipop之前版本, 弹出dialog可能导致Android应用内存泄露

由 [Pierre-Yves Ricau](https://twitter.com/Piwai) 原创

这篇文章开始于在我构建LeakCanary作为内部邮件专题时. 我发现了一个奇怪的内存泄露, 然后开始深入挖掘找出发生了什么事情。

**艺术大师**

我得到来自 [LeakCanary](http://squ.re/leakcanary)的内存泄漏报告:

```java
* GC ROOT thread com.squareup.picasso.Dispatcher.DispatcherThread.<Java Local>
* references android.os.Message.obj
* references com.example.MyActivity$MyDialogClickListener.this$0
* leaks com.example.MyActivity.MainActivity instance
```

简而言之：[Picasso](https://github.com/square/picasso) 线程将一个Message实例作为一个局部变量放在堆栈上。该Message 引用了一个DialogInterface.OnClickListener，该接口本身引用了一个被销毁的Activity。局部变量通常是短暂存活的，因为它们只存在于堆栈中。 在线程上调用方法时，会分配一个堆栈帧。当方法返回时，该堆栈帧被清除，并且所有的局部变量都被垃圾收集。 如果一个局部变量导致泄漏，那么通常意味着一个线程正在循环或阻塞，同时还持有对Message实例的引用。

[Dimitris](https://github.com/square/picasso/graphs/contributors)和我看了Picasso 的源代码。

Dispatcher.DispatcherThread 是一个简单的HandlerThread

```java
static class DispatcherThread extends HandlerThread {
  DispatcherThread() {
    super(Utils.THREAD_PREFIX + DISPATCHER_THREAD_NAME, THREAD_PRIORITY_BACKGROUND);
  }
}
```

这个线程通过一个用一种很常规的方式实现的Handler接收Message

```java
private static class DispatcherHandler extends Handler {
  private final Dispatcher dispatcher;
  public DispatcherHandler(Looper looper, Dispatcher dispatcher) {
    super(looper);
    this.dispatcher = dispatcher;
  }
  @Override public void handleMessage(final Message msg) {
    switch (msg.what) {
      case REQUEST_SUBMIT: {
        Action action = (Action) msg.obj;
        dispatcher.performSubmit(action);
        break;
      }
      // ... handles other types of messages
    }
  }
}
```

这是条死路 。 Dispatcher.DispatcherHandler.handleMessage（）中没有明显的bug，它以某种方式通过局部变量保持对消息的引用。



**队列提示**

最终，更多的内存泄漏报告出现了。不仅仅是Picasso。 我们从各种类型的线程中得到了局部变量泄漏，并且总牵扯到一个dialog点击监听器。泄漏的线程有一个共同的特点：它们都是工作线程，通过某种阻塞队列接收然后工作。我们来看看HandlerThread是如何工作的：

```java
for (;;) {
    Message msg = queue.next(); // might block
    if (msg == null) {
        return;
    }
    msg.target.dispatchMessage(msg);
    msg.recycleUnchecked();
}
```

百分百确定有一个引用消息的本地变量。 但它可能存活的非常短暂, 只要一循环迭代就会被清除。我们尝试通过编写一个带有阻塞队列的裸露线程，然后只发送一条消息去重现它: 

```java
static class MyMessage {
  final String message;
  MyMessage(String message) {
    this.message = message;
  }
}
static void startThread() {
  final BlockingQueue<MyMessage> queue = new LinkedBlockingQueue<>();
  MyMessage message = new MyMessage("Hello Leaking World");
  queue.offer(message);
  new Thread() {
    @Override public void run() {
      try {
        loop(queue);
      } catch (InterruptedException e) {
        throw new RuntimeException(e);
      }
    }
  }.start();
}
static void loop(BlockingQueue<MyMessage> queue) throws InterruptedException {
  while (true) {
    MyMessage message = queue.take();
    System.out.println("Received: " + message);
  }
}
```



一旦消息在日志中打印, 我们希望MyMessage实例被垃圾回收.

可LeakCanary 却不赞同我的设想:

```java
* GC ROOT thread com.example.MyActivity$2.<Java Local> (named 'Thread-110')
* leaks com.example.MyActivity$MyMessage instance
```

只要我们发送一个新的消息到队列中，之前的消息就被垃圾回收了，而这个新消息就马上泄漏。

在VM中，每个堆栈帧都有一组局部变量。垃圾收集器是保守的：如果有一个可能活着的引用，它将不会收集它。

迭代之后，局部变量不再可达，但它仍然保留对消息的引用。 只要局部变量不再可达，解释器/ JIT就该人工地清空引用，但现在它依旧保持引用存活，并假设不会造成任何危害。

为了确认这个理论，我们手动将引用置为null, 并再次打印，以防置空不起作用：

```java
static void loop(BlockingQueue<MyMessage> queue) throws InterruptedException {
  while (true) {
    MyMessage message = queue.take();
    System.out.println("Received: " + message);
    message = null;
    System.out.println("Now null: " + message);
  }
}
```

在测试上述更改时，我们看到MyMessage实例在message设置为null后立即被垃圾回收。 我们关于虚拟机VM忽略“*局部***Message***变量* ”理论似乎被证实了。

由于这个泄露可以在各种线程和队列实现中重现，所以我们现在确定这是一个虚拟机的bug。 值得注意的是，我们只能在Dalvik VM上重现，而不能在ART VM或JVM上重现。

### **回收池里的Message**

我们发现一个bug，但它会造成巨大的内存泄漏吗？ 让我们再看看我们原来的泄漏：

```java
* GC ROOT thread com.squareup.picasso.Dispatcher.DispatcherThread.<Java Local>
* references android.os.Message.obj
* references com.example.MyActivity$MyDialogClickListener.this$0
* leaks com.example.MyActivity.MainActivity instance
```

在发送给Picasso dispatcher thread的消息中，我们从未将Message.obj设置为DialogInterface.OnClickListener。 它怎么会停在那里？

而且，message被处理后，它立即被回收然后Message.obj被置为空。 只有这样，HandlerThread才会等待下一条消息，临时泄漏上一个message：

```java
for (;;) {
    Message msg = queue.next(); // might block
    if (msg == null) {
        return;
    }
    msg.target.dispatchMessage(msg);
    msg.recycleUnchecked();
}
```

就这一点而言，我们知道泄露的message已经被回收，因此不会保留以前的内容。

一旦回收，message回到静态池中：

```java
void recycleUnchecked() {
    // Mark the message as in use while it remains in the recycled object pool.
    // Clear out all other details.
    flags = FLAG_IN_USE;
    what = 0;
    arg1 = 0;
    arg2 = 0;
    obj = null;
    replyTo = null;
    sendingUid = -1;
    when = 0;
    target = null;
    callback = null;
    data = null;
    synchronized (sPoolSync) {
        if (sPoolSize < MAX_POOL_SIZE) {
            next = sPool;
            sPool = this;
            sPoolSize++;
        }
    }
}**
```

我们泄漏一个空Message，它可能会被重复使用，并填充不同的内容。 这个Message总是以相同的方式使用：从池中脱离，并填充内容，再放到MessageQueue，然后处理，最终又回收并放回静态池中。

因此，它应该从不持续保留它的内容。 那为什么我们总是泄露DialogInterface.OnClickListener实例？我们换个方向。

### **Alert Dialogs**

我们来创建一个简单的提醒对话框：

```java
new AlertDialog.Builder(this)
    .setPositiveButton("Baguette", new DialogInterface.OnClickListener() {
      @Override public void onClick(DialogInterface dialog, int which) {
        MyActivity.this.makeBread();
      }
    })
    .show();**
```

请注意，点击监听器具有对Activity的引用。 此匿名类转换后为以下代码：

```java
// First anonymous class of MyActivity.
class MyActivity$0 implements DialogInterface.OnClickListener {
  final MyActivity this$0;
  MyActivity$0(MyActivity this$0) {
    this.this$0 = this$0;
  }
  @Override public void onClick(DialogInterface dialog, int which) {
    this$0.makeBread();
  }
}
new AlertDialog.Builder(this)
    .setPositiveButton("Baguette", new MyActivity$0(this));
    .show();**
```

AlertDialog在内部将工作委托给AlertController：



```java
/**
 * Sets a click listener or a message to be sent when the button is clicked.
 * You only need to pass one of {@code listener} or {@code msg}.
 */
public void setButton(int whichButton, CharSequence text,
        DialogInterface.OnClickListener listener, Message msg) {
    if (msg == null && listener != null) {
        msg = mHandler.obtainMessage(whichButton, listener);
    }
    switch (whichButton) {
        case DialogInterface.BUTTON_POSITIVE:
            mButtonPositiveText = text;
            mButtonPositiveMessage = msg;
            break;
        case DialogInterface.BUTTON_NEGATIVE:
            mButtonNegativeText = text;
            mButtonNegativeMessage = msg;
            break;
        case DialogInterface.BUTTON_NEUTRAL:
            mButtonNeutralText = text;
            mButtonNeutralMessage = msg;
            break;
    }
}
```

所以OnClickListener被封装在一个Message中，并设置为AlertController.mButtonPositiveMessage。 让我们来看看这个消息何时被使用：

```java
private final View.OnClickListener mButtonHandler = new View.OnClickListener() {
    @Override public void onClick(View v) {
        final Message m;
        if (v == mButtonPositive && mButtonPositiveMessage != null) {
            m = Message.obtain(mButtonPositiveMessage);
        } else if (v == mButtonNegative && mButtonNegativeMessage != null) {
            m = Message.obtain(mButtonNegativeMessage);
        } else if (v == mButtonNeutral && mButtonNeutralMessage != null) {
            m = Message.obtain(mButtonNeutralMessage);
        } else {
            m = null;
        }
        if (m != null) {
            m.sendToTarget();
        }
        // Post a message so we dismiss after the above handlers are executed.
        mHandler.obtainMessage(ButtonHandler.MSG_DISMISS_DIALOG, mDialogInterface)
                .sendToTarget();
    }
};
```

注意这句: Message.obtain(mButtonPositiveMessage).

该消息被克隆，并**发送其副本**。 这意味着原始消息不会被发送，因此**不会被回收**。 所以它永远保持其内容，直到垃圾收集。

现在我们假设由于HandlerThread局部引用，副本消息对象在从循环池中复用之前的message。 直到该对话框最后被垃圾收集，所以会释放对mButtonPositiveMessage的消息引用。

但是，因为原消息对象此刻泄漏了，所以肯定它没有被回收。 同理, 内容、OnClickListener以及Activity都没有被回收。

### **吸口烟**

我们能证明我们的理论吗？

我们需要发送一个消息给HandlerThread，让它消耗回收，并不发送任何其他消息到该线程，以便泄漏最后一条message。 然后，我们需要显示一个按钮的对话框，并希望这个对话框将从池中获得相同的message。 这很可能会发生，因为一旦循环利用，有个Message就成为池中的第一个消息。

```java
HandlerThread background = new HandlerThread("BackgroundThread");
background.start();
Handler backgroundhandler = new Handler(background.getLooper());
final DialogInterface.OnClickListener clickListener = new DialogInterface.OnClickListener() {
  @Override public void onClick(DialogInterface dialog, int which) {
    MyActivity.this.makeCroissants();
  }
};
backgroundhandler.post(new Runnable() {
  @Override public void run() {
    runOnUiThread(new Runnable() {
      @Override public void run() {
        new AlertDialog.Builder(MyActivity.this) //
            .setPositiveButton("Baguette", clickListener) //
            .show();
      }
    });
  }
});
```

如果我们运行上面的代码，然后旋转屏幕来销毁Activity，Activity很可能会泄漏。

LeakCanary准确地检测到泄漏：

```java
* GC ROOT thread android.os.HandlerThread.<Java Local> (named 'BackgroundThread')
* references android.os.Message.obj
* references com.example.MyActivity$1.this$0 (anonymous class implements android.content.DialogInterface$OnClickListener)
* leaks com.example.MyActivity instance
```

至此我们已经正确重现了它，让我们看看我们可以做些什么来解决了。

### **平台修复**

仅支持使用ART VM的设备，即Android 5+。再也没有bugs了！ 当然，也不会有更多的用户。

### **放弃修复**

你也可以假设这些泄漏的影响是有限的，你有更棒的事情要做，或者是去解决更简单的泄漏。 LeakCanary [默认](#L112)忽略所有Message泄露。 但要注意，一个Activity 在内存中保存了整个视图层次结构，它可以保留好几M。

### **内部修复**

确保您的DialogInterface.OnClickListener实例不持有对Ativity实例的强引用，例如，在对话窗口detached后清除对监听器listener的引用。 这里是一个包装类，方便清除listener引用: 

```
public final class DetachableClickListener implements DialogInterface.OnClickListener {
```

```
  public static DetachableClickListener wrap(DialogInterface.OnClickListener delegate) {
    return new DetachableClickListener(delegate);
  }
```

```
  private DialogInterface.OnClickListener delegateOrNull;
```

```java
  private DetachableClickListener(DialogInterface.OnClickListener delegate) {
    this.delegateOrNull = delegate;
  }
  @Override public void onClick(DialogInterface dialog, int which) {
    if (delegateOrNull != null) {
      delegateOrNull.onClick(dialog, which);
    }
  }
  public void clearOnDetach(Dialog dialog) {
    dialog.getWindow()
        .getDecorView()
        .getViewTreeObserver()
        .addOnWindowAttachListener(new OnWindowAttachListener() {
          @Override public void onWindowAttached() { }
          @Override public void onWindowDetached() {
            delegateOrNull = null;
          }
        });
  }
}
```

然后你可以包装所有的OnClickListener实例：

```java
DetachableClickListener clickListener = wrap(new DialogInterface.OnClickListener() {
  @Override public void onClick(DialogInterface dialog, int which) {
    MyActivity.this.makeCroissants();
  }
});
AlertDialog dialog = new AlertDialog.Builder(this) //
    .setPositiveButton("Baguette", clickListener) //
    .create();
clickListener.clearOnDetach(dialog);
dialog.show();
```

### **管道修复**

定期刷新工作线程：当HandlerThread**空闲时**发送一条空的Message，以确保没有消息泄漏很长时间。

```java
static void flushStackLocalLeaks(Looper looper) {
  final Handler handler = new Handler(looper);
  handler.post(new Runnable() {
    @Override public void run() {
      Looper.myQueue().addIdleHandler(new MessageQueue.IdleHandler() {
        @Override public boolean queueIdle() {
          handler.sendMessageDelayed(handler.obtainMessage(), 1000);
          return true;
        }
      });
    }
  });
}
```

这对第三方库很有用，因为你无法控制开发者要用dialogs做什么。 我们在Picasso中使用上了它，在其他类型的工作线程也用了[此类修复](https://github.com/square/picasso/pull/932)。

### **总结**

正如我们所看到的，一个微妙的和意想不到的虚拟机行为会发生一个小泄漏导致大量的内存崩溃，最终导致应用程序崩溃，出现OutOfMemoryError内存溢出。 一个小小的泄漏让一艘大船沉没了。

非常感谢[Josh Humphries](https://twitter.com/jhumphries_sq), [Jesse Wilson](https://twitter.com/jessewilson), [Manik Surtani](https://twitter.com/maniksurtani), 和 [Wouter Coekaerts](https://twitter.com/WouterCoekaerts) 在我们的内部电子邮件专题中提供帮助。

 

译者注: 原文在[这里](https://medium.com/square-corner-blog/a-small-leak-will-sink-a-great-ship-efbae00f9a0f)