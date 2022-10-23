# Powermock CheatSheet: Tldr and Tips

### Why we need Powermock?

Although Mockito is most popular Mocking Library for Java Developers, But there are certain cases where Simple Mockito does not work. For Example, if your codebase uses any Singleton, you cannot mock them with Simple Mockito.

There are many other use cases, such as:
1. Mock Static Methods
2. Mock Final Classes and Methods
3. Inject mock for object creation with `whenNew()`
4. `suppress()`, `stub()` or `replace()` any method of a Class.
5. Suppress static initialisation.
6. Invoke private methods.
7. Inject field with `Whitebox`


### 1. PowerMockRunner
You must annotate each class you test with `PowerMockRunner`
```Java
@RunWith(PowerMockRunner.class)
public class ClassToTest {
}
```

### 2. PrepareForTest
Each class you add in `@PrepareForTest` will generate an different bytecode that can be manipulated by PowerMock.
```Java
@RunWith(PowerMockRunner.class)
@PrepareForTest({
    StaticClass.class,
    FinalClass.class,
    ClassWithFinalMethod.class
})
public class ClassToTest {
}
```
PreparingForTest unlocks class for a wide variety of things such as mocking final classes, classes with final, private, static or native methods that should be mocked and also classes that should be return a mock object upon instantiation.

You can read more about what `@PrepareForTest` unlocks from [here](Link)

### 3. SuppressStaticInitializationFor
If you are working with classes which have static initializers such as:
```java
package com.example;

public class ClassWithStaticInitializer {
        private static Handler mHandler = new Handler();
}
```
As `mHandler` belongs to `ClassWithStaticInitializer.class` not instance of `ClassWithStaticInitializer`, JVM will initialize this `mHandler` as soon as execution starts, thus we will not get a chance to mock it with `whenNew()`. In this case, we can suppress its initialization and later set its value with `Whitebox`.

Here is an example to suppress `mHandler` in `ClassWithStaticInitializer.class`:
```Java
@RunWith(PowerMockRunner.class)
@SuppressStaticInitializationFor({
        "com.example.ClassWithStaticInitializer"
})
public class ClassWithStaticInitializerTest {
        // now when we call
        @Before
        public void setup throws Exception {
                // At this stage, ClassWithStaticInitializer.mHandler will be null
                Whitebox.setInternalState(ClassWithStaticInitializer.class, "mHandler", mckHandler);
        }
}
```
> **tip:** if you see any error with StackTrace containing similar looking line `com.examlpe.ExampleClass.<clinit>`. It means there is an static initializer, just add `com.examlpe.ExampleClass` in your `SuppressStaticInitializationFor` annotation.

### 4. PowerMockito.suppress()
`suppress()` is one of the most powerful API's provided by PowerMock.

With `suppress()` you can manipulate bytecode to skip executing any method or constructor.

For example, lets say you had an class which extended an library or code that you don't own, you can suppress its constructor.

```Java
// Example Source Class
public class ExampleClass extends CustomLibClass {
      public ExampleClass() {
            System.out.println("class initialised");
      }

      public void someMethod() {
            super.someMethod();
      }
}
```
Here, our source class extends `CustomLibClass` which we do not need to test. But creating a object of `ExampleClass` will also call constructor of `CustomLibClass`. This can be avoided if we tell Powermock to suppress constructor of `CustomLibClass`.
```Java
// Test of Example Source Class
@RunWith(PowerMockRunner.class)
@PrepareForTest({
    ExampleClass.class
})
public class ExampleClassTest {

    private ExampleClass classUnderTest;

    @Before
    public void setUp() throws Exception {
        // We should suppress constructor of CustomLibClass
        // There are multiple ways to do that
        // 1. Suppress with matching constructor, in below case it will only suppress constructor without any parameter
        PowerMockito.suppress(MemberMatcher.constructor(CustomLibClass.class));
        // Or using Java Reflection
        PowerMockito.suppress(CustomLibClass.class.getConstructor());
        // 2. Suppress all declared constructor
        PowerMockito.suppress(MemberMatcher.constructorsDeclaredIn(CustomLibClass.class));
        // Or Using Java Reflection API's
        PowerMockito.suppress(CustomLibClass.class.getConstructors());

        // Create constructor of ExampleClass, it will not call constructor of CustomLibClass
        classUnderTest = new ExampleClass();
    }

	@Test
	public void someMethod_shouldDoNothing() {
		// This is just an example, it is not necessary for an example test to test something :D
		PowerMockito.suppress(CustomLibClass.class.getDeclaredMethod("someMethod"));
	}
}
```
> ***Note: *** keep in mind, if you need to manipulate behavior (byte code) of any class, as we are doing in case of suppress constructor, we need to add `ExampleClass.class` in `@PrepareForTest`.

### 5. PowerMockito.stub()
`suppress()` works best for `void` methods or constructors, but what if need to return something. For that we have `stub()`.

```Java
// Example Source Class
public class ExampleClass extends CustomLibClass {
	public int someMethod() {
		final int valueFromSuper = super.someMethodThatOnlyWorksOnRumtime();
		return valueFromSuper * 2;
	}
}

``` 
Here, we have a method with that return something, and to test our method we need get a fixed value from `CustomLibClass.someMethodThatOnlyWorksOnRumtime()`. To achive that, we can use `stub()`.

```Java
@RunWith(PowerMockRunner.class)
@PrepareForTest({
    ExampleClass.class
})
public class ExampleClassTest {

    private ExampleClass classUnderTest;

    @Before
    public void setUp() throws Exception {
        // Create constructor of ExampleClass
        classUnderTest = new ExampleClass();
    }

	@Test
	public void someMethod_shouldReturnFour_whenCustomLibSomeMethodThatOnlyWorksOnRumtimeReturnTwo() throws NoSuchMethodException {
		// Given
		PowerMockito.stub(CustomLibClass.class.getDeclaredMethod("someMethodThatOnlyWorksOnRumtime")).toReturn(2);

		// When
		final int actualResult = classUnderTest.someMethod();

		// Then
		// As we stubbed someMethodThatOnlyWorksOnRumtime() to return 2, our result will be 2 * 2 = 4
		Assert.assertEquals(4, actualResult);
	}
}
```
> ***Note: *** keep in mind, if you need to manipulate behavior (byte code) of any class, as we are doing in case of stub method, we need to add `ExampleClass.class` in `@PrepareForTest`.

### 5. PowerMockito.replace()
`stub()` works best when we don't care about the parameters and want same result irrespective of parameter passed.

With `replace()` we can go one step ahead of `stub()`, depending upon the supplied parameters, we can return different results. Let's see another similar example:

```Java
// Example Source Class
public class ExampleClass extends Screen {
	public int getHeaderView() {
		// Lets say we have a generic method which provides different result base on the provided id
		final TextView textView = findViewById(ViewFactory.text_view);
		textView.setText("Header");
		final ImageView imageView  = findViewById(ViewFactory.image_view);
		imageView.setImage(BitmapFactory.getUserProfileImage());
		return new HStack(textView, imageView);
	}
}
```

Now, if we want to test our method with dynamic values, maybe you need to return different values based on parameter.

```Java
@RunWith(PowerMockRunner.class)
@PrepareForTest({
    ExampleClass.class
})
public class ExampleClassTest {

    private ExampleClass classUnderTest;

    @Before
    public void setUp() throws Exception {
        // Create constructor of ExampleClass
        classUnderTest = new ExampleClass();
    }

	@Test
	public void getHeaderView_shoudReturnHStack() throws NoSuchMethodException {
		// Given
		PowerMockito.replace(View.class.getDeclaredMethod("someMethodThatWillReturnDifferentValue", int.class)).with(new InvocationHandler() {
			@Override
			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
				final int viewId = (int) args[0];
		
				if (viewId == ViewFactory.text_view) {
					// return mock of TextView
					return Mockito.mock(TextView.class);
				} else if (viewId == ViewFactory.image_view) {
					// return mock of ImageView
					return Mockito.mock(ImageView.class);
				}
				// else you can call real method as well
				return method.invoke(proxy, args);
			}
		});

		// When
		final HStack actualResult = classUnderTest.getHeaderView();

		Assert.assertNotNull(actualResult);
	}
}
```
> ***Note: *** keep in mind, if you need to manipulate behavior (byte code) of any class, as we are doing in case of replace method, we need to add `ExampleClass.class` in `@PrepareForTest`.

### 6. PowerMockito.whenNew()
There are some cases where source code directly create a new instance of object without any injection mechanism. JVM will create an instance with actual class with actual methods. For these cases, we can use `PowerMockito.whenNew()` method to 

Similar to Mockito's `when()` statement, we can tell PowerMock to inject our mock instead of creating new instance of real object.

Here is an example:
```Java
// Example Source Class
public class ExampleClass {
	public int getHeight() {
		// Lets say we have a generic method which provides different result base on the provided id
		final CustomClass customClassInstance = new CustomClass();
		return customClass.getHeight();
	}
}
```
In above example, CustomClass instance is created without any mechanism to inject it, lets see how can we tell PowerMockito to inject our mock.
```Java
@RunWith(PowerMockRunner.class)
@PrepareForTest({
    ExampleClass.class
})
public class ExampleClassTest {

    private ExampleClass classUnderTest;

    @Before
    public void setUp() throws Exception {
        // Create constructor of ExampleClass
        classUnderTest = new ExampleClass();
    }

	@Test
	public void getHeight_shoudReturnHeightOfView() throws NoSuchMethodException {
		final int expectedHeight = 100;
		// Given
		final CustomClass mckCustomClass = Mockito.mock(CustomClass.class);
		Mockito.when(mckCustomClass.getHeight()).theReturn(expectedHeight);
		// Make sure you match exact parameters while mocking (same as we do for Mockito.mock statements).
		// There is also withArguments(first, second, ....)
		// and withAnyArguments() for cases when you don't care for arguments passed.
		PowerMockito.whenNew(CustomClass.class).withNoArguments().thenReturn(mckCustomClass);

		final int actualResult = classUnderTest.getHeight();

		Assert.assertEquals(expectedHeight, actualResult);
	}
}
```
> ***Note: *** keep in mind, if you need to manipulate behavior (byte code) of any class, as we are doing in case of whenNew() method, we need to add `ExampleClass.class` in `@PrepareForTest`.
>***Note*** Keep in mind, whenNew() will not work when new instance was created inside a nested scope.

### 7. Whitebox.invokeMethod()
Used to call an private method, just a wrapper on Java Reflection API's

```Java
// Example Source Class
public class ExampleClass {
	private int getHeight() {
		// Lets say we have a generic method which provides different result base on the provided id
		final CustomClass customClassInstance = new CustomClass();
		return customClass.getHeight();
	}
}
```
We can use `Whitebox.invokeMethod()` to execute that method, here an example:
```Java
@RunWith(PowerMockRunner.class)
@PrepareForTest({
    ExampleClass.class
})
public class ExampleClassTest {

    private ExampleClass classUnderTest;

    @Before
    public void setUp() throws Exception {
        // Create constructor of ExampleClass
        classUnderTest = new ExampleClass();
    }

	@Test
	public void getHeight_shoudReturnHeightOfView() throws NoSuchMethodException {
		final int expectedHeight = 100;
		// Given
		final CustomClass mckCustomClass = Mockito.mock(CustomClass.class);
		Mockito.when(mckCustomClass.getHeight()).theReturn(expectedHeight);
		// Make sure you match exact parameters while mocking (same as we do for Mockito.mock statements).
		// There is also withArguments(first, second, ....)
		// and withAnyArguments() for cases when you dont care for arguments passed.
		PowerMockito.whenNew(CustomClass.class).withNoArguments().thenReturn(mckCustomClass);

		final int actualResult = Whitebox.invokeMethod(classUnderTest, "getHeight");;

		Assert.assertEquals(expectedHeight, actualResult);
	}
}
```
> ***Note: *** you should not call `invokeMethod()` unnecessarily, try to follow code execution flow to get to any private method.

### 8. Whitebox.setInternalState()
PowerMockito allows you to inject value to globally declared member fields, this is also a wrapper around Java Reflection API's.
```Java
Whitebox.setInternalState(classUnderTest, "mCustomClassObject", (CustomClass) null);
```
> ***Note: *** You should not set values left and wright with `setInternalState()` method, this should only be used when there is some if condition that is always `true` and you need to test else condition. for e.x. a null check when that object cannot be null at that time.

### 8. Whitebox.getInternalState()
Similar to `setInternalState()` you can get value of any global field at my time with `getInternalState()` method.

This method is really useful to test any void methods, we can get any state change caused by that void method and write our assertion on that.

for example:
```Java
// Example Source Class
public class ExampleClass {

	private int mHeight = 0;

	public void clear() {
		// this is a void method that updated height and does not return anything.
		mHeight = 0;
	}
}
```
With method's like `clear()` that return void but change state of class, we can get value of `mHeight` and check if it was unset or not.

Here is how we would achieve that:
```Java
@RunWith(PowerMockRunner.class)
@PrepareForTest({
    ExampleClass.class
})
public class ExampleClassTest {

    private ExampleClass classUnderTest;

    @Before
    public void setUp() throws Exception {
        // Create constructor of ExampleClass
        classUnderTest = new ExampleClass();
    }

	@Test
	public void getHeight_shoudReturnHeightOfView() throws NoSuchMethodException {
		// Given

		// When
		classUnderTest.clear()

		// Then
		final int mHeight = Whitebox.getInternalState(classUnderTest, "mHeight");;
		Assert.assertEquals(0, mHeight);
	}
}
```