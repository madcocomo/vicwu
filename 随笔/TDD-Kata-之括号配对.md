新年将至，微信里满屏都是年会和中奖，透露着越来越近的节日气氛。
小波同学在TDD群里抛出了一道题。
> 来抛个题，想了一晚上没做出来

题目是Balanced Parentheses，英文描述就不贴了。说起来挺简单，
> 一个字符串，判断它里面的括号是否配对
比如：
(5+6)∗(7+8)/(4+3)
是配对的，而
:-)
())(
则不是

顺便一提，在cyber-dojo.org上有这道题，而且是升级版。

不知道是不是因为每个程序员或多或少都有过人肉数括号的烦恼，还是小波声称的一晚上没做出来带来的激励效果。一下在群里引发了热议。
就在周师傅还在遗憾手边没电脑，正琢磨着是不是要手机上TDD的时候。鄢倩已经贴出了第一份代码
![鄢倩 Clojure](http://upload-images.jianshu.io/upload_images/2453618-ef7c3f6ceefd490b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

紧随其后的是没有尾巴
![没有尾巴跑得快 Scala](http://upload-images.jianshu.io/upload_images/2453618-e59732dc91bedfe4.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

申导两连发
![申导 人生苦短，我用Python可以写两遍](http://upload-images.jianshu.io/upload_images/2453618-1c0457cc6144fd2f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

代码好多，美中不足的是只有结果，没看到产生的过程。周师傅终于拿到电脑贴出了TDD记录：
[http://cyber-dojo.org/kata/edit/6320CF1239?avatar=hippo](http://cyber-dojo.org/kata/edit/6320CF1239?avatar=hippo)
顺便给还没机会用上函数式语言的同学带来些许安慰。
```
public class BalancedParentheses {
    public boolean valid(String exp) {
        return valid(exp, 0);
    }

    private boolean valid(String exp, int balance) {
        if (exp.isEmpty())
            return isBalanced(balance);
        if (isLeftParentheses(headChar(exp)))
            return valid(tailOf(exp), balance + 1);
        if (isRightParentheses(headChar(exp)) && isBalanced(balance))
            return false;
        if (isRightParentheses(headChar(exp)))
            return valid(tailOf(exp), balance - 1);
        return valid(tailOf(exp), balance);    
    }

    private boolean isLeftParentheses(char c) {
        return c == '(';
    }

    private boolean isRightParentheses(char c) {
        return c == ')';
    }

    private boolean isBalanced(int balance) {
        return balance == 0;
    }

    private char headChar(String exp) {
        return exp.charAt(0);
    }

    private String tailOf(String exp) {
        return exp.substring(1);
    }
}
```

回答太过热烈，没有尾巴同学忍不住提高难度，挑战升级版。
> 如何修改代码能够同时支持[] () 和{}
要考虑这样的用例 [(])

但是这样的程度显然不够看，瞬间被秒
![还是Python](http://upload-images.jianshu.io/upload_images/2453618-1beb2801d46cc083.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

![还是Clojure](http://upload-images.jianshu.io/upload_images/2453618-9edfdfe4c6010d64.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1080/q/50)

一直在围观，似乎对+1 -1非常在意的麦宇安老师。终于出手了。弄出了一个没有加减一，又不压栈的版本，说是什么用函数组合来代替栈或者计数。
具体来说就是…… 
好吧，算了还是自己看吧。
![麦宇安 F#](http://upload-images.jianshu.io/upload_images/2453618-87b41863d3507bad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

---
更新的分割线
大概是函数组合过于华丽，周师傅忍不住按照这个思路做了一遍，而且是用没有函数式特性的JAVA。
```
public class BalancedParentheses {
    public boolean matchBalancedParentheses(String exp) {
        if (hasLeftParentheses(exp))
            return matchRightParentheses(afterLeftParentheses(exp)); 
        if (hasRightParentheses(exp))
            return false;
        return true;
    }
 
    private boolean matchRightParentheses(String exp) {
        if (hasLeftParentheses(exp))
            return matchBalancedParentheses(exp) &&
                   matchRightParentheses(afterRightParentheses(exp));
        if (hasRightParentheses(exp))
            return true;
        return false;
    }
 
    private String afterLeftParentheses(String exp) {
        return exp.substring(exp.indexOf('(')+1);
    }
 
    private String afterRightParentheses(String exp) {
        return exp.substring(exp.indexOf(')')+1);
    }
 
    private boolean hasLeftParentheses(String exp) {
        return exp.indexOf('(') >= 0;
    }
 
    private boolean hasRightParentheses(String exp) {
        return exp.indexOf(')') >= 0;
    }
}
```
看起来很像吧，然而其实有着本质的不同。
而且，随后申导发现了一个失败的测试用例。
"I told him (that not (yet)done).(but he)"
经过一长串的修改后，JAVA版函数式终于闪亮登场
```
import java.util.function.Predicate;
 
public class BalancedParentheses {
    public boolean matchBalancedParentheses(String exp) {
        if (hasLeftParentheses(exp))
            return matchRightParentheses(afterLeftParentheses(exp), 
                   nextExp -> matchBalancedParentheses(nextExp));
        if (hasRightParentheses(exp))
            return false;
        return true;
    }
 
    private boolean matchRightParentheses(String exp, Predicate<String> nextStep) {
        if (hasLeftParentheses(exp))
            return matchRightParentheses(afterLeftParentheses(exp), 
                   nextExp -> matchRightParentheses(nextExp, nextStep));
        if (hasRightParentheses(exp))
            return nextStep.test(afterRightParentheses(exp));        
        return false;
    }
 
    private String afterLeftParentheses(String exp) {
        return exp.substring(exp.indexOf('(')+1);
    }
 
    private String afterRightParentheses(String exp) {
        return exp.substring(exp.indexOf(')')+1);
    }
 
    private boolean hasLeftParentheses(String exp) {
        return exp.indexOf('(') >= 0 && (exp.indexOf(')') == -1 || 
               exp.indexOf('(') < exp.indexOf(')'));
    }
 
    private boolean hasRightParentheses(String exp) {
        return exp.indexOf(')') >= 0;
    }
}
```
http://cyber-dojo.org/kata/edit/677B72062C?avatar=crab

最后是我的作弊解法。话说用语言自带的库作弊，也算作弊么？ [汗]……
```
import scala.util.parsing.combinator.RegexParsers

class ParenthesesParser extends RegexParsers {
  def part: Parser[Any] = "[^()]+".r
  def factor: Parser[Any] = part | "("~expr~")"
  def expr: Parser[Any] = rep(factor)
}

  //Test
  "expr" should "accept parentheses in row" in {
    parser.parseAll(parser.expr, "").successful shouldBe true
    parser.parseAll(parser.expr, "(abc)(12)").successful shouldBe true
    parser.parseAll(parser.expr, "(abc)12)").successful shouldBe false
    parser.parseAll(parser.expr, "(abc(123)*(12))").successful shouldBe true
    parser.parseAll(parser.expr, "I told him (that not (yet)done).(but he)").successful shouldBe true
  }
```

---
如果你也是这么一个习惯解题，磨练编程技巧，对TDD充满好奇或是热情的程序员。
快到群里来吧。
