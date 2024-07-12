# 第一个view的measureSpec是如何生成的

### activity
```java
void makeVisible() {
    if (!mWindowAdded) {
        ViewManager wm = getWindowManager();
        // 此处的 vm 为 WindowManagerImpl
        wm.addView(mDecor, getWindow().getAttributes());
        mWindowAdded = true;
    }
    mDecor.setVisibility(View.VISIBLE);
}
```

### WindowManagerImpl
```java
@Override
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyTokens(params);
    // 此处的 mGlobal 为 WindowManagerGlobal
    mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
            mContext.getUserId());
}
```

### WindowManagerGlobal
```java
public void addView(View view, ViewGroup.LayoutParams params, Display display, Window parentWindow, int userId) {
    // 省略代码
    ...
    // 此处的 root 为 ViewRootImpl
    root.setView(view, wparams, panelParentView, userId);
}
```

### ViewRootImpl
```java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView,
        int userId) {
    // 省略代码
    ...
    mWindowAttributes.copyFrom(attrs);
}

// 该函数在view执行渲染时调用
private void performTraversals() {
    // 省略代码
    ...
    WindowManager.LayoutParams lp = mWindowAttributes;
    if (!mStopped || mReportNextDraw) {
        if (mWidth != host.getMeasuredWidth() || mHeight != host.getMeasuredHeight()
                || dispatchApplyInsets || updatedConfiguration) {
            // 这里获取 root view 的 MeasureSpec
            int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width,
                    lp.privateFlags);
            int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height,
                    lp.privateFlags);

            // 执行测量
            performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
        }
    }
}

// windowSize 是窗口的大小
// measurement 是来源于 activity 的 getWindow().getAttributes()，其默认值为 new WindowManager.LayoutParams(),
private static int getRootMeasureSpec(int windowSize, int measurement, int privateFlags) {
    int measureSpec;
    final int rootDimension = (privateFlags & PRIVATE_FLAG_LAYOUT_SIZE_EXTENDED_BY_CUTOUT) != 0
            ? MATCH_PARENT : measurement;
    switch (rootDimension) {
        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
    }
    return measureSpec;
}
```

## LayoutParams
```java
// 默认构造函数
public LayoutParams() {
    super(LayoutParams.MATCH_PARENT, LayoutParams.MATCH_PARENT);
    type = TYPE_APPLICATION;
    format = PixelFormat.OPAQUE;
}
```

## 结论
视图层级的 Root View 的 MeasureSpec 是 MeasureSpec.EXACTLY + windowSize 的组合。