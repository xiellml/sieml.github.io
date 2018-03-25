---
title: Fragment Transactions & Activity State Loss(译文)
date: 2018-01-16 20:58:00
tags: translation
---


以下堆栈追踪和异常信息自从Honeycomb's初期版本以来就困扰着StackOverflow:

```java
java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState
    at android.support.v4.app.FragmentManagerImpl.checkStateLoss(FragmentManager.java:1341)
    at android.support.v4.app.FragmentManagerImpl.enqueueAction(FragmentManager.java:1352)
    at android.support.v4.app.BackStackRecord.commitInternal(BackStackRecord.java:595)
    at android.support.v4.app.BackStackRecord.commit(BackStackRecord.java:574)
```
这篇文章将会解释为什么以及何时抛出异常, 然后总结一些建议有助于确保你的应用程序不再奔溃.

## **为什么抛出异常?**

这个异常被抛出是因为你尝试在Activity的状态被save之后, commit一个FragmentTransaction, 才导致一个被称为Activity状态丢失的现象.不管怎样, 在我们详细讨论这个实际的含义之前, 让我们首先来看看在调用onSaveInstanceState()时在背后发生了什么. 正如我在上一篇关于 [Binders & Death Recipients](https://www.androiddesignpatterns.com/2013/08/binders-death-recipients.html)的文章所讨论的那样, Android应用在Android runtime环境中几乎无法控制自己的命运(译者注: Android应用开发似乎这是通病, 但收紧系统利于降低开发成本和开发者快速面向功能实现而非分心去应付庞杂的事情). Android系统有权在任何时候终结进程以释放内存,而后台Activity可能因此而几乎没任何警告而被杀死.

为了确保这种对用户来讲是隐藏的, 有时不稳定的行为,Framework层在Activity变得易受破坏前, 会给每个Activity一个机会调用它的onSaveInstanceState()方法去保存它的状态. 当保存的状态稍后恢复时, 无论Activity是否被系统杀死,用户会感觉在前台Activity和后台Activity之间是无缝切换的,当framework层调onSaveInstanceState()时, 它传递一个含Bundle对象的方法来使Activity保存它的状态, 记录内部Dialog, Fragment和View的状态.当方法返回时, 系统通过Binder接口包装Bundle对象到System Server进程, 在那里被安全地存放.在系统稍后决定重新创建Activity时, 会发送那个相同的Bundle对象到应用程序, 以便它用来恢复Activity过去的状态.

那么为什么会抛出异常呢? 好吧, 问题源于这样一个事实, 即这些Bundle对象都代表了onSaveInstanceState()被调用时Activity的一张快照, 仅此而已.这意味着当你在调用onSaveInstanceState()之后所有的FragmentTransaction# commit(), transaction事务操作没有记住因此它从来没有被记录为Activity状态的一部分.从用户的角度看, transaction操作将丢失, 导致意外的UI状态丢失. 为了用户体验, Android不惜一切代价避免状态丢失, 即任何时候要发生意外UI丢失时就会简单抛出一个IllegalStateException异常.

### **什么时候抛出异常?**

如果您之前遇到过这个异常，您可能已经注意到，在不同的平台版本中，引发的时机略有不同。例如，您可能发现较旧的设备倾向于少抛出异常，或者使用第三方支持库时应用程序比使用官方框架类时更容易崩溃。这些微小的不一致导致许多人认为支持库是错误的，不能被信任。 但是，这些假设通常是不正确的。

之所以存在这些微小的不一致之处，是因为对Honeycomb中的Activity生命周期的重大改变。在Honeycomb之前，活动在被暂停之前不会被视为killable，这意味着onSaveInstanceState（）在onPause（）之前被立即调用。然而，从Honeycomb开始，活动只有在停止(指stop, 不是指暂停pause)之后才被认为是可杀死的，这意味着onSaveInstanceState（）现在将在onStop（）之前而不是在onPause（）之前被调用。这些差异总结在下表中：

|                                          | **pre-Honeycomb** | **post-Honeycomb** |
| ---------------------------------------- | ----------------- | ------------------ |
| Activities can be killed before onPause()? | NO                | NO                 |
| Activities can be killed before onStop()? | YES               | NO                 |
| onSaveInstanceState(Bundle) is guaranteed to be called before… | onPause()         | onStop()           |


由于对Activity生命周期所做的轻微更改，支持库有时需要根据平台版本来改变其行为。例如，在Honeycomb 设备及其以上版本中，每次在onSaveInstanceState（）之后调用commit（）警告开发者状态丢失已经发生。然而，每次发生这种情况时都抛出一个异常，对于Honeycomb 之前版本设备来说，它们的限制性太强了，因为它们的onSaveInstanceState（）方法在Activity的生命周期中调用的时间要早得多，因此更容易受到意外的状态丢失的威胁。Android团队不得不做出妥协：为了更好地与旧版本的平台进行交互，较旧的设备必须忍受onPause（）和onStop（）之间可能导致的意外状态丢失。下表总结了两种平台上支持库的行为：

|                                         | **pre-Honeycomb** | **post-Honeycomb** |
| --------------------------------------- | ----------------- | ------------------ |
| commit() before onPause()               | OK                | OK                 |
| commit() between onPause() and onStop() | STATE LOSS        | OK                 |
| commit() after onStop()                 | EXCEPTION         | EXCEPTION          |

### **怎么避免这个异常?**

一旦你明白了实际发生的事情，避免活动状态的丢失就变得容易了很多。如果你已经在这篇文章中读了这么多，希望你能更好地理解支持库的工作原理，以及为什么避免应用程序丢失状态是非常重要的。如果你找到这篇文章是为了寻找一个快速修复，那么当你在你的应用程序中使用FragmentTransactions的时候，记注下面的一些建议：

- **在Activity生命周期方法中提交事务时要小心。** 大多数应用程序只会在第一次调用onCreate（）时和/或响应用户输入时才提交事务，因此不会面临任何问题。但是，随着事务开始冒险进入其他Activity生命周期方法，如onActivityResult（），onStart（）和onResume（），事情会变得有点棘手。例如，您不应该在FragmentActivity＃onResume（）方法内提交事务，因为在某些情况下，可以在恢复活动状态之前调用该方法（有关详细信息，请参阅[文档](http://developer.android.com/reference/android/support/v4/app/FragmentActivity.html#onResume()) ）。如果您的应用程序需要在除onCreate（）之外的Activity生命周期方法中提交事务，则在FragmentActivity＃onResumeFragments（）或Activity＃onPostResume（）中执行。这两种方法保证在Activity恢复到原来的状态之后被调用，从而避免了状态一起丢失的可能性。 （作为如何做到这一点的一个例子，请查看我对这个[StackOverflow question](http://stackoverflow.com/q/16265733/844882) 的回答，了解如何提交FragmentTransactions以响应对Activity＃onActivityResult（）方法的调用）。
- **避免在异步回调方法中执行事务。 **  这包括常用的方法，如AsyncTask＃onPostExecute（）和LoaderManager.LoaderCallbacks＃onLoadFinished（）。 在这些方法中执行事务的问题是，在调用它们时，他们不了解Activity生命周期的当前状态。 例如，考虑以下一系列事件：

1. 一个活动执行一个AsyncTask。

2. 用户按下“Home”键，导致活动的onSaveInstanceState（）和onStop（）方法被调用。

3. AsyncTask完成并调用onPostExecute（），但不知道Activity已经停止。

4. FragmentTransaction在onPostExecute（）方法内被提交，导致异常被抛出。

通常，在这些情况下避免异常的最好方法是简单地避免在异步回调方法中一起提交事务。 Google工程师似乎也同意这一观点。根据Android开发者小组的这篇文章，Android团队认为，从异步回调方法中提交FragmentTransaction导致的用户界面的主要变化可能会对用户体验造成不利影响。如果您的应用程序需要在这些回调方法中执行事务，并且没有简单的方法来保证在onSaveInstanceState（）之后不会调用回调，你可能不得不求助于使用commitAllowingStateLoss（）并处理可能发生的状态丢失。 （另请参阅这两个StackOverflow帖子，了解更多提示，[这里](http://stackoverflow.com/q/8040280/844882) and [这里](http://stackoverflow.com/q/7992496/844882)）。


- **commitAllowingStateLoss（）的使用只作为最后的手段。 **  调用commit（）和commitAllowingStateLoss（）之间的唯一区别是，如果发生状态丢失，后者不会抛出异常。
   通常情况下你不会想使用这种方法，因为这意味着有可能出现状态损失。 当然，更好的解决方案是编程确保在保存Activity状态之前调用commit（），因为这会带来更好的用户体验。除非无法避免状态丢失的可能性，否则不应使用commitAllowingStateLoss（）。

希望这些建议能帮助您解决过去遇到此异常的任何问题。 如果您仍然遇到麻烦，请在[StackOverflow](http://stackoverflow.com/)发布问题，并在下面的评论中发布链接，我可以看看。:)

一如既往，感谢您的阅读, 如果您有任何问题留下评论。 如果您觉得很棒的话，请不要忘记+1这个博客，并在Google+上分享这篇文章！

译者注: 此文是翻译的[这篇文](https://www.androiddesignpatterns.com/2013/08/fragment-transaction-commit-state-loss.html), 可以说是最先总结Fragment事务操作异常的大神, 国内也很多同行翻译, 但是大都是机翻有错误我很强迫症. 故本文借助机翻并校正了两三个小时(笑哭..英文太烂), 达到了我满意的程度. 如果你看完了文章还是不懂怎么操作, 我这里提示下, 主要是考虑Activity生命周期!!! 所以你的事务操作必须处于安全状态, 然后再去操作. 如果还是不知代码如何写, email联系我siesielee@gmail.com