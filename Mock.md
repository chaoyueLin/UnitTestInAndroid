# Mock
 
## 什么是mock
 
谓的mock就是创建一个类的虚假的对象，在测试环境中，用来替换掉真实的对象，以达到两大目的：
 
验证这个对象的某些方法的调用情况，调用了多少次，参数是什么等等
 
指定这个对象的某些方法的行为，返回特定的值，或者是执行特定的动作
 
要使用Mock，一般需要用到mock框架，这Mockito这个是Java界使用最广泛的一个mock框架。
 
## Mockito
 
使用
 
* 1.Mockito.mock();
 
* 2.set进去
 
* 3.验证Mockito.verify()

 
	    public class LoginPresenter {
	 
	        private UserManager mUserManager = new UserManager();
	 
	        public void login(String username, String password) {
	            if (username == null || username.length() == 0) return;
	            if (password == null || password.length() < 6) return;
	            mUserManager.performLogin(username, password);
	 
	        }
	 
	        public void setUserManager(UserManager userManager) {  //<==
	 
	            this.mUserManager = userManager;
	        }
	 
	    }
	 
	 
	    @Test
	 
	    public void testLogin() throws Exception {
	 
	        UserManager mockUserManager = Mockito.mock(UserManager.class);
	        LoginPresenter loginPresenter = new LoginPresenter();
	        loginPresenter.setUserManager(mockUserManager);  //<==
	        loginPresenter.login("xiaochuang", "xiaochuang password");
	        Mockito.verify(mockUserManager).performLogin("xiaochuang", "xiaochuang password");
	 
	    }
	 

 
## 其他验证
 
* 1.方法返回特定的值
 
	    Mockito.when(mockObject.targetMethod(args)).thenReturn(desiredReturnValue);
	 
	    //先创建一个mock对象
	    PasswordValidator mockValidator = Mockito.mock(PasswordValidator.class);
	 
	    //当调用mockValidator的verifyPassword方法，同时传入"xiaochuang_is_handsome"时，返回true
	    Mockito.when(mockValidator.verifyPassword("xiaochuang_is_handsome")).thenReturn(true);
	 
	    //当调用mockValidator的verifyPassword方法，同时传入"xiaochuang_is_not_handsome"时，返回false
	 
	    Mockito.when(validator.verifyPassword("xiaochuang_is_not_handsome")).thenReturn(false);
 
 
* 2.执行某个特定的回调
 
 
	    public void loginCallbackVersion(String username, String password) {
	        if (username == null || username.length() == 0) return;
	        //假设我们对密码强度有一定要求，使用一个专门的validator来验证密码的有效性
	        if (mPasswordValidator.verifyPassword(password)) return;
	        //login的结果将通过callback传递回来。
	        mUserManager.performLogin(username, password, new NetworkCallback() {  //<==
	 
	            @Override
	 
	            public void onSuccess(Object data) {
	                //update view with data
	            }
	 
	            @Override
	 
	            public void onFailure(int code, String msg) {
	                //show error msg
	            }
	 
	        });
	 
	    }
	 
	 
	 
	    Mockito.doAnswer(new Answer() {
	 
	        @Override
	 
	        public Object answer(InvocationOnMock invocation) throws Throwable {
	 
	            //这里可以获得传给performLogin的参数
	            Object[] arguments = invocation.getArguments();
	            //callback是第三个参数
	            NetworkCallback callback = (NetworkCallback) arguments[2];
	            callback.onFailure(500, "Server error");
	            return 500;
	 
	        }
	 
	    }).when(mockUserManager).performLogin(anyString(), anyString(), any(NetworkCallback.class));
 
 
 
 
## Spy
 
同样也是Mock,只是有默认实现
 
对于第二大功能: 指定方法的特定行为，如果不指定的话，一个mock对象的所有非void方法都将返回默认值：int、long类型方法将返回0，boolean方法将返回false，对象方法将返回null等等；而void方法将什么都不做。
 
Spy就有默认实现，又能验证
 
 
    //假设目标类的实现是这样的
 
    public class PasswordValidator {
 
        public boolean verifyPassword(String password) {
            return "xiaochuang_is_handsome".equals(password);
 
        }
    }
 
    @Test
 
    public void testSpy() {
 
        //跟创建mock类似，只不过调用的是spy方法，而不是mock方法。spy的用法
        PasswordValidator spyValidator = Mockito.spy(PasswordValidator.class);
        //在默认情况下，spy对象会调用这个类的真实逻辑，并返回相应的返回值，这可以对照上面的真实逻辑
        spyValidator.verifyPassword("xiaochuang_is_handsome"); //true
 
        spyValidator.verifyPassword("xiaochuang_is_not_handsome"); //false

        //spy对象的方法也可以指定特定的行为
 
        Mockito.when(spyValidator.verifyPassword(anyString())).thenReturn(true);
 
        //同样的，可以验证spy对象的方法调用情况
 
        spyValidator.verifyPassword("xiaochuang_is_handsome");
 
        Mockito.verify(spyValidator).verifyPassword("xiaochuang_is_handsome"); //pass
 
    }
 
Mockito 不能 mock静态方法,  对final类、匿名类、java的基本数据类型也是无法进行mock或者spy的
PowerMock基本上cover了所有Mockito不能支持的case。它仅支持EasyMock和Mockito，我们使用时可以把它作为Mockito的拓展。


Mock 方法内部 new 出来的对象

	PowerMockito.whenNew(MockClass.class).withArguments(someArgs).thenReturn(expectedObject);

Mock 静态方法

	PowerMockito.mockStatic(Class clazz);
mock 私有方法

	PowerMockito.when(mockObject, “privateMethodName”).thenReturn(true);
Mock 私有属性

	Whitebox.setInternalState(mockObject, “privateFiledName”, expected);

## Mockk
MockK 是一个用 Kotlin 写的 Mocking 框架。在kotlin环境下, 建议针对Kotlin单元测试的mock，使用Mockk框架。Mockk和mockito使用上差异不大, 只是语法上更为接近英文语法。

1，mock 对象

	mockk<Class>(),
	mockkObject（类似针对Companion object， 单例类(object)，可以调用mockObject 来实现mock

2，mock行为

      every { mother.giveMoney() } returns 30  — 返回某个值
      every { mother.inform(any()) } just Runs   — 为这个没有返回值的方法分配一个默认行为。
      answers{ }  — 执行某个语句块  
3，Verify

      verify { mother.inform(any()) }
      verify(exactly = 10) { mother.inform(any()) }
4，mockkStatic

     mockkStatic(StaticClass::class)
