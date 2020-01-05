本文是[[TDD磕算法] 我为什么尝试用TDD解算法题](http://www.jianshu.com/p/ec2d8fd08c85)系列的一篇。
前情提要见[[TDD磕算法] 排排队，吃果果（一）失败尝试](http://www.jianshu.com/p/f6114bdd29b5)

####题目
考虑到可能有人没看前一篇，这里把题目再描述一次。
>假定一队人排成一列
每个人向前看，如果看到前面有1个比自己高、或者和自己同样高的人。就举个牌子报数1。同理如果有3个，就报3。
如果把每个人的身高和报数列出来，就形成一个二元组的队列，比如：
[(172, 0), (160, 1), (182, 0), (170, 2)]
就表示第一个人172cm，前面没有高于或等于他高度的人；第二个160cm，前面有一个；……

>这道题目是，给定打乱顺序的一个序列，把他们排回应有的顺序。
比如：
输入：[ (7,2), (4,3), (8,0), (7,1), (6,0)]
输出：[ (6,0), (8,0), (7,1), (4,3), (7,2)]

####对于失败的反思
在第一次尝试，按部就班却失败了的经历后，我反思了一下这个题目究竟应该怎么解决。是不是真的不适合用TDD的方式做？
感觉在以下方面是可以改进的。
- 在做的过程中，其实心里一直有种“这道题肯定有很漂亮的解法，说不定试试就试到了”的心态。有这样的心态，有意无意的会避免一些“傻”的写法，而且会在还没有足够用例支持的时候就急于重构出较为一般化的代码结构。然而并不真正匹配问题，反倒给后续演进造成了障碍。
- 因此首先应该向着逻辑上而言“绝对可以解决”的方案进行，在完全解决前，不要被性能的担忧或者设计偏好所干扰。
- 另一个问题是，在第一次做的时候其实没有仔细想清楚测试顺序。虽然按一个人、二个人、三个人这样的顺序递增穷尽用例看起来很合情合理，但是由此产生的测试却未必与程序需要处理的子问题同步的。

![步骤也许是曲折的，但是一定要通向目标](http://upload-images.jianshu.io/upload_images/2453618-133e3b053cc239f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####第二次尝试
全过程见 http://cyber-dojo.org/review/show/8922CE128B?avatar=rabbit&was_tag=1&now_tag=1
第一个测试仍然是以一个人开始，和第一次完全一样，就不详细说了。

######第二个测试
两个人一队，低的在前。
这次没有像上次一样选择高的在前，因为这样一次引入了两个变化：队列人数和排在前面人的逻辑。
```
reconstruct_should_be([[4,0],[3,0]], [[3,0],[4,0]]);
```
实现代码，按高度排序
```
return people.sort( (p1,p2) => p1[HEIGHT] - p2[HEIGHT] );
```

######第三个测试
两人，高的在前。引入了前面有一个人的逻辑。
```
reconstruct_should_be([[3,1],[4,0]], [[4,0],[3,1]]);
```
实现代码
这次学乖了，刻意先老老实实的写hardcode通过。
```
  let sortedByHeight = people.sort( (p1,p2) => p1[HEIGHT] - p2[HEIGHT] );
  if (deepEqual([[3,1],[4,0]], sortedByHeight)) {
    return [[4,0],[3,1]];
  }
```

######第三个测试的重构
然后开始考虑这个4在3前面的逻辑是怎么来的，可以假定有个函数能给出每个人应有的位置，那么就按它返回的index插入数组即可。

```
for(let p of sortedByHeight) {
  result[peoplesBefore(p, sortedByHeight)] = p;
}
```
这样，就把硬编码挪入了新的函数里。[3,1],[4,0]的时候特殊处理，否则返回在按高度排序数组里的index。
```
function peoplesBefore(people, peoples) {
  if (deepEqual([[3,1],[4,0]], peoples)) {
    if (deepEqual([4,0], people)) {
      return 0;
    }
    if (deepEqual([3,1], people)) {
      return 1;
    }
  }

  for (let i in peoples) {
    if (deepEqual(peoples[i], people)) {
      return i;
    }
  }
}
```
进一步分解硬编码，把[3,1]的情况留在硬编码里特殊处理。先想办法统一前面没有更高的人的情况。
再次假定一个函数，它可以数出前面的人数，比如对于[4,0]而言，在[3,0],[4,0]中返回1，而在[3,1],[4,0]中时返回0。
```
  if (deepEqual([[3,1],[4,0]], peoples)) {
    if (deepEqual([3,1], people)) {
      return 1;
    }
  }
  if (people[PEOPLE_BEFORE] === 0) {
    return shorterPeoplesWhoHasNoTallerBefore(people, peoples);
  }
...
function shorterPeoplesWhoHasNoTallerBefore(people, peoples) {
  let result = 0;
  for (let p of peoples) {
    if (p[HEIGHT] < people[HEIGHT] && p[PEOPLE_BEFORE] === 0) {
      result += 1;
    }
  }
  return result;
}
```
这时候发现[3,1]的硬编码可以简化为前面更高人数为1。同时，既然已经每次数一遍更矮的人数了，前面的排序就没有必要了。
```
function reconstructQueue(peoples) {
  let result = [];
  for(let p of peoples) {
    result[peoplesBefore(p, peoples)] = p;
  }
  return result;
}
 
function peoplesBefore(people, peoples) {
  if (people[PEOPLE_BEFORE] === 0) {
    return shorterPeoplesWhoHasNoTallerBefore(people, peoples);
  }
  if (people[PEOPLE_BEFORE] === 1) {
    return 1;
  }
}
```
经过一大段重构到这个点时，感觉硬编码基本上消除了。添加后续测试。

######第四个测试
与第一次不同，我没有按照人数递增穷举所有可能的思路，而是根据当前代码逻辑。进一步扩充有PEOPLE_BEFORE === 1的逻辑分支。

三个人，中等高度的人排在最后。这样因为他的位置不再是1所以会失败。
```
reconstruct_should_be([[3,1],[8,0],[2,0]], [[2,0],[8,0],[3,1]]);
```
实现代码
发现可以稍作调整就快速通过了。
把原来硬编码的位置1，改为如果没有更高人在前时应在的位置加1。
也就是说本来3排在2后面，现在再后挪一位。
```
  if (people[PEOPLE_BEFORE] === 1) {
    return shorterPeoplesWhoHasNoTallerBefore(people, peoples) + 1;
  }
```

######第五个测试
继续扩充前面有一个人的逻辑分支。

三个人最低的在中间。
```
reconstruct_should_be([[3,1],[8,0],[5,0]], [[5,0],[3,1],[8,0]]);
```
这个用例初看起来和第三个测试类似，但是失败了。原因是数位置的时候是只算[x,0]的个数的。这样8和3都会认为是在位置1的，结果排出来的数组只有两个元素。
实现代码
发现实际上是要把[x,1]这样的元素插入已经排好的[x,0]队列里合适的位置。改为如下实现。
先用原有来的逻辑处理前面没有更高的人，也就是[x,0]的情况。再把[x,1]插入结果队列
```
function reconstructQueue(peoples) {
  let result = [];
  for(let p of peoples) {
    if (p[PEOPLE_BEFORE] === 0) {
      result[shorterPeoplesWhoHasNoTallerBefore(p, peoples)] = p;
    }
  }
  for (let p of peoples) {
    if (p[PEOPLE_BEFORE] === 1) {
      insertAfterFirstNoShorter(p, result);
    }
  }
  return result;
}
```
`insertAfterFirstNoShorter`的实现是找到第一个不低于当前需要插入高度的人，在他后面加入新的人。
比如[5,0],[8,0],需要加入[3,1]，数到5是高于3的，把3加入5之后的位置。
```
function insertAfterFirstNoShorter(people, peoples) {
  for (let i in peoples) {
    if (peoples[i][HEIGHT] >= people[HEIGHT]) {
      peoples.splice(i+1,0,people);
      return;
    }
  }
}
```
######第六个测试
[x,1]的逻辑还没有完备，增加用例有两个[x,1]的情况。
```
reconstruct_should_be([[3,1],[5,1],[8,0]], [[8,0],[3,1],[5,1]]);
```
结果不是预期的8,3,5，而是8,5,3。因为首先3加在8后面，之后5也是加入8随后的位置。顺序反了。
实现代码，修改`insertAfterFirstNoShorter`，把加入第一个更高的人之后改为第二个更高的人之前或者队尾。名字也相应的修改为`insertBefore2ndNoShorter`。
```
function insertBefore2ndNoShorter(people, peoples) {
  let afterFirst = false;
  for (let i in peoples) {
    if (peoples[i][HEIGHT] >= people[HEIGHT]) {
      if (afterFirst) {
        peoples.splice(i,0,people);
        return;
      }
      afterFirst = true;
    }
  }
  peoples.push(people);
}
```

######第六个测试的重构
又补充了几个[x,1]测试都成功了，我确信代码已经完全处理了有一个人在前的逻辑。开始重构。重构的出发点是觉得[x,0]和[x,1]的两段逻辑是可以统一的。
首先修改插入[x,0]的为函数为`insertBefore1stNoShorter`，找到第一个更高的人，加在他前面。
```
  peoples = peoples.sort( (p1, p2) => p1[PEOPLE_BEFORE] - p2[PEOPLE_BEFORE]);
  ...
  for(let p of peoples) {
    if (p[PEOPLE_BEFORE] === 0) {
      insertBefore1stNoShorter(p, result);
    }
    if (p[PEOPLE_BEFORE] === 1) {
      insertBefore2ndNoShorter(p, result);
    }
  }
```
显而易见两个函数的区别只是数人头的个数。
进一步重构，把人数作为参数传入。
```
  for(let p of peoples) {
    insertBeforeNTimesNoShorter(p, result, p[PEOPLE_BEFORE]+1);
  }
```

######第七个测试
增加两个人高度相等的用例
```
reconstruct_should_be([[8,1],[3,1],[8,0]], [[8,0],[3,1],[8,1]]);
```
直接通过了，因为前面写代码的时候其实已经加入了相等的判断。可以说这个测试加的太晚了，或者说是实现代码写的过多。
这个测试算是补救，确保提前写的实现是正确的。

######第八个测试
引入有两个人在前面的用例。
```
reconstruct_should_be([[6,0],[5,2],[3,2],[8,0]], 
      [[6,0],[8,0],[3,2],[5,2]]);
```
出人意料的是直接过了，原因是[x,0],[x,1]或[x,2]已经在前面重构中一般化了。
仔细想想，并增加了几个测试后，发现**原来已经做完了**。
成功好像比上次的失败来的更加突然嘛。

####体会
- 这次成功做法的最大不同，是抛开了对性能的担心。最终做出的结果性能也确实是相当的差。但是至少做完了。
- 另一个重大区别是测试顺序。很明显每一步引起的修改都集中在程序的一部分中，这表明问题被分解开了。
- 写这篇回顾时才发现的这次练习一个不寻常的地方，在于大段的重构。一般大的重构是前面欠债引起的。而这次却不是这个情况。很多代码结构是在消除硬编码的时候经过很多步骤产生出来的，同时外部行为并无变化。这究竟是小步带来的自然演进，还是因为我经过上次失败后脑子里已经预想了一些设计。目前我还不是特别确定。

如前面所言，这个解法是非常低效的，实在不能算是个算法解答。
下一篇将会回顾我探索更好的解法的过程。
