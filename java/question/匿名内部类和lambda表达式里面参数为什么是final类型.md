###### <center>匿名内部类和lambda表达式里面参数为什么是final类型?</center>

&ensp;&ensp;&ensp;&ensp;在写代码的过程中经常会遇到匿名内部类和lambda表达式里面参数都必须用final类型的变量，即使偶而可以不加final也只是这个变量只有在内部类中使用的情况下(这个情况在编译的时候会在class中自动加上final，下面会讲到)。
&ensp;&ensp;&ensp;&ensp;其实在方法里面或lambda表达式里面都是匿名内部类，实质上都是接口的实现类。
&ensp;&ensp;&ensp;&ensp;具体看代码，为了更好的说明问题lambda表达式都用内部类来表示，如下：
```java
/**
 * @author Administrator
 * @date 2019/2/14
 */
interface TestA{
    void testA();
}
class InnerClass {
    public void test1(TestA a){
        a.testA();
    }
}

class TestMain{
    public static void main(String[] args) {
        InnerClass innerClass = new InnerClass();
        List<String> testStrings = new ArrayList<>();
        //只在内部类里面使用可以省略final（但其实它是final类型的）
        int a = 1;
        Integer[] b = {1};
        AtomicInteger c = new AtomicInteger(1);
        innerClass.test1(new TestA() {
            @Override
            public void testA() {
                    b[0] = 3;
                    b[0] = a;
            }
        });
        testStrings.forEach(new Consumer<String>() {
            @Override
            public void accept(String s) {
                b[0] = 3;
                c.set(2);
                c.set(a);
            }
        });
    }
}
```
以上代码用javac编译成class文件如下。
```

/**
* TestA接口->TestA.class
*/

interface TestA {
    void testA();
}
/**
* InnerClass.java -> InnerClass.class
*/

class InnerClass {
    InnerClass() {
    }

    public void test1(TestA var1) {
        var1.testA();
    }
}

/**
* TestMain.java -> TestMain.class  TestMain&1.class TestMain$2.InnerClass
* 其中TestMain&1.class TestMain$2.InnerClass 这两个类是 TestMain.java中两个匿名
* 内部类编译出来的class文件。
*/
class TestMain {
    TestMain() {
    }

    public static void main(String[] var0) {
        InnerClass var1 = new InnerClass();
        ArrayList var2 = new ArrayList();
        //如果在原始的java文件中没用final修饰，这里会自动加上
        final byte var3 = 1;
        final Integer[] var4 = new Integer[]{1};
        final AtomicInteger var5 = new AtomicInteger(1);
        //这里的TestA的实现实质上是TestMain&1类。
        var1.test1(new TestA() {
            public void testA() {
                var4[0] = 3;
                var4[0] = Integer.valueOf(var3);
            }
        });
        //这里的Consumer的实现实质上是TestMain&2类。
        var2.forEach(new Consumer<String>() {
            public void accept(String var1) {
                var4[0] = 3;
                var5.set(2);
                var5.set(var3);
            }
        });
    }
}
/**
* 编译的时候会自动生成final的实现
* 类
*/
final class TestMain$1 implements TestA {
    //这里我们可以看到，在new TestA 中使用到的两个变量Integer[] b，int a
    //的引用被拷贝到TestMain$1实现类中来使用
    TestMain$1(Integer[] var1, int var2) {
        this.val$b = var1;
        this.val$a = var2;
    }

    public void testA() {
        this.val$b[0] = 3;
        this.val$b[0] = this.val$a;
    }
}

final class TestMain$2 implements Consumer<String> {
    TestMain$2(Integer[] var1, AtomicInteger var2, int var3) {
        this.val$b = var1;
        this.val$c = var2;
        this.val$a = var3;
    }

    public void accept(String var1) {
        this.val$b[0] = 3;
        this.val$c.set(2);
        this.val$c.set(this.val$a);
    }
}
```
>&ensp;&ensp;&ensp;&ensp;由上面的编译文件可以知道，在编译的时候匿名内部类会自动编译成一个final的实现类并把所需要的共享变量的引用拷贝到实现类里面供自已使用（在内部类中的属性和外部方法的参数两者从外表上看是同一个东西，但实际上却不是，所以他们两者是可以任意变化的，也就是说在内部类中对属性的改变并不会影响到外部类中的变量）,假如在匿名内部类的内部可以改变外部的引用变量，那么在匿名内部类里面的变量值就和外部类中的变量值不一样了，这样就违背了内部类可以访问外部类变量的规则了，这个是不允许的。
>注：这里的外部的变量指的是内部类所在方法的局部变量，并不是外部类里面的全局变量。外部类里面的全局变量可以不用为final，因为全局变量和内部类可见性处于同一等级。
