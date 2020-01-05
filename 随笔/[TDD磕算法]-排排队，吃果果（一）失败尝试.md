本文是[[TDD磕算法] 我为什么尝试用TDD解算法题](http://www.jianshu.com/p/ec2d8fd08c85)系列的一篇。
####题目
原谅我标题党了，其实是这次解的题目很难一言两语概括出来。这里我尝试解释解释。
>假定一队人排成一列
每个人向前看，如果看到前面有1个比自己高、或者和自己同样高的人。就举个牌子报数1。同理如果有3个，就报3。
如果把每个人的身高和报数列出来，就形成一个二元组的队列，比如：
[(172, 0), (160, 1), (182, 0), (170, 2)]
就表示第一个人172cm，前面没有高于或等于他高度的人；第二个160cm，前面有一个；……

>这道题目是，给定打乱顺序的一个序列，把他们排回应有的顺序。
比如：
输入：[ (7,2), (4,3), (8,0), (7,1), (6,0)]
输出：[ (6,0), (8,0), (7,1), (4,3), (7,2)]

这道题可以在刷题网站Leetcode上看到，编号406
https://leetcode.com/problems/queue-reconstruction-by-height

![](http://upload-images.jianshu.io/upload_images/2453618-f96e1482f5c9a8bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####初步分析
初看起来题目非常适合TDD解决。排序的逻辑似乎有点复杂，但是应该比较容易一步步从简单到复杂演进。比如先从最简单的情况开始，队列里只有一个人，可以直接返回。两个人的话有两种情况……
另外这道题目不禁让我想起了Combined Number这道Kata。同样也是实现排序，排序的逻辑有些绕。最终可以很漂亮的把问题的几个方面分离开解决掉。

    关于Combined Number，可以在这里看到更详细的练习和讨论。
    http://cyber-dojo.org/kata/edit/FCDDF1?avatar=zebra
    https://groups.google.com/forum/#!topic/agileshanghai/d6cy6tCXHA8

####失败尝试
大致想想之后就开始信心满满的TDD起来了，果不其然失败了……
全过程见 http://cyber-dojo.org/review/show/8922CE128B?avatar=ray&was_tag=1&now_tag=1
大体历程如下
######第一个测试
只有一个人的话，必然前面没有更高的人。
```
reconstruct_should_be([[3,0]], [[3,0]]);
```
`reconstruct_should_be`的第一个参数是输入，第二个是预期的输出。

实现代码，输入什么就返回什么。
```
function reconstrutQueue(people) {
  return people;
}
```
######第二个测试
两个人，有一个前面有更高的人，另一个没有。
```
reconstruct_should_be([[2,1],[3,0]], [[3,0],[2,1]]);
```
实现代码，按前面人数排序。
```
function reconstructQueue(people) {
  return people.sort( (p1, p2) => p1[1] -p2[1]);
}
```
稍作重构，抽取常量
```
return people.sort( (p1, p2) => p1[PRE] -p2[PRE]);
```
######第三个测试
两个人，前面都没有更高的人。也就是说是低的在前。
```
reconstruct_should_be([[2,0],[3,0]], [[2,0],[3,0]]);
```
直接通过了。
######第四个测试
三个人，从低到高排。
```
reconstruct_should_be([[2,0],[3,0],[1,0]], [[1,0],[2,0],[3,0]]);
```
实现代码，如果前面人数一样，按高度排序。
```
  return people.sort( (p1, p2) => {
    if (p1[PRE] === p2[PRE]) return p1[0] - p2[0]
    return p1[PRE] -p2[PRE]
  });
```
######第五个测试
三个人，中等高度的排在最高的后面。
```
reconstruct_should_be([[2,1],[3,0],[1,0]], [[1,0],[3,0],[2,1]]);
```
又一次直接通过了。
######第六个测试
三个人，最矮的排在中间。
```
reconstruct_should_be([[2,0],[3,0],[1,1]], [[2,0],[1,1],[3,0]]);
```
实现代码，排序方式改为先按高度排列，相同高度的按前面人数排列。这样三个人就会排成[1,1],[2,0],[3,0]。
```
  let result = people.sort( (p1, p2) => {
    if (p1[VAL] === p2[VAL]) { 
      return p1[PRE] - p2[PRE];
    }
    return p1[VAL] - p2[VAL];
  });
```
在排序之后增加一个调整顺序的过程。硬编码快速通过，把前面有1个人的项目向后挪一位。也就是[1,1]挪到[2,0]后面。
```
  for (let i=0; i<result.length; i++) {
    if (result[i][PRE] === 1) {
      let temp = result[i];
      result[i] = result[i+1];
      result[i+1] = temp;
      i++;
    }
  }
```
######第七个测试
三个人，最矮的排在最后。
```
reconstruct_should_be([[2,0],[3,0],[1,2]], [[2,0],[3,0],[1,2]]);
```
实现代码，增加第二个硬编码分支，处理有2个人在前面的情况。
```
  for (let i=0; i<result.length; i++) {
    if (result[i][PRE] === 1) {
    ...
    }
    if (result[i][PRE] === 2) {
      let temp = result.splice(i,1)[0];
      result.splice(i+2,0,temp);
      i+=2;
    }
  }
```
重构，把两个分支改写成相同的逻辑。
```
    if (result[i][PRE] === 1) {
      let temp = result.splice(i,1)[0];
      result.splice(i+1,0,temp);
      i+=1;
    }
    if (result[i][PRE] === 2) {
      let temp = result.splice(i,1)[0];
      result.splice(i+2,0,temp);
      i+=2;
    }
```
重构，统一为一个分支。
```
    if (sortedByVal[i][PRE] > 0) {
      let nLater = sortedByVal[i][PRE];
      let temp = sortedByVal.splice(i,1)[0];
      sortedByVal.splice(i+nLater,0,temp);
      i+=nLater;
    }
```
######第八个测试
最高的人在前面，后面的两个按高度排。
```
reconstruct_should_be([[2,1],[3,0],[1,1]], [[3,0],[1,1],[2,1]]);
```
实现代码，呃…… 就这么掉坑里了。
首先是没有很容易的快速通过的思路。
反复几次后发现在排好的数组里挪动的方式，把一个人向后挪了之后，换来的人可能也需要后移。
所以决定不在原来的数组里挪动位置，而是全部插入一个新数组。插入时比较新数组里的位置和这个人前面人的个数，直到所有的人都插入新数组为止。
```
function reorder(sortedByVal) {
  let result = [];
  let length = sortedByVal.length;
  for (let i=0; i<length; i++) {
    for (let j=0; j<sortedByVal.length; j++) {
      if (sortedByVal[j][PRE] <= i) {
        result.push(sortedByVal[j]);
        sortedByVal.splice(j,1);
        break;
      }
    }
  }
  return result;
}
```
但是发现这时候[ [ 1, 0 ], [ 3, 0 ], [ 2, 1 ] ]这个测试又失败了。
经过一段思索无法继续后，我觉得就算通过苦思冥想把这个题解出来，也已经和TDD无关了。就此停止了这次练习。
####感想
从来没有尝试过仔细描述一段失败的过程。不知道你读到这里是和感受？
- “简直是浪费时间嘛”？
- “早就知道这样做不靠谱”？
- “看起来还有救啊，怎么就失败了”？
- “太菜了，让我来”？

这里说说掉坑的心得。怎么知道自己掉坑了。
- 最终爬不出来的点很容易判定。突然发现自己思考了10分钟，还是没有写代码。
- 顾此失彼，写了一段代码快速通过当前测试，结果前面的测试挂了。
- 感觉前面重构出的结构在阻碍新的用例快速通过。
- 一个早期征兆是想不清楚下一个测试是什么？比如这个过程中两次直接通过的测试，都代表了对当前程序的行为没有理解清楚。
- 测试想不清楚的背后是没有想出逐步分解问题的思路，进而加入测试的顺序可能不是适宜代码演进。导致重构时对实现进行重大调整。比如第六步和最后一步。

下一篇文章会总结我在反思之后重新解决的过程，敬请期待。
