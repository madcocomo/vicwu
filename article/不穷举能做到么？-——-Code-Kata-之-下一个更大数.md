> 这是在code wars上看到的一道题
编写一个函数，接受一个正整数作为输入，然后输出由相同数字组成的下一个更大的数：
```
next_bigger(12)==21 
next_bigger(513)==531 
next_bigger(2017)==2071```
> 如果找不到由相同数字组成的更大整数，则返回-1：
``` 
next_bigger(9)==-1 
next_bigger(111)==-1 
next_bigger(531)==-1```
> code wars 链接
http://www.codewars.com/kata/55983863da40caa2c900004e

这道题最直接的想法，就是穷举出所有可能的数，找出比输入更大的数中最小的即可。
比如 李小波 的绘图分解法
> 算法很简单嘛
![图片发自简书App](http://upload-images.jianshu.io/upload_images/2453618-7abb4a70de9be935.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)
> 这是我的算法，先split，再穷尽组合，再排序，再查下一个更大的
可理解性高，性能可能差点

小宝 Alex 很快写出了一个偷懒解
![Spock](http://upload-images.jianshu.io/upload_images/2453618-bb8dffcd2647de5c.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)
所谓偷懒，是因为直接把实现代码写在了when部分。一眼看起来可能有点晕。
只要把when的内容包成next(number)
测试部分改成
expect:
next(number) == next_larger_number
就会清楚一些了。

当然大家都觉得应该有更节省的办法，能不能用TDD的方式推导出来呢？
赵阳
> 从测试用例开始，1位数f(1)=-1
两位数：f(10), f(11), f(12)
三位数：百位大于等于十个位，百位小于十个位
没完全想清楚，但是ms可以用穷举归纳法
还要归纳，比如f(12)，f(13), f(14)都一样的

张克强
> 采用冒泡算法，一次冒泡即可以。

童小成
> 从右往左比较相邻两数，如果右边比左边大，对调两数输出，遍历后没有对调输出-1，看行不行
……
好像不行，7354

光勇
> 从右向左遍历遍历每一位，找到第一个比当前位上的数大的，就交换两者位置，就是我们要找的数;如果遍历完全部都没有找到，就说明不存在这样的数。不考虑复杂度的情况下，这样对吗？

赵阳：找到第一个比当前数小的吧
> 对的，笔误[呲牙]。应该是找到第一个比当前位小的数交换
应该交换后，还要对交换位后的数排一次序

在思路逐步形成的讨论中，没有尾巴 同学率先写出实现

![Python 版本二](http://upload-images.jianshu.io/upload_images/2453618-9342415fc62d3e7b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

并解释了思路
> AtB
B是有序的
t是第一个破坏有序的
A是前缀
那么我们获得下一个大点的数，只要操作t，B
就一定能获得
在B里面找一个最小的和t换
然后把B变成最小的序列
注意到B是从大到小的
所以只要将B倒序即可
以上操作就可以获得了结果

> 对于这个操作反复进行，最后整个数组会变成有序， 之后结果不会发生变化，产生一个不动点
这个时候可以返回-1
这个性质是可以证明的， 也可以用来写测试

可是我还没完全明白：
> 注意到B是从大到小的， 为什么啊[疑问]

没有尾巴：
> 第一个while 跑完， B就是有顺序的啊
一个while 的目的就是发现最大有序后缀

小宝 Alex 看不过去，也来用比喻来帮我理解：
> 在所有的组合中，最小的是12345顺序上升，最大的是54321顺序下降。观察一下特征，最小的时候山峰在最右边，最大的时候山峰在最左边。中间情况下山峰在中间的某个位置。所以其中一个思路是从最右边开始找，找到第一个山峰然后再找到右边中稍大的数字交换，最后右边升序排。

终于，我明白了。你明白了吗？
这是我的作业
http://cyber-dojo.org/kata/edit/EEF0FCBE08?avatar=gopher

![Groovy Spock](http://upload-images.jianshu.io/upload_images/2453618-a4e140237a56cc11.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

---
*更新的分割线*
做这道题的时候忙着摆弄Groovy了，写出Bug都没发现，多谢 91 同学指出。
下面是修正后的代码，又摆弄了半天Groovy的闭包，[汗]
![改正的Groovy版本](http://upload-images.jianshu.io/upload_images/2453618-9c5fe68c4335e7b6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

91 同学也做了一遍，并写下了很棒的心得。
https://github.com/hatelove/NextLargerNumber/blob/master/NextLargerNumber/NextLargerNumberTests.cs

>【TDD 過程心得】
這個練習我刻意以 TDD 為主，algorithm 為輔。有刻意先忘了演算法，而盡量先用 baby step 往演算法走，希望每一次的測試案例驅動都帶著某個特定的商業邏輯，在 TDD 堆砌過程中，這些邏輯的處理可以被保留下來重用。

>雖然最後優化演算法時，還是避不了就像整個重寫一般的心情。但此時的「重寫」好處：
① 該有的關鍵測試案例已經存在
② 腦袋思路清晰
③ 已先滿足需求，調整算法過程中，隨時可以 reverse

>【交流心得】
很喜歡這樣透過 kata 跟一些夥伴練習與交流。
① 彼此給彼此建議
② 減少盲點
③ 增加技能的邊界
④ 夥伴情感交流
⑤ coding 手感與思路的練習
⑥ 欣賞不同語言的美

>**即使我們不曾見過面，卻也有種惺惺相惜的感覺。**

#####思考题

> 给定一个字符串，求由相同字符可以组成的所有字符串。如：
```
are →[aer, are, ear, era, rae, rea]
```
这道题目很容易推导出递归的解法。
然而如果用今天Kata的思路，可以写出一个非递归的解法。
你想到了么？
