# AccessibilityService

## 一、自定义AccessibilityService实现
1. 创建一个继承自AccessibilityService的类。
```java
import android.accessibilityservice.AccessibilityService;
import android.view.accessibility.AccessibilityEvent;

public class MyAccessibilityService extends AccessibilityService {

    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
        // 在这里处理接收到的辅助功能事件
    }

    @Override
    public void onInterrupt() {
        // 在这里处理服务被中断的情况
    }

    @Override
    protected void onServiceConnected() {
        super.onServiceConnected();
        // 在这里处理服务连接成功后的逻辑
    }
}
```
<br/>

2. 在AndroidManifest.xml中注册服务
```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myaccessibilityservice">

    <application
        ...
        >
        ...
        <service
            android:name=".MyAccessibilityService"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>
            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessibility_service_config" />
        </service>
    </application>

</manifest>
```
<br/>

3. 创建辅助功能配置文件
```xml
<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeAllMask"
    android:accessibilityFeedbackType="feedbackAllMask"
    android:accessibilityFlags="flagDefault"
    android:canRetrieveWindowContent="true"
    android:description="@string/accessibility_service_description"
    android:notificationTimeout="100"
    android:packageNames="com.example.myaccessibilityservice"
    android:settingsActivity="com.example.myaccessibilityservice.SettingsActivity" />
```
<br/>

配置文件字段解析：
```
android:accessibilityEventTypes: 指定辅助功能服务需要监听的事件类型。你可以使用|来分隔多个事件类型。例如，typeViewClicked|typeViewFocused表示服务将监听点击事件和焦点事件。typeAllMask表示监听所有类型的事件。

android:accessibilityFeedbackType: 指定辅助功能服务提供的反馈类型。这可以是声音、振动、语音等。例如，feedbackSpoken表示服务将使用语音来提供反馈。feedbackAllMask表示提供所有类型的反馈。

android:accessibilityFlags: 指定辅助功能服务的一些行为标志。例如，flagDefault表示使用默认行为，flagIncludeNotImportantViews表示包括那些被标记为不重要的视图。你可以使用|来分隔多个标志。

android:canRetrieveWindowContent: 指定辅助功能服务是否可以访问活动窗口的内容。将此属性设置为true表示服务可以访问和操作窗口中的视图。这对于实现一些高级功能（如自动填充表单、点击按钮等）是必需的。

android:description: 指向一个字符串资源，该资源包含辅助功能服务的简短描述。当用户在设备的辅助功能设置中查看服务列表时，这个描述将显示在服务的名称下方。

android:notificationTimeout: 指定辅助功能事件的通知超时（以毫秒为单位）。如果在此时间段内没有新的事件发生，系统可能会中断服务以节省资源。通常，较短的超时值会导致更高的事件处理延迟，而较长的超时值会导致更低的事件处理延迟。

android:packageNames: 指定辅助功能服务需要监听的应用程序包名。你可以使用,来分隔多个包名。例如，com.example.app1,com.example.app2表示服务将只监听这两个应用程序的事件。如果省略此属性或将其设置为空字符串，服务将监听所有应用程序的事件。

android:settingsActivity: 指定辅助功能服务的设置界面的活动类名。当用户在设备的辅助功能设置中选择服务并点击“设置”按钮时，系统将启动此活动。如果你的服务没有设置界面，可以省略此属性。
```
<br/>

4. 引导用于到设置界面开启辅助功能
```java
Intent intent = new Intent(Settings.ACTION_ACCESSIBILITY_SETTINGS);
startActivity(intent);
```
<br>

## 二、AccessibilityService crash捕获
开发过程中发现，AccessibilityService crash并不会打印堆栈信息。通过设置Thread.setDefaultUncaughtExceptionHandler来获取堆栈信息。
```java
protected void onServiceConnected() {
    Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
        @Override
        public void uncaughtException(@NonNull Thread thread, @NonNull Throwable ex) {
            Log.e(TAG, "------------Crash(Base)--------------");
            StackTraceElement[] stackTrace = ex.getStackTrace();
            StringBuilder sb = new StringBuilder();
            sb.append("Uncaught exception: ").append(ex).append("\n");
            for (StackTraceElement element : stackTrace) {
                sb.append(element.toString()).append("\n");
            }
            Log.e(TAG, sb.toString());
            Log.e(TAG, "----------------------------------------");
        }
    });
}
```