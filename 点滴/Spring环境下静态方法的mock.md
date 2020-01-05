![](https://upload-images.jianshu.io/upload_images/2453618-3b4d03a901b8f5c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其实我不是很理解为什么用了Spring还需要使用静态方法来提供单例之类的服务。也许是传承自较老的代码，也许对Spring对象的生命周期不太肯定，也许要与不属于Spring上下文的代码共享逻辑。总之现实常常还是能看到这种代码的。
```java
/**
*提供静态方法的类
*/
public class PersonPool {
    private static PersonPool instance;
    static {
        instance = new PersonPool();
    }
    private PersonPool() {
        // ...
    }
    public static PersonPool getInstance() {
        return instance;
    }
    public String next() {
        throw new RuntimeException("resource not available in test");
    }
}
/**
*使用静态方法的服务
*/
public class SampleService {
    public String greeting() {
        String name = PersonPool.getInstance().next();
        return "Hello " + name;
    }
}
```
一旦这些静态方法中用到了环境相关，需要在测试中隔离的资源，写单元测试就时就会遇到困难。
这类代码是无法用mockito来mock的。PowerMock可以mock静态方法，不过使用中会发现会给测试引入更多的细节，这些细节都是妨碍表达测试意图的噪音。
而且往往mock掉方法本身，又发现类初始化有个坑……，一点点解决下去，最后测试凑合能跑起来，但是充满了各种和实现紧密耦合的细节。实现代码里两行换个顺序，也许测试就不工作了。

其实这种处境，往往是因为抱着**~~“不能为了测试修改实现代码”~~**的信念造成的。
如果抛开这个假设，对代码稍作修改，测试就不再是个问题了。
```java
/**
*提供静态方法的类保持不变
*/
public class PersonPool {...}
/**
*增加适配类
*/
@Component
public class PersonPoolProvider {
    public PersonPool getInstance() {
        return PersonPool.getInstance();
    }
}
/**
*改用自动注入的适配对象
*/
public class SampleService {
    @Autowired
    PersonPoolProvider personPoolProvider;

    public String greeting() {
        String name = personPoolProvider.getInstance().next();
        return "Hello " + name;
    }
}
```
可以看到只需要对实现代码进行影响很小的修改：
- 未改变原有静态方法的执行逻辑和生命周期。
- 不影响其它使用静态方法的代码。
- 适配类足够简单，无需测试。


希望这不是个为遗留代码补测试的权宜之计，而是进一步改进已有代码的开端。
有以下问题可以探讨：
- util类、方法是否应该有某种资源的所有权？
- 现实中单个存在的事物是否就一定用单例实现？
- 使用单例时，是在用它“单个实例”的保证，还是在利用可以从类名获得实例的“可访问性”？
