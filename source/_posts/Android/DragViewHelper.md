## DragViewHelper

为了免于有些同学的迷惑。我们再来看看 tryCaptureView() 方法的调用流程。


```
public void processTouchEvent(MotionEvent ev) {
    switch (action) {
        case MotionEvent.ACTION_DOWN: {
            final float x = ev.getX();
            final float y = ev.getY();
            final int pointerId = ev.getPointerId(0);
            final View toCapture = findTopChildUnder((int) x, (int) y);

            tryCaptureViewForDrag(toCapture, pointerId);

            break;
        }

    }
}


boolean tryCaptureViewForDrag(View toCapture, int pointerId) {
    if (toCapture != null && mCallback.tryCaptureView(toCapture, pointerId)) {
        captureChildView(toCapture, pointerId);
        return true;
    }
    return false;
}
```


精简了相关代码，我们可以看到这样的流程。

**Callback.tryCaptureView() 在 captureChildView() 之前调用，它是正常流程中判断 captureChildView() 的一个依据，只有它返回 true 时，captureChildView 才能被调用。**

到这里，边缘拖拽的效果也实现了，那么 ViewDragHelper 还有什么有趣的操作方法呢？可能大家对这两个会感兴趣。

```
//快速滚动的意思，一般手指离开后 view 还会由于惯性继续滑动。
void    flingCapturedView(int minLeft, int minTop, int maxLeft, int maxTop);

// 让 child 平滑地滑动到某个位置
boolean smoothSlideViewTo(View child, int finalLeft, int finalTop)
```

如果要移动像 Button 这样 clickable == true 的控件，要复写 ViewDragHelper.Callback 中的两个回调方法 getViewHorizontalDragRange() 和 getViewVerticalDragRange()，使它们对应方法的返回值大于 0 就好了。