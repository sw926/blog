---
layout: android
title: Android Test
date: 2015-09-08 14:59:11
tags:
	- Android
category: Android
---
新建工程的时候会自动创建一个androidTest
我们首先在androidTest下新建一个Activity test
```java
public class MainActivityTest extends ActivityInstrumentationTestCase2<MainActivity> {

    private MainActivity mMainActivity;
    private TextView mTextView;

    public MainActivityTest() {
        super(MainActivity.class);
    }

    @Override
    protected void setUp() throws Exception {
        super.setUp();
        mMainActivity = getActivity();
        mTextView = (TextView) mMainActivity.findViewById(R.id.tv_text);
    }

    public void testPreconditions() {
        assertNotNull("mFirstTestActivity is null", mMainActivity);
        assertNotNull("mFirstTestText is null", mTextView);
    }
}
```
需要注意，在setUp中获取Activity的引用，每个test case必须以test开头，testPreconditions用了测试是否正常初始化了测试环境。
运行测试时，Activity是在UI线程上运行，test是在另一个线程上运行，test可以获取Activity和view的引用，我们可以测试一个view的状态是否正确
```java
@MediumTest
public void testButton() {
    final View decorView = mMainActivity.getWindow().getDecorView();

    ViewAsserts.assertOnScreen(decorView, mBtnTest);

    final ViewGroup.LayoutParams layoutParams = mBtnTest.getLayoutParams();
    assertNotNull(layoutParams);
    assertEquals(layoutParams.width, WindowManager.LayoutParams.WRAP_CONTENT);
    assertEquals(layoutParams.height, WindowManager.LayoutParams.WRAP_CONTENT);
}
```
这个测试用例用了测试一个按钮是否在Activity中，LayoutParams是否正确。测试函数使用了@MediumTest的注解，
这类注解表示测试用例的执行时间
@SmallTests < 100ms
@MediumTests < 2s
@LargeTests < 120s
具体教程可以查看 https://plus.google.com/+AndroidDevelopers/posts/TPy1EeSaSg8

在test中不能直接改变UI的状态，下面的用例用来测试button的点击行为
```java
@MediumTest
public void testClickButton() {
    TouchUtils.clickView(this, mBtnTest);
    assertTrue(View.GONE == mTextView.getVisibility());
}
``` 
TouchUtils提供了view的模拟触屏操作，它被设计用来安全的从test线程向UI线程发送event，是不能直接在UI线程中使用的。


