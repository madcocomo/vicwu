**警告：本文包括大段代码，读到代码会感到不适的同学请绕行。**

---
这是我和张云雷在群里进行code review和重构的练习过程。
####题目
Count Coins。大致是这样的，有1分，5分，10分和25分四种硬币。给定一个金额，算出有多少种组合方式表示这个金额。
比如：
- 3分钱只有1种组合方式，3枚分币；
- 5分钱有2种组合，5枚一分硬币，或，1枚五分硬币；
- 10分钱有4种组合，10枚一分，5枚一分加1枚五分，2枚五分，1枚10分；

读到这里，你是不是已经对怎么解决这个问题有些设想了？
实话说我还没有很仔细的想清楚这道题。张云雷在Cyber Dojo上做了练习，把代码贴在TDD群里求Review和重构。

####重构前代码
```
#include "CountCoins.hpp"
enum class CoinsType:int{
    ZERO = 0,
    PENNY=1,
    NICKEL=5,
    DIME=10,
    QUARTER=25
} ;
 
int operator /(int value, CoinsType type)
{
    return value / static_cast<int>(type);
}
 
int operator *(int value, CoinsType type)
{
    return value * static_cast<int>(type);
}
 
int operator -(int value, CoinsType type)
{
    return value - static_cast<int>(type);
}
 
CoinsType operator -- (CoinsType type, int)
{
    switch (type){
        case CoinsType::QUARTER:
            return CoinsType::DIME;
        case CoinsType::DIME:
            return CoinsType::NICKEL;
        case CoinsType::NICKEL:
            return CoinsType::PENNY;
        case CoinsType::PENNY:
            return CoinsType::ZERO;
        default:
            return CoinsType::PENNY;
    }
}
 
int makeChangesWithoutCurrentType(int totalCents, CoinsType type)
{
    return makeChangesWith(totalCents, type--);
}
int makeChangesWithCurrentType(int totalCents, CoinsType type)
{
    int ways = 0;
    for( int i = 1; i <= totalCents / type; i++)
    {
        int smallerNorminalCents = totalCents - i * type;  
        ways += makeChangesWith(smallerNorminalCents, type--);
    }
    return ways;
}
int makeChangesWith(int totalCents, CoinsType type)
{
    if( type == CoinsType::PENNY)
    {
        return 1;
    }
    int ways = 0;
    ways += makeChangesWithCurrentType(totalCents, type);
    
    ways += makeChangesWith(totalCents, type--);
    return ways;
}
int makeChange(int totalCents)
{
    if( totalCents >= 1 )
    {
        return makeChangesWith(totalCents, CoinsType::QUARTER);
    }
    return 0;
}
```

不知道你读这段代码的感受如何，先说说我读到的感觉吧。
####我的review感想
因为我并没有很清晰的思路，对问题也是一知半解。所以第一眼看上去必然是懵圈的。下面是些直观感受：
- 有点长。一屏看不完就不禁让人产生压力，有种无法一手掌握的感觉。
- 当然，有可能问题本身就比较复杂，必须要相当篇幅的代码。不过这样想的话，压力就更大了。
- 仔细看看，其实相当一部分行数是又单独一行的大括号贡献的。不是很熟悉C++，对此就不评论了。
- 可以感觉到张云雷在写这段代码时尽量使它可读的努力。比如函数名字都比较长。函数都很短小。
- 一开始的领域对象非常有效，我甚至在不知道题目的时候，就已经大致知道这段程序是和不同金额的硬币相关的。
- 虽然已经尽力起了有意义的名称，但是名字太像了。
  `makeChangesWithoutCurrentType`，`makeChangesWithCurrentType`， `makeChangesWith`， `makeChangesWith`
总之我是晕了。
- 因为测试没有一起贴出来，造成另一个问题。不知道这几个函数谁是入口，这段代码是怎么使用的。

把我的这些不成熟意见说给张云雷后，还好他并没有对我的理解能力提出质疑，而是颇有同感的说他自己也对命名不满意，只是一时没想到怎么更好的表达。并请我重构一下试试。

但是，自己都不是很明白的问题，看起来有点复杂的代码，其实并不会用的语言。
怎么重构呢？


当然先加测试咯。

张云雷是用TDD写出这段代码的，当然是可以要到测试的。不过转念一想，其实这个情况和现实工作中接手代码有点像，并不一定完全理解，也不一定有测试。所以我决定自己先把测试补出来，一来理解问题和程序的行为，二来为后面的重构保驾护航。

####重构过程
Cyber Dojo用的多了以后，我发现仅仅观察每次执行测试后红绿灯的记录，就能发现某些模式，代表了逐步掌握代码的过程。
比如我重构这段代码的记录如下：
![重构红绿灯](http://upload-images.jianshu.io/upload_images/2453618-d3f4761bc3f775d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
1. 首先是让代码跑起来的过程。可以看到有较多的黄灯和红灯，标志着一开始的尝试。这时候很容易出现一些低级错误比如缺失头文件，文件命名错误等等。
这道题目是个单纯的练习，这个阶段很快就度过了。而对于真实的遗留代码，也许会有相当长的黄灯红灯循环，仅仅是为了能new出一个可以测试的目标类。 
1. 接下来是通过测试学习程序现有行为的过程。基本上就是写一个失败的测试，然后根据错误信息把实际的输出填写在测试期待值里。
由于这段程序的输入输出单纯，所以我偷懒一次写出一串数字，然后一个一个的改变期待值。如果真的一板一眼的做，这里应该是严格的红灯->绿灯->红灯->绿灯循环。
1. 有了测试保证后，下一个阶段就正式进入修改代码重构的阶段。如果注意方法，步子够小的话。会产生一长串绿灯，间或有一两个黄灯或红灯。比如我这样。^_^
1. 之后是比较有趣的部分。相当长一段时间的绿灯后。好像会觉得有些无聊。这时随着自信心的增长，会开始一些新的尝试，比如不熟悉的语言特性，或者改变基本的程序流程来达到更加可读或更简洁的目的。
这时有可能因为尝试，产生连续的黄灯红灯。而一旦突破了想要实现的转变，后续又会是一长串的绿灯。

####补出测试
托Cyber Dojo的福，不用做我最头疼的搭建环境工作，直接建立个C++的练习即可。
赫然发现原来有两种不同的C++可选，果然水很深。胡乱选一个，但愿对这段代码没区别。
然后是测试框架，也有好几个。还是胡乱选了第一个Boost。
把代码贴进去，改了几个文件命名错误，然后就一句一句贴出了如下的测试代码。
```
BOOST_AUTO_TEST_CASE(Count_Coins)
{
    BOOST_REQUIRE_EQUAL(0, makeChange(0));
    BOOST_REQUIRE_EQUAL(1, makeChange(1));

    BOOST_REQUIRE_EQUAL(2, makeChange(5));
    BOOST_REQUIRE_EQUAL(2, makeChange(7));

    BOOST_REQUIRE_EQUAL(4, makeChange(10));
    BOOST_REQUIRE_EQUAL(4, makeChange(13));

    BOOST_REQUIRE_EQUAL(6, makeChange(15));
    BOOST_REQUIRE_EQUAL(6, makeChange(18));

    BOOST_REQUIRE_EQUAL(9, makeChange(20));
    BOOST_REQUIRE_EQUAL(9, makeChange(22));

    BOOST_REQUIRE_EQUAL(13, makeChange(25));
    BOOST_REQUIRE_EQUAL(13, makeChange(29));

    BOOST_REQUIRE_EQUAL(18, makeChange(30));
    BOOST_REQUIRE_EQUAL(24, makeChange(35));
    BOOST_REQUIRE_EQUAL(31, makeChange(40));
    BOOST_REQUIRE_EQUAL(39, makeChange(45));
    BOOST_REQUIRE_EQUAL(49, makeChange(50));
    BOOST_REQUIRE_EQUAL(60, makeChange(55));
    BOOST_REQUIRE_EQUAL(73, makeChange(60));
}
```
这里是对应于那一长串的红灯。
由于偷懒，写成了在一个case里罗列一长串assert的样子，实在不是一种很好的风格。不过对于重构保护已经足够了。
就算这么无脑的尝试，也给了我一些启发。比如我发现每5分钱一个档次，当金额增长没有超过5时，输出是一样的。而每次增长了一档，就会多出好多个组合，而且随着金额越来越大，增幅也越来越大。

####第一版重构
主要是从以下角度对代码进行了改变。
- 尽一步凸显代码的意图，比如
![把函数名字改成它实际做的事情](http://upload-images.jianshu.io/upload_images/2453618-766574f8c8fb45e9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![给if条件一个名字](http://upload-images.jianshu.io/upload_images/2453618-247c31156fb3fc59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 修改了易于混淆的命名。把makeChange这个动作留给发起动作的入口函数，而把分别计算不同情况的函数命名为changeWaysWithxxx。
- 意外的发现有些没用的代码，愉快的删掉了。

这时相当于红绿灯图里第一个长串绿灯结束的位置。代码变成了这样
```
#include "countCoins.hpp"
enum class CoinsType:int{
    PENNY=1,
    NICKEL=5,
    DIME=10,
    QUARTER=25
} ;
 
int operator *(int value, CoinsType type)
{
    return value * static_cast<int>(type);
}
 
CoinsType operator -- (CoinsType type, int)
{
    switch (type){
        case CoinsType::QUARTER:
            return CoinsType::DIME;
        case CoinsType::DIME:
            return CoinsType::NICKEL;
        case CoinsType::NICKEL:
            return CoinsType::PENNY;
        default:
            return CoinsType::PENNY;
    }
}
 
const CoinsType BIGGEST_COIN = CoinsType::QUARTER;
bool noSmallerCoin(CoinsType type) {
    return type == CoinsType::PENNY;
}
 
int makeChange(int cents)
{
    if( cents == 0 ) {
        return 0;
    }
    return changeWaysWithCoinOrSmaller(cents, BIGGEST_COIN);
}
 
int changeWaysWithCoinOrSmaller(int cents, CoinsType type)
{
    if( noSmallerCoin(type) )
    {
        return 1;
    }
    int ways = 0;
    ways += changeWaysWithCoin(cents, type);    
    ways += changeWaysWithCoinOrSmaller(cents, type--);
    return ways;
}
 
int changeWaysWithCoin(int cents, CoinsType type)
{
    int ways = 0;
    for( int i = 1; i * type <= cents ; i++)
    {
        int remaining = cents - i * type;  
        ways += changeWaysWithCoinOrSmaller(remaining, type--);
    }
    return ways;
}
 ```
有没有好一点？
好吧也许看起来变化不大，但是至少短了一点吧。

####第二版重构
由于第一版重构后感觉不过瘾，之后又进行了第二版重构。
这次重构主要是围绕着`changeWaysWithCoin`这个函数进行的，基于以下几点考虑
- 本来仅仅是为了取巧，把循环条件里判断金额除以硬币面额，和循环体内每次取出若干个硬币金额的代码。统一成了i乘以面额。这样可以少定义一个除法操作。
结果意外的发现，其实这个循环相当于每次去掉一枚硬币，只和剩余金额有关。所以这段逻辑可以简化为每次取出一枚硬币面额直到剩余的金额小于面额为止。不需要一个循环次数作为噪音。
- 进一步思索，发现这种循环可以改变成递归。由于程序的总体流程是递归，所以一个局部的循环实际上在代码中增加了行为范式，增加了复杂度。
- 去掉了循环之后，这个函数的职责更加清楚了。从而带来了一个更有意义的名字`waysTakeOneCoinAway`。
附带的效果是，原本`changeWaysWithCoinOrSmaller`这个名字显得十分笨拙。现在它的名字就像是代码的自然描述，显得一目了然。

![名副其实](http://upload-images.jianshu.io/upload_images/2453618-e5fb85304d2520d1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终代码如下：
```
#include "countCoins.hpp"
enum class CoinsType:int{
    PENNY=1,
    NICKEL=5,
    DIME=10,
    QUARTER=25
} ;
 
int operator -(int value, CoinsType type)
{
    return value - static_cast<int>(type);
}
 
CoinsType operator -- (CoinsType type, int)
{
    switch (type){
        case CoinsType::QUARTER:
            return CoinsType::DIME;
        case CoinsType::DIME:
            return CoinsType::NICKEL;
        case CoinsType::NICKEL:
            return CoinsType::PENNY;
        default:
            return CoinsType::PENNY;
    }
}
 
const CoinsType BIGGEST_COIN = CoinsType::QUARTER;
bool noSmallerCoin(CoinsType type) {
    return type == CoinsType::PENNY;
}
 
int makeChange(int cents)
{
    return cents == 0 ? 0 : changeWaysWithCoinOrSmaller(cents, BIGGEST_COIN);
}
 
int waysTakeOneCoinAway(int cents, CoinsType coin)
{
    int afterTaken = cents - coin;
    return afterTaken < 0 ? 0 : afterTaken == 0 ? 1 :
            changeWaysWithCoinOrSmaller(afterTaken, coin);
}
 
int waysWithSmallerCoin(int cents, CoinsType coin)
{
    return noSmallerCoin(coin) ? 0 : changeWaysWithCoinOrSmaller(cents, coin--);
}
 
int changeWaysWithCoinOrSmaller(int cents, CoinsType coin)
{
    return waysWithSmallerCoin(cents, coin) + waysTakeOneCoinAway(cents, coin);
}
```
整个过程在
http://cyber-dojo.org/kata/edit/DBEB923C89?avatar=tuna

####心得
这不是第一次在群里与张云雷交换对代码的看法了。
我发现一个有趣的现象，有几次看云雷重构我的代码，对函数的重命名让我十分惊艳。我还感叹云雷特别擅长起名字。
结果这次反过来了，他看完我的重命名后说这就是他想表达的意思，只是写的时候找不到合适的表达方式。
看来也许并不完全是能力问题，重构和review给了我们另一个理解代码的角度。这大概就是结对的意义所在吧。
