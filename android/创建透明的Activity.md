# 创建透明Activity
在开发中，我们有时需要一个透明的Activity作为DeepLink事件分发的中转，参考了一些文章，并进行了实践，有两种方法可以满足需求。

## 1. 使用Android自带的Theme.Translucent
```
@android:style/Theme.Translucent
@android:style/Theme.Translucent.NoTitleBar
@android:style/Theme.Translucent.NoTitleBar.Fullscreen
```
在AndroidManifest中直接使用即可：
```
<activity
    android:name="TranslucentActivity"
    android:theme="@android:style/Theme.Translucent.NoTitleBar" />
```

这个方法使用简单，如果只是开发简单需求，是用这种方式更为快捷。
但也有它的不足：

TranslucentActivity只能继承原生的Activity，才能使用Theme.Translucent，而不能继承AppCompatActivity，但我们现在开发使用androix包会继承AppCompatActivity
不能灵活定制style，如statusBar不能设置成透明

## 2. 使用自定义Theme
```
<style name="TranslucentStyle" parent="Theme.AppCompat.Light.NoActionBar">
    <item name="android:windowBackground">@android:color/transparent</item> <!-- 背景色透明 -->
    <item name="android:windowIsTranslucent">true</item> <!-- 是否有透明属性 -->
    <item name="android:backgroundDimEnabled">false</item> <!-- 背景是否半透明 -->
    <item name="android:statusBarColor">@android:color/transparent</item> <!-- 状态栏透明 -->
    <item name="android:windowAnimationStyle">@android:style/Animation.Translucent</item> <!-- activity窗口切换效果 -->
</style>
```

在AndroidManifest中使用：
```
<activity
    android:name="TranslucentActivity"
    android:theme="@style/TranslucentStyle" />
```
这样自定义Theme，TranslucentActivity可以继承AppCompatActivity，且能更灵活的配置Activity style属性。