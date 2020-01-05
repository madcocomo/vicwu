
本文是[[TDD磕算法] 我为什么尝试用TDD解算法题](http://www.jianshu.com/p/ec2d8fd08c85)系列的一篇。
![](https://upload-images.jianshu.io/upload_images/2453618-78142152fd22271c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###题目
> 在一个行列都升序排列的矩阵中找第n个最小的数。

所谓行列都升序，是指矩阵中的没个数字都比它下面和左面的数字小。比如对于表格：
||||
|-|-|-|
|1|2|4|
|4|6|8|
|5|7|9|
第3小的数字是**4**。
第4小的数字是**4**。
第6小的数字是**6**。
题目见 [Leetcode 378. Kth Smallest Element in a Sorted Matrix](https://leetcode.com/problems/kth-smallest-element-in-a-sorted-matrix)

###初步尝试
这道题目是我第一道尝试TDD的题，如果那时候已经有了后面足够多的教训，结果也许会有不同吧。开始尝试失败的方式与前一篇非常的类似。这里只简单描述一下。

**1. 只有一个数字的矩阵**
```
Given the matrix is:
    |1|
Then I will get 1 at the 1 smallest element
```
最小的数字永远在左上角
**2. 2x2的矩阵，第2小的数字**
```
Given the matrix is:
    |1|2|
    |3|4|
Then I will get 2 at the 2 smallest element

Given the matrix is:
    |1|3|
    |2|4|
Then I will get 2 at the 2 smallest element
```
代码
```
if (index == 1) { ... }
if (at(0,1) < at(1,0)) {
    return at(0,1);
} else {
    return at(1,0);
}
```
**4. 3x3矩阵，第3小的数字**
```
Given the matrix is:
    |1|2|3|
    |4|5|6|
    |7|8|9|
Then I will get 3 at the 3 smallest element
...
```
```
if (index == 1) { ... }
if (index == 2) {
    return Math.min(at(0,1), at(1,0));
}
return at(0,2);
```
……
**n. 取到第5个数字，分支越来越复杂，仍然没有重构出合适的结构**
```
...
Given the matrix is:
    |1|2|3|9|9|
    |4|5|9|9|9|
    |9|9|9|9|9|
    |9|9|9|9|9|
    |9|9|9|9|9|
Then I will get 5 at the 5 smallest element
...
Given the matrix is:
    |1|2|3|9|9|
    |4|9|9|9|9|
    |5|9|9|9|9|
    |9|9|9|9|9|
    |9|9|9|9|9|
Then I will get 5 at the 5 smallest element
```
代码长成了这样
```
if (index > size()) { return max(at(0,1), at(1,0)); }
if (index == 1) { return at(0,0); }
if (index == 2) { return min(at(0,1), at(1,0)); }
if (index == 3) {
    if (at(1,0) > at(0,2)) { return at(0,2); } 
    if (at(0,1) > at(2,0)) { return at(2,0); }
    return max(at(0,1), at(1,0));
} 
if (index == 4) {
    if (at(1,0) > at(0,3)) { return at(0,3); }
    if (at(0,1) > at(3,0)) { return at(3,0); }
    if (at(1,0) > at(0,2)) { return at(1,0); }
    if (at(0,1) > at(2,0)) { return at(0,1); } 
    return min(at(0,2), at(2,0), at(1,1));
}
if (at(1,0) > at(0,4)) { return at(0,4); }
if (at(1,0) > at(0,3)) { return at(1,0); }
if (at(2,0) > at(0,3)) { return at(0,3); }
if (at(1,1) < at(0,2) || at(1,1) < at(2,0)) { return at(1,1); }
return max(at(2,0), at(0,2));
```
如果你看过这个系列第一篇的失败例子，会发现基本上是一样的。
简而言之，单纯的从少到多穷举罗列用例并不能保证浮现出设计。
这次回顾，最惊讶的是当初竟然坚持了这么久才放弃。看来反复失败还是学乖了一点。
想看失败历程的可以点这里，欢迎指点。
http://cyber-dojo.org/review/show/91F6A9AFC1?avatar=lobster&was_tag=-1&now_tag=-1

###重新思考
放下键盘，开始重新思考这个问题。
先抛开性能方面的担心，想想怎么找出一个逻辑上必然能解决的方法。

有个方法是显而易见的，把所有数字放进一个列表，排序，然后取第n个。但是这个方法明显不是正解。原因是在解决的过程中已经抛弃了题面提供的重要信息——行列都有序。所以这种解法是不可能在后续优化来达到性能目标的。

因此还是要考虑能够利用到题目给出信息的解法。
非常类似的，这次解题的过程中我也感觉到了写Test Case的困难。虽然有两个最容易看到的维度可以让测试从简单到复杂演化。这两个维度就是矩阵的大小、和n的大小。但是可以看到随着这两个数字增大，出现的分支情况越来越多。

一个更明显的迹象是，我在写用例的时候从没想清楚过应该怎么产生一个这样的矩阵。
一个简单的规则是把一长队排序的数字从左往右，逐行排满矩阵。比如：1,2,3,4,5,6,7,8,9。可以分三行排成：

    1,2,3
    4,5,6
    7,8,9
当然也可以从上往下，逐列排成：

    1,4,7
    2,5,8
    3,6,9
但是还有更多的情况，比如：

    1,2,6
    3,4,7
    5,8,9
而且随着数字增多，可能的排列也越多。怎么保证测试到了所有的可能呢？

在思考如何排出有代表性的组合的过程中，脑子里浮现出了一个意象：堆柴火。

![](http://upload-images.jianshu.io/upload_images/2453618-ce265ad5687b2fbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

想象一下把一堆均匀的木头堆在墙角，就像柱状图一样。
与柱状图不同的是，一根柱子不会独树一帜特别的高。否则上面的“木头”因为“太重”就会滚到旁边，这样才能保证每块木头都比它上面、右边的木头轻。
所以一个这样的木材堆可以这样产生：
第一根木头放好，第二根要么堆在它的顶上，要么滚下来堆在它旁边。第三根的时候要么堆在最靠近墙角一列的顶上，要么在旁边的一列的顶部。
把矩阵的左上角想象为墙角，数字想象成木头。那么只要按照堆积的这种逻辑，从小到大逐一判断每根“木头”堆在了哪个格子，直到第n个。也就解出了问题。
至此，心中豁然开朗，再也不担心无穷无尽可能的组合。信心满满的开始编码了。

###以隐喻为模型重新实现
起始的套路一样，还是从1x1的矩阵开始，再到2x2。这时因为脑子里有了“木材堆”，实现不再是茫无头绪的if了，将找下一个的任务委托给了`next()`方法。
```java
public int get(int index) {
    if (index == 2) {
        Integer result;
        result = next();
        result = next();
        return result;
    }
    return at(0,0);
}
```

显然，`next()`需要维持一个状态，这个状态就是上面想象中的“堆木头”的过程，随着每一块木头放好，记录每一列木柴柱的高度，这样下一块木头的可能位置就确定了。
根据这个想法写出单元测试。
```java
@Test
public void should_get_smallest_from_avaliable_cells() {
    //given
    doReturn(new int[]{2,1,0,0,0}).when(retriever).rowRecords();
    doReturn(5).when(retriever).at(0,2);
    doReturn(4).when(retriever).at(1,1);
    doReturn(6).when(retriever).at(2,0);
    //when
    int actual = retriever.next();
    //then
    verify(retriever).at(0,2);
    verify(retriever).at(1,1);
    verify(retriever).at(2,0);
    verify(retriever).record(1,1);
    assertEquals(4, actual);
}
```
上面测试假定了如下的木材堆：
0|1|2|3|4
-|-|-|-|-
□ 5|||||
■|□ 4||||
■|■|□ 6|||

■ 表示已经堆好的数字，□ 表示在这时新的数字可能堆在的位置。那么只需判断这几个点的数字大小就知道实际“堆”在了哪里。并且更新状态。比如这次next()取到数字4后，记录的状态变为。
0|1|2|3|4
-|-|-|-|-
□ 5|□||||
■|■ ||||
■|■|□ 6|||

经过一段重构后，`next`的实现如下
```java
protected Integer next() {
    int[] rowRecords = rowRecords();
    int row = 0;
    int col = rowRecords[0];
    Integer result = at(0, col);
    for (int i = 1; i < rowRecords.length; i++) {
        Integer candidate = at(i, rowRecords[i]);
        if (result > candidate) {
            result = candidate;
            row = i;
            col = rowRecords[i];
        }
        if (rowRecords[i] == 0) {
            break;
        }
    }
    record(row, col);
    return result;
}

protected void record(int row, int col) {
    rowRecords[row]=col+1;
}  
```
这时主线的逻辑已经完成，而且与矩阵的大小无关。后面是把`next`遍历的职责与矩阵分离，一些处理边界值和优化效率的部分。有兴趣的话可以在下面地址看到实现过程：
http://cyber-dojo.org/review/show/91F6A9AFC1?avatar=bee&was_tag=-1&now_tag=-1&filename=undefined

###回顾
通过两次开发过程的差别，可以发现在试图通过罗列测试来演进实现失败的时候，往往最先遇到的困难不是如何设计实现，而是如何选定测试用例。
常见的感受是列举了测试发现和已有的无差别，或是列出无数个用例还感觉无法完全覆盖。这体现的是对问题的性质理解不够，何谈浮现设计呢。
不过即便是失败的尝试，也会产生一些启发。当启发足够对问题产生成型的理解后。这时的测试用例的添加会是对问题有效的深化、分解，而非简单的罗列。在这个阶段的测试驱动开发，就可以比较好的确保分析到实现的转换。

#####附记
在自己琢磨实现后，看了看别人是如何解决的。
发现有一种解法是使用“最大/最小堆”这种数据结构，会比自己瞎琢磨的更有效率。
此前对这个数据结构没有了解，没想到不谋而合的找到了“堆”这个意象。
