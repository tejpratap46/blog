# Mockito: ArgumentMatcher vs ArgumentCaptor

`Mockito.when()` allows us to mock behavior of methods in that class. But in order to achieve this we need to mock supplied parameters exactly as our source class expected, If any of our parameter is mismatched, our method will not be mocked and it will return `null` as a result.

We can avoid this by using **ArgumentMatcher** and **ArgumentCaptor**.

Here is a simple example:

```Java
// This is a simple method that has to be mocked
final int addition = calculator.getAddition(2, 3);
```
```Java
// For cases, where you do not know exact parameter then you can use Mockito.anyInt()
Mockito.when(mckCalculator.getAddition(Mockito.anyInt(), Mockito.anyInt())).thenReturn(5);
```
These `any` matchers comes too handy when parameters are either computed by program or you simply want to assert if correct parameter are passed to the method.

Similar to `any()`, we also have type specific matchers, but keep in mind these matcher will not work if your argument is null.

Here is a list of all other matchers available to you:

Mockito.anyInt(), Mockito.anyLong(), Mockito.anyString(), Mockito.anyBoolean(), Mockito.anyByte(), Mockito.anyChar(), Mockito.anyFloat(), Mockito.anyDouble(), Mockito.anyShort(), Mockito.anyList(), Mockito.anySet() and Mockito.anyMap()

Lets see an example:
We have a `TaskManager.class` which will be mocked eventually.
```Java
public class TaskManager {
    private Map<String, Task> taskMap = new ConcurrentHashMap<>();
    public Task createTask(String name) {
        final task = new Task();
        taskMap.put(name, task);
        return task;
    }

    public void deleteTask(String name) {
        taskMap.remove(name);
    }

    public void executeTask(String name, Runnable runnable) {
        // Some long running task
        runnable.run();
    }
}
```
We have another class which uses `TaskManager.class` internally, lets call it `ExampleWorker`. We have to write tests for `ExampleWorker`.
```Java
public class ExampleWorker {
    private final String TASK_NAME = "exampleWorkedTask";
    
    private final TaskManager mTaskManager;
    private final Task mTask;

    public ExampleTask(TaskManager taskManager) {
        mTaskManager = taskManager;
        mTask = mTaskManager.createTask(TASK_NAME);
    }

    public Task getTask() {
        return mTask;
    }

    public void deleteTask() {
        mTaskManager.deleteTask(TASK_NAME);
        mTask = null;
    }

    public void runTask() {
        mTaskManager.executeTask(TASK_NAME, new Runnable() {
            @Override
            public void run() {
                // Do something here
            }
        });
    }
}
```
We can write test for `ExampleWorker` as below:
```Java
public class ExampleWorkerTest {
    private ExampleWorker classUnderTest;

    private TaskManager mckTaskManager;

    @Before
    public void setUp() throws Exception {
        mckTaskManager = Mockito.mock(TaskManager.class);

        classUnderTest = new ExampleWorker(mckTaskManager);
    }

    @After
    public void tearDown() throws Exception {
        classUnderTest = null;
    }

    @Test
    public void constructor_shouldCreateTask() {
        // Then
        final Task actualResult = classUnderTest.getTask();
        Assert.assertNotNull(actualResult);
    }
}
```
Now comes the fun part where we try to test `deleteTask()` method which will internally call `TaskManager`'s `deleteTask(taskName)` method.

We can write simple tests with **ArgumentMatcher**.
```Java
@Test
public void deleteTask_shouldRemoveTask() {
    // Given
    // There are multiple ways to mock same statement, use eany one of these three based on your requirement
    // You can match exact parameter
    Mockito.doNothing().when(mckTaskManager).deleteTask("exampleWorkedTask");
    // You can match anyString(), but it will not work when string is null
    Mockito.doNothing().when(mckTaskManager).deleteTask(Mockito.anyString());
    // You can match any(), it will work even if parameter String is null
    Mockito.doNothing().when(mckTaskManager).deleteTask(Mockito.any());

    // When
    classUnderTest.deleteTask();

    // Then
    final Task actualResult = classUnderTest.getTask();
    Assert.assertNotNull(actualResult);
}
```
But what if we want to assert on the parameters passed while calling `deleteTask(taskName)` method, we can use **ArgumentCaptor**
```Java
@Test
public void deleteTask_shouldRemoveTask_assertWithArgumenCaptor() {
    // Given
    final ArgumentCaptor<String> taskNameArgumentMatcher = ArgumentCaptor.forClass(String.class);
    // You can create a captor and validate
    Mockito.doNothing().when(mckTaskManager).deleteTask(taskNameArgumentMatcher.captor());

    // When
    classUnderTest.deleteTask();

    // Then
    // Now we can get the value from taskNameArgumentMatcher and assert it with expectedValue
    final String actualTaskName = taskNameArgumentMatcher.getValue();
    Assert.assertEquals("exampleWorkedTask", actualTaskName);
}
```
But guess what, you can also get parameters used with ***ArgumentMatcher*** same as ***ArgumentCaptor***. Here is an example:
```Java
@Test
public void deleteTask_shouldRemoveTask_assertWithArgumenMatcher() {
    // Given
    final AtomicReference<String> taskNameReference = new AtomicReference<>();
    Mockito.doAnswer(invocation -> {
        taskNameReference.set(0, invocation.getArgument(0, String.class));
        return null;
    }).when(mckTaskManager).deleteTask(Mockito.anyString());

    // When
    classUnderTest.deleteTask();

    // Then
    // Now we can get the value from taskNameArgumentMatcher and assert it with expectedValue
    final String actualTaskName = taskNameReference.get();
    Assert.assertEquals("exampleWorkedTask", actualTaskName);
}
```