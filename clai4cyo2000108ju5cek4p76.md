# Android Studio/IntelliJ: How to break on NullPointer or any other Excetion

Sometimes when you are running tests in Android Studio or Intellij Idea, you might face a failed case with following error statement:

> test should never throw an exception to this level

Cause of these errors are usually `NullPointer` or some other Exception which is not handled properly and statement is not clear where this Exception might be.

Now we can use Debugger to find exact line which is causing this failure.

Here are the steps on how:
1. Press `ctrl+shift+F8` or search for 'View Breakpoints'

You will see a dialog like this:
![image.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1668510612650/O34iADbDd.png align="left")

2. Check 'Any Exception' checkbox. It will now tell debugger to pause on any line that throws any uncaught exception.

3. Debug you test case and you will get the exact line that is causing this failure.