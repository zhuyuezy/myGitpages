---
title: 多数投票算法
date: 2019-05-17 11:49:56
categories: 
- 算法
tags: 
- 算法
- leetcode题解
---

在 [leetcode 169.求众数](https://leetcode-cn.com/problems/majority-element/)这道题的交流区看见的这个算法，比我的解法妙多了...( ´•̥̥̥ω•̥̥̥` )

# 算法描述
多数投票算法（Boyer–Moore majority vote algorithm）用于寻找一个数字序列中的众数，此处众数指出现次数大于等于1/2序列大小的数。

<!-- more -->

该算法用自然语言描述如下：
 

```
Initialize an element m and a counter i with i = 0
For each element x of the input sequence:
If i = 0, then assign m = x and i = 1
else if m = x, then assign i = i + 1
else assign i = i − 1
Return m
```

原理：不断从序列中选择一对不同的数字消去，直到无法消除，剩下的就是众数。

举例理解：
> 来自：[喝七喜](https://www.zhihu.com/question/49973163/answer/235921864)

>major 初始化随便一个数，count 初始化为0
>输入：{1,2,1,3,1,1,2,1,5}
>
>扫描到1，count是0（没有元素可以和当前的1抵消），于是major = 1，count = 1（此时有1个1无法被抵消）
>
>扫描到2，它不等于major，于是可以抵消掉一个major => count -= 1，此时count = 0,其实可以理解为扫到的元素都抵消完了，这里可以暂时不改变major的值
>
>扫描到1，它等于major，于是count += 1 => count = 1
>
>扫描到3，它不等于major，可以抵消一个major => count -= 1 => count = 0，此时又抵消完了(实际的直觉告诉我们，扫描完前四个数，1和2抵消了，1和3抵消了)
>
>扫描到1，它等于major，于是count += 1 => count = 1
>
>扫描到1，他等于major，无法抵消 => count += 1 => count = 2 (扫描完前六个数，剩两个1无法抵消)
>
>扫描到2，它不等于major，可以抵消一个major => count -= 1 => count = 1,此时还剩1个1没有被抵消
>
>扫描到1，它等于major，无法抵消 => count += 1 => count = 2
>
>扫描到5，它不等于major，可以抵消一个major => count -= 1 => count = 1
>
>至此扫描完成，还剩1个1没有被抵消掉，它就是我们要找的数。

# 题解

如果确定数组中一定存在众数，简单运用此算法即可，也就是169题中的情况。如果可能不存在众数，这个算法会输出一个不符合条件的元素m。在这种情况下，需要再扫描一次来判断m的出现次数是否足够多。

题解代码：

```c
int majorityElement(int* nums, int numsSize){
    
    int m = nums[0];
    int count=1;
    
    for (int i=1; i<numsSize; i++)
    {
        if (count==0)
        {
            m=nums[i];
            count++;
        }
        
        else if (m == nums[i]) 
            count ++;
        else
            count --;
        
    }
    
    return m;
    
}
```


