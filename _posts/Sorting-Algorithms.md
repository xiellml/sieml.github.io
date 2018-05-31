---
title: (Repost) Sorting Algorithms
date: 2018-03-23 12:13:53
tags:
---

#### 记忆:

n*n: 选择->插入->冒泡* nlog(2)n: 快速->归并->堆排

(shell排->基排->桶排)

#### 时间复杂度:

[![时间复杂度](http://paliysha.site/2018/03/23/Sorting-Algorithm/sort_time_complexity.jpg)](http://paliysha.site/2018/03/23/Sorting-Algorithm/sort_time_complexity.jpg)

**注：**

1、归并排序每次递归都要用到一个辅助表，长度与待排序的表长度相同，虽然递归次数是O(log2n)，但每次递归都会释放掉所占的辅助空间，

2、快速排序空间复杂度只是在通常情况下才为O(log2n)，如果是最坏情况的话，很显然就要O(n)的空间了。当然，可以通过随机化选择pivot来将空间复杂度降低到O(log2n)。

**相关概念：**

1、时间复杂度

 时间复杂度可以认为是对排序数据的总的操作次数。反映当n变化时，操作次数呈现什么规律。

 常见的时间复杂度有：常数阶O(1),对数阶O(log2n),线性阶O(n), 线性对数阶O(nlog2n),平方阶O(n2)

 时间复杂度O(1)：算法中语句执行次数为一个常数，则时间复杂度为O(1),

2、空间复杂度

 空间复杂度是指算法在计算机内执行时所需存储空间的度量，它也是问题规模n的函数

 空间复杂度O(1)：当一个算法的空间复杂度为一个常量，即不随被处理数据量n的大小而改变时，可表示为O(1)

 空间复杂度O(log2N)：当一个算法的空间复杂度与以2为底的n的对数成正比时，可表示为O(log2n)

 ax=N，则x=logaN，

 空间复杂度O(n)：当一个算法的空间复杂度与n成线性比例关系时，可表示为0(n).

引自[AmyAlisa](https://www.cnblogs.com/xiaochun126/p/5086037.html)

# 一. 选择排序

**代码实现：**

首先确定循环次数，并且记住当前数字和当前位置。
将当前位置后面所有的数与当前数字进行对比，小数赋值给key，并记住小数的位置。
比对完成后，将最小的值与第一个数的值交换。
重复2、3步。

分析: 选择排序重点在于挑选出最值**作一次交换**, 而**不需挪动多个**完成一趟循环(但是每趟循环还是会对每个元素作比较). 所以, 逻辑是这样 – 遍历整个序列，将最小的数放在最前面。遍历剩下的序列，将最小的数放在最前面。重复第二步，直到只剩下一个数。

1. if(a[j] < val) { val = a[j]; pos = j;}//只要一看到小的值, 就用它覆盖掉自己充当小人物, 并记录下标.
2. for(int j=i+1; j<len; j++){ 1. }//因为是将挑到的最小值放在前面, 所以j的每次起点要记得挪位.
3. int val = a[i]; int pos = i;
4. for(int i: 0~len-1)

# 二. 插入排序

**代码实现：**

首先设定插入次数，即循环次数，for(int i=1;i<length;i++)，1个数的那次不用插入。
设定插入数和得到已经排好序列的最后一个数的位数。insertNum和j=i-1。
从最后一个数开始向前循环，如果插入数小于当前数，就将当前数向后移动一位。
将当前数放置到空着的位置，即j+1。

分析: 对待排的一组数, 假设前面第一个数是排好顺序的, 现在要把后面的数据依次插入, 每次插入都能构成一个新的有序序列, 直到把所有数据插入排序顺序.

1. while(a[j] > insert && j>=0){ a[j+1] = a[j]; j–;}//从已排好序列的末尾开始, 若当前位置的数值若大于待插入元素值, 将其后挪一位, 循序比较, 如果所有数都大于待插入数值, 则比较到第一位0号元素.
2. { 1. } a[j+1] = insert;
3. int j=i-1; insert = a[i];
4. for(int i:1~len-1)//从1开始是因为, 加入数组只有一个则不用排
5. int len = a.length; int insert;

# 三. 冒泡排序

**代码实现：**

设置循环次数。
设置开始比较的位数，和结束的位数。
两两比较，将最小的放到前面去。
重复2、3步，直到循环次数完毕。

分析: 其实思想就是这么个事情, 记住了也不一定能一下写出来, 为什么? 那是由于你的代码转换思维能力的发挥.

1. if(a[j] > a[j+1]) { int temp = a[j]; a[j] = a[j+1]; a[j+1] = temp;}
2. for(int j=o; j<n-1 - i; j++){ 1. }
3. for(int i=0; i<n; i++)

# 四. 快速排序

**代码思路:**

要求时间最快时。
选择第一个数为p，小于p的数放在左边，大于p的数放在右边。
递归的将p左边和右边的数都按照第一步进行，直到不能递归。

分析: 分治也就是化整为零, 分成小块->小块排好->整体排好

1. void partition(int s[], int start, int end)//分治挖坑
2. while(s[j] >= x && i<j) j–;//从右开水找小于x的(因为不符合的就是小于的数)数据来填a[i]
3. if(i<j) { s[i] = s[j]; i++;}//将找到的s[j]填入s[i], 是s[j]就会形成一个新的坑
4. while(s[i] =的数)数来填s[j]
5. if(i<j) { s[j] = s[i]; j–;}//将找到的s[i]填入s[j], 是s[i]就会形成一个新的坑
6. while(i4.;}//外层循环
7. s[i] = x;//i和j相遇了, 此时i==j; 记得将基准值插入最后停留的位置(任何位置都可能, 取决于数据组)
8. int i = start; j = end; int x = s[start]; {5->6.} return i;//将i返回给作为新的基准位置
9. void quickSort(int s[], int start, int end)//快排递归
10. if(start<end){ int baseIndex = partition(s,start,end); quickSort(s, start, baseIndex - 1); quickSort(s, baseIndex+1, end);}//使用递归将每次的分治规模减小, 知道所有数据排好顺序

注： baseIndex=privot=x;

# 五. 归并排序

> 速度仅次于快排，内存少的时候使用，可以进行并行计算的时候使用。

**代码思路：**

1. 选择相邻两个数组成一个有序序列。
2. 选择相邻的两个有序序列组成一个有序序列。
3. 重复第二步，直到全部组成一个有序序列。

代码：

```
public static void mergeSort(int[] numbers, int left, int right) {   
    int t = 1;// 每组元素个数   
    int size = right - left + 1;   
    while (t < size) {   
        int s = t;// 本次循环每组元素个数   
        t = 2 * s;   
        int i = left;   
        while (i + (t - 1) < size) {   
            merge(numbers, i, i + (s - 1), i + (t - 1));   
            i += t;   
        }   
        if (i + (s - 1) < right)   
            merge(numbers, i, i + (s - 1), right);   
    }   
}   
private static void merge(int[] data, int p, int q, int r) {   
    int[] B = new int[data.length];   
    int s = p;   
    int t = q + 1;   
    int k = p;   
    while (s <= q && t <= r) {   
        if (data[s] <= data[t]) {   
            B[k] = data[s];   
            s++;   
        } else {   
            B[k] = data[t];   
            t++;   
        }   
        k++;   
    }   
    if (s == q + 1)   
        B[k++] = data[t++];   
    else  
        B[k++] = data[s++];   
    for (int i = p; i <= r; i++)   
        data[i] = B[i];   
}
```

# 六. 堆排序

> 对简单选择排序的优化。

**代码思路：**

1. 将序列构建成大顶堆。
2. 将根节点与最后一个节点交换，然后断开最后一个节点。
3. 重复第一、二步，直到所有节点断开。

代码：

```
public  void heapSort(int[] a){
        System.out.println("开始排序");
        int arrayLength=a.length;
        //循环建堆  
        for(int i=0;i<arrayLength-1;i++){
            //建堆  

            buildMaxHeap(a,arrayLength-1-i);
            //交换堆顶和最后一个元素  
            swap(a,0,arrayLength-1-i);
            System.out.println(Arrays.toString(a));
        }
    }
    private  void swap(int[] data, int i, int j) {
        // TODO Auto-generated method stub  
        int tmp=data[i];
        data[i]=data[j];
        data[j]=tmp;
    }
    //对data数组从0到lastIndex建大顶堆  
    private void buildMaxHeap(int[] data, int lastIndex) {
        // TODO Auto-generated method stub  
        //从lastIndex处节点（最后一个节点）的父节点开始  
        for(int i=(lastIndex-1)/2;i>=0;i--){
            //k保存正在判断的节点  
            int k=i;
            //如果当前k节点的子节点存在  
            while(k*2+1<=lastIndex){
                //k节点的左子节点的索引  
                int biggerIndex=2*k+1;
                //如果biggerIndex小于lastIndex，即biggerIndex+1代表的k节点的右子节点存在  
                if(biggerIndex<lastIndex){
                    //若果右子节点的值较大  
                    if(data[biggerIndex]<data[biggerIndex+1]){
                        //biggerIndex总是记录较大子节点的索引  
                        biggerIndex++;
                    }
                }
                //如果k节点的值小于其较大的子节点的值  
                if(data[k]<data[biggerIndex]){
                    //交换他们  
                    swap(data,k,biggerIndex);
                    //将biggerIndex赋予k，开始while循环的下一次循环，重新保证k节点的值大于其左右子节点的值  
                    k=biggerIndex;
                }else{
                    break;
                }
            }
        }
    }
```

# 七. Shell排序

> 对于直接插入排序问题，数据量巨大时。

代码思路：

1. 将数的个数设为n，取奇数k=n/2，将下标差值为k的书分为一组，构成有序序列。
2. 再取k=k/2 ，将下标差值为k的书分为一组，构成有序序列。
3. 重复第二步，直到k=1执行简单插入排序。

代码实现：

如何写成代码：

1. 首先确定分的组数。
2. 然后对组中元素进行插入排序。
3. 然后将length/2，重复1,2步，直到length=0为止。

代码：

```
public  void sheelSort(int[] a){
        int d  = a.length;
        while (d!=0) {
            d=d/2;
            for (int x = 0; x < d; x++) {//分的组数
                for (int i = x + d; i < a.length; i += d) {//组中的元素，从第二个数开始
                    int j = i - d;//j为有序序列最后一位的位数
                    int temp = a[i];//要插入的元素
                    for (; j >= 0 && temp < a[j]; j -= d) {//从后往前遍历。
                        a[j + d] = a[j];//向后移动d位
                    }
                    a[j + d] = temp;
                }
            }
        }
    }
```

# 八. 基数排序

代码思路：

用于大量数，很长的数进行排序时。

将所有的数的个位数取出，按照个位数进行排序，构成一个序列。

将新构成的所有的数的十位数取出，按照十位数进行排序，构成一个序列。

代码：

```
public void baseSort(int[] a) {
               //首先确定排序的趟数;    
               int max = a[0];
               for (int i = 1; i < a.length; i++) {
                   if (a[i] > max) {
                       max = a[i];
                   }
               }
               int time = 0;
               //判断位数;    
               while (max > 0) {
                   max /= 10;
                   time++;
               }
               //建立10个队列;    
               List<ArrayList<Integer>> queue = new ArrayList<ArrayList<Integer>>();
               for (int i = 0; i < 10; i++) {
                   ArrayList<Integer> queue1 = new ArrayList<Integer>();
                   queue.add(queue1);
               }
               //进行time次分配和收集;    
               for (int i = 0; i < time; i++) {
                   //分配数组元素;    
                   for (int j = 0; j < a.length; j++) {
                       //得到数字的第time+1位数;  
                       int x = a[j] % (int) Math.pow(10, i + 1) / (int) Math.pow(10, i);
                       ArrayList<Integer> queue2 = queue.get(x);
                       queue2.add(a[j]);
                       queue.set(x, queue2);
                   }
                   int count = 0;//元素计数器;    
                   //收集队列元素;    
                   for (int k = 0; k < 10; k++) {
                       while (queue.get(k).size() > 0) {
                           ArrayList<Integer> queue3 = queue.get(k);
                           a[count] = queue3.get(0);
                           queue3.remove(0);
                           count++;
                       }
                   }
               }
        }
```

# 九. 桶排

**代码思路：**

------

桶排序的思想近乎彻底的分治思想。假设现在需要对一亿个数进行排序。我们可以将其等长地分到10000个虚拟的“桶”里面，这样，平均每个桶只有10000个数。如果每个桶都有序了，则只需要依次输出为有序序列即可。具体思路是这样的：

1.将待排数据按一个映射函数f(x)分为**连续的**若干段。理论上最佳的分段方法**应该使**数据平均分布；实际上，通常采用的方法都做不到这一点。显然，对于一个已知输入范围在【0，10000】的数组，最简单的分段方法莫过于x/m这种方法，例如，f(x)=x/100。

“连续的”这个条件非常重要，它是后面数据按顺序输出的理论保证。

2.分配足够的桶，按照f(x)从数组起始处向后扫描，并把数据放到合适的桶中。对于上面的例子，如果数据有10000个，则我们需要分配101个桶（因为要考虑边界条件：f(x)=x/100会产生【0，100】共101种情况），理想情况下，每个桶有大约100个数据。

3.对每个桶进行内部排序，例如，使用快速排序。注意，如果数据足够大，这里可以继续递归使用桶排序，直到数据大小降到合适的范围。

4.按顺序从每个桶输出数据。例如，1号桶【112，123，145，189】，2号桶【234，235，250，250】，3号桶【361】，则输出序列为【112，123，145，189，234，235，250，250，361】。

5.排序完成。

代码：

```
public static void bucketSort(int[] arr){
    //分桶，这里采用映射函数f(x)=x/10。
    //输入数据为0~99之间的数字
    int bucketCount =10;
    Integer[][] bucket = new Integer[bucketCount][arr.length];  //Integer初始为null,以与数字0区别。
    for (int i=0; i<arr.length; i++){
        int quotient = arr[i]/10;   //这里即是使用f(x)
        for (int j=0; j<arr.length; j++){
            if (bucket[quotient][j]==null){
                bucket[quotient][j]=arr[i];
                break;
            }
        }
    }
    //小桶排序
    for (int i=0; i<bucket.length; i++){
            //insertion sort
            for (int j=1; j<bucket[i].length; ++j){
                if(bucket[i][j]==null){
                    break;
                }
                int value = bucket[i][j];
                int position=j;
                while (position>0 && bucket[i][position-1]>value){
                    bucket[i][position] = bucket[i][position-1];
                    position--;
                }
                bucket[i][position] = value;
            }
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             
    }
    //输出
    for (int i=0, index=0; i<bucket.length; i++){
        for (int j=0; j<bucket[i].length; j++){
            if (bucket[i][j]!=null){
                arr[index] = bucket[i][j];
                index++;
            }
            else{
                break;
            }
        }
    }
}
```

参考文章:

- [Java常用的八种排序算法与代码实现](https://www.cnblogs.com/10158wsj/p/6782124.html?utm_source=tuicool&utm_medium=referral)
- [一遍记住 Java 常用的八种排序算法与代码实现](https://zhuanlan.zhihu.com/p/27005757)
- [白话经典算法系列之六 快速排序 快速搞定](https://blog.csdn.net/morewindows/article/details/6684558)
- [用Java写算法之八：桶排序](http://blog.51cto.com/flyingcat2013/1286645)
