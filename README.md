# 单元测试
### [JUnit4使用](./Junit4使用.md)
### [Mock](./Mock.md)
### [Robolectric](./Robolectric.md)
### [依赖注入](./依赖注入.md)
### [单元测试范围](./单元测试范围.md)
### [代码覆盖率](./代码覆盖率.md)
### [测试基础](./测试基础.md)

## 简介
在计算机编程中，单元测试（英语：Unit Testing）又称为模块测试, 是针对程序模块（软件设计的最小单位）来进行正确性检验的测试工作。 程序单元是应用的最小可测试部件。
最小单位理解为一个类的方法。
我们举一个例子说明一下，假如你有一个类，定义如下：

		
		public class Calculator {
		    public int add(int one, int another) {
		        //为了简单起见，暂不考虑溢出等情况。
		        return one + another;
		    }
		}
		
那么为了测试这个Calculator类的add()方法，我们可以写如下的单元测试代码：

	public class CalculatorTest {
	    public void testAdd() throws Exception {
	        Calculator calculator = new Calculator();
	        int sum = calculator.add(1, 2);
	        Assert.assertEquals(3, sum);
	    }
	}

这里的CalculatorTest是Calculator对应的测试类。而这里的testAdd()就是add()这个方法对应的测试方法。
一般来说，一个方法对应的测试方法主要分为3部分，以上面的测试方法为例：

* setup。一般是new出你要测试的那个类，以及其他一些前提条件的设置：Calculator calculator = new Calculator();
* 执行操作。一般是调用你要测试的那个方法，获得运行结果：int sum = calculator.add(1, 2);
* 验证结果。验证得到的结果跟预期中是一样的：Assert.assertEquals(3, sum);
 
** 单元测试不是集成测试 **

这里需要强调一个观念，那就是单元测试只是测试一个方法单元，它不是测试一整个流程。
举个例子来说，一个Login页面，上面有两个输入框和一个button。两个输入框分别用于输入用户名和密码。点击button以后，有一个UserManager会去执行performlogin操作，然后将结果返回，更新页面。
那么我们给这个东西做单元测试的时候，不是测这一整个login流程。这种整个流程的测试：给两个输入框设置正确的用户名和密码，点击login button, 最后页面得到更新。叫做集成测试，而不是单元测试。
当然，集成测试也是有他的必要性的，然而这不是我们每个程序员应该花多少精力所在的地方。在这方面，有一个理论叫做Test Pyramid，如下图所示：
![](http://7xod3k.com1.z0.glb.clouddn.com/qtijqabixtlihxsuujkwnlzelrqnwqnz)
 
## 为啥要做单元测试
* 快速反馈bug，跑一遍单元测试用例，定位bug
* 设别依赖
* 回归验证

 
## Android单元测试需要用到哪些技术
JUnit4 + Mockito + Dagger2 + Robolectric。

Mockito是一种mock框架，目前安卓最常用的有两个，Mockito和JMockit。

mock就是创建一个虚假的、模拟的对象。在测试环境下，用来替换掉真实的对象。这样就能达到两个目的：

1. 可以随时指定mock对象的某个方法返回什么样的值，或执行什么样的动作。
2. 可以验证mock对象的某个方法有没有得到调用，或者是调用了多少次，参数是什么等等。
接下来的一个问题就是，如何在测试环境下，把DataModel换成mock的对象，而正式代码中，DataModel又是正常的对象呢？
这个问题也有两种解决方案，一是使用专门的testing product flavor；二是使用依赖注入。
 
先简单介绍一下依赖注入这个模式，他的基本理念是，某一个类（比如说DataActivity），用到的内部对象（比如说DataModel）的创建过程不在DataActivity内部去new，而是由外部去创建好DataModel的实例，然后通过某种方式set给DataActivity。
Dagger2就是一种依赖注入框架。
Android单元测试最大的痛点，那就是JVM上面运行纯JUnit单元测试时是不能使用Android相关的类的，因为我们开发用到的安卓环境是没有实现的，里面只定义了一些接口，所有方法的实现都是throw new RuntimeException("stub");，如果我们单元测试代码里面用到了安卓相关的代码的话，那么运行时就会遇到RuntimeException("Stub")。

要解决这个问题，一般来说有三种方案：

1. 使用Android提供的Instrumentation系统，将单元测试代码运行在模拟器或者是真机上。
2. 用一定的架构，比如MVP等等，将安卓相关的代码隔离开了，中间的Presenter或Model是存java实现的，可以在JVM上面测试。View或其他android相关的代码则不测。
3. 使用Robolectric框架，这个框架基本可以理解为在JVM上面实现了一套安卓的模拟环境，同时给安卓相关的类增加了其他一些增强的功能，以方便做单元测试，使用这个框架，我们就可以在JVM上面跑单元测试的时候，就可以使用安卓相关的类了。

## 单元测试原则，FIRST
F——Fast：快速
在调试bug时，需要频繁去运行单元测试验证结果是否正确。如果单元测试足够快速，就可以省去不必要浪费的时间，提高工作效率。

I——Isolated：隔离

R——Repeatable：可重复
单元测试需要保持运行稳定，每次运行都需要得到同样的结果，如果间歇性的失败，会导致我们不断的去查看这个测试，不可靠的测试也就失去了意义。

S——Self-verifying：自我验证

T——Timely：及时