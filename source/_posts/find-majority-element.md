---
title: (翻)多数投票算法--查找列表中的多数元素
date: 2019-05-30 10:39:25
updated: 2019-05-30 10:39:25
tags: algorithm
---

[原文链接](https://gregable.com/2013/10/majority-vote-algorithm-find-majority.html)

## 背景
假设有一个未排序的值列表，问列表中是否存在一个多数元素，该元素在整个列表中出现次数占列表的一半以上。  
如果存在，求取该元素的值，要求算法尽量高效。  

> 一般情况下，我们可以想到两种直接的方法：
1. 申请额外的空间来记录每个元素出现的次数，这是用空间换时间的方法，时间复杂度和空间复杂度均为O(n)。
2. 如果原列表允许进行修改，可以对原列表进行快排。对于排序后的有序列表，可以很方便地求出是否存在多数元素。时间复杂度为O(n\*lgn)，空间复杂度为O(1)。

<!-- more -->
上面提到的方法都包含冗余的计算，针对这个问题有一个更方便的解决方法：
## Boyer-Moore多数投票算法
具体的算法可以参照这里：[Boyer-Moore Majority Vote Algorithm](http://www.cs.rug.nl/~wim/pub/whh348.pdf)  
该算法仅需要额外的O(1)空间和O(n)的时间复杂度。它需要对输入列表做两次值传递。该算法的实现很简单，但是理解上有一点复杂。  
在第一次对列表的遍历中，我们生成一个候选值，如果存在多数元素，则该候选值即为多数元素。第二遍遍历只是计算该值的频率进行确认。算法的重点在第一部分：  
在第一次遍历中，我们需要俩个值：
1. 一个candidate值，表示可能为多数元素的候选值，最初可以设置为任何值。
2. 一个count值，表示该候选值的出现次数，最初设置为0。
对于输入列表中的每个元素，我们首先检查count值，如果count为0，我们将candidate设置为该元素的值。接下来，将元素的值与当前的candidate值进行比较，如果相同，count加1，否则count减1。（对count和candidate的检查可以互换，这个看每个人代码的写法）  
算法的伪代码如下：
```C++
int candidate = 0;
int count = 0;
for(auto it : input){
	if(count == 0)
		candidate = it;
	if(candidate == it)
		++count;
	else
		--count;
}
```
遍历之后，若列表中确定存在多数元素，则candidate即为该元素的值。第二次的遍历即验证candidate是否为多数元素。  
## 解释
对于算法的原理，若列表中不存在满足要求的多数元素时，第二次遍历很轻易就能看出问题；若存在多数元素，当该元素的要求为出现次数大于[n/2]时很好理解，下面举例说明：  
假如有下面一个多数值为0的列表：  
```
[5,5,0,0,0,5,0,0,5]  
```
处理第一个元素时，candidate = 5, count = 1。  
由于5不是多数值，因此跑上面的算法时，在列表的某个节点上，candidate必定会变成多数元素对应的那个值。  
在上面的例子中，发生在第4个元素：  
```
candidate: [5, 5, 5, 0, 0, 5, 0, 0, 0]  
count:	   [1, 2, 1, 0, 1, 0, 1, 2, 1]   
```
（个人理解：从count方面出发，多数元素和其他所有元素以1：1进行消耗，而由于多数元素占据超过半数的数量，因此最后多数元素会胜出）  
原文中有更详细的解释，个人觉得很容易理解就略过。  

## 进阶
上面的算法进行了两次值传递，最坏情况下要比较2N次。如果考虑到count最终为0的情况，则还需要另外一个N。还有一个更复杂的算法只使用3N/2-2的时间，但是需要N的额外空间。[论文](http://www.cs.yale.edu/publications/techreports/tr252.pdf)  
大致的方法是重新排列所有元素，以便没有两个相邻的元素具有相同的值并跟踪桶中的元素。  
在第一次遍历中，我们从一个空的重排列表和空桶开始。从输入列表中获取元素并与重排列表的最后一个元素进行比较。如果相同，则将元素放在桶中，如果不想等，则将元素添加到列表的末尾，然后将一个元素从桶中移动到列表的末尾。在此阶段结束后，列表中的最后一个值即为candidate。  
在第二次遍历中，将candidate于列表中的最后一个值进行比较，如果相同，则从列表末尾丢弃两个值；如果不同，则丢弃列表末尾的一个值和桶中的一个值。不断执行下去，如果桶已经清空但是列表不为空，则说明没有多数元素；若重排的列表已清空而桶不为空，则candidate即为多数元素。  
在实际的应用中，除了一些人为设计的列表，该算法具有更好的性能。  

## 分布式Boyer-Moore
关于并行执行来提高效率，[文档](http://www.crm.umontreal.ca/pub/Rapports/3300-3399/3302.pdf)已经证明，可以通过并行的方法来查找多数元素。  
解决方案归结为，证明Boyer-Moore的第一阶段可以通过组合原始输入的子序列的结果来解决，只要保留两个candidate和count值即可。  
比如下面的列表:  
```
[1,1,1,2,1,2,1,2,2] 
```
如果直接跑上面的算法，能够得到：  
```
candidate = 1
count = 1
```
如果把原始列表拆分成两部分并在每个部分上执行算法，得到结果如下：
```
分裂列表：[1,1,1,2,1]
candidate =  1
count = 3
```
```
分裂列表：[2,1,2,2]
candidate = 2
count = 2
```
如果分别运行得到的candidate相同，则可以直接得到结果。否则，按照下面的算法对count值进行统计后输出。  
```C++
candidate = 0;
count = 0;
for(auto it : parallel_output){
	if(it.candidate == candidate)
		count += it.count;
	else if(it.count > count){
		coutn = it.count;
		candidate = it.candidate;
	}
	else
		count = count - it.count;
}
```
该算法可以多次运行，以便在必要时以树状方式组合并行输出以获得更优的性能。  
