# JUnit4使用
## 在Android项目里面使用JUnit
在Android项目里面使用JUnit是很简单的，你只需要将JUnit这个library加到你的dependencies里面。
```
testCompile 'junit:junit:4.12'
```
 
1.setup
2.执行操作
3.验证结果
@Test
@Before
@After
@Ignore
@Test(expected = IllegalArgumentException.class)
 
## 断言的使用Assert
assertEquals(expected, actual)
验证expected的值跟actual是一样的，如果是一样的话，测试通过，不然的话，测试失败。如果传入的是object，那么这里的对比用的是equals()
 
assertEquals(expected, actual, tolerance)
这里传入的expected和actual是float或double类型的，大家知道计算机表示浮点型数据都有一定的偏差，所以哪怕理论上他们是相等的，但是用计算机表示出来则可能不是，所以这里运行传入一个偏差值。如果两个数的差异在这个偏差值之内，则测试通过，否者测试失败。
 
assertTrue(boolean condition)
验证contidion的值是true
 
assertFalse(boolean condition)
验证contidion的值是false
 
assertNull(Object obj)
验证obj的值是null
 
assertNotNull(Object obj)
验证obj的值不是null
 
assertSame(expected, actual)
验证expected和actual是同一个对象，即指向同一个对象
 
assertNotSame(expected, actual)
验证expected和actual不是同一个对象，即指向不同的对象
 
fail()
让测试方法失败
 
注意：上面的每一个方法，都有一个重载的方法，可以在前面加一个String类型的参数，表示如果验证失败的话，将用这个字符串作为失败的结果报告。
比如：
assertEquals("Current user Id should be 1", 1, currentUser.id());
当currentUser.id()的值不是1的时候，在结果报道里面将显示"Current user Id should be 1"，这样可以让测试结果更具有可读性，更清楚错误的原因是什么。
 
## JUnit Rule
一个JUnit Rule就是一个实现了TestRule的类，这些类的作用类似于@Before、@After，是用来在每个测试方法的执行前后执行一些代码的一个方法。
自带使用
```
@Rule
public Timeout timeout = new Timeout(1000);
```
扩展
```
    public class MethodNameExample implements TestRule {
        @Override
        public Statement apply(final Statement base, final Description description) {
            return new Statement() {
                @Override
                public void evaluate() throws Throwable {
                    //想要在测试方法运行之前做一些事情，就在base.evaluate()之前做
                    String className = description.getClassName();
                    String methodName = description.getMethodName();
 
                    base.evaluate();  //这其实就是运行测试方法
 
                    //想要在测试方法运行之后做一些事情，就在base.evaluate()之后做
                    System.out.println("Class name: "+className +", method name: "+methodName);
                }
            };
        }
    }
 
    @Rule
    public MethodNameExample methodNameExample = new MethodNameExample();
```