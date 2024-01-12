在Android WebView中获取类似PC浏览器控制台（console）中的信息，可以通过几种方法来实现。以下是一些常用的方法：

1. 使用`WebChromeClient`的`onConsoleMessage`方法：
   你可以为你的WebView设置一个`WebChromeClient`，并重写`onConsoleMessage`方法来捕获JavaScript控制台的消息。

   ```java
   webView.setWebChromeClient(new WebChromeClient() {
       @Override
       public boolean onConsoleMessage(ConsoleMessage consoleMessage) {
           Log.d("WebView", consoleMessage.message() + " -- From line "
                   + consoleMessage.lineNumber() + " of "
                   + consoleMessage.sourceId());
           return super.onConsoleMessage(consoleMessage);
       }
   });
   ```

2. 使用`addJavascriptInterface`方法：
   你可以向WebView添加一个JavaScript接口，然后在JavaScript代码中调用该接口来发送消息。

   ```java
   public class WebAppInterface {
       Context mContext;

       WebAppInterface(Context c) {
           mContext = c;
       }

       @JavascriptInterface
       public void showConsoleMessage(String message) {
           Log.d("WebView", message);
       }
   }

   webView.addJavascriptInterface(new WebAppInterface(this), "Android");

   // 然后在你的JavaScript代码中，你可以这样调用：
   console.log = function(message) {
       Android.showConsoleMessage(message);
   };
   ```

3. 启用远程调试：
   从Android 4.4 (KitKat)开始，你可以启用WebView的远程调试功能，这样你就可以使用Chrome DevTools来查看你的WebView中的调试信息，包括控制台消息。

   在你的应用程序中启用远程调试的方法如下：
   ```java
   if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
       WebView.setWebContentsDebuggingEnabled(true);
   }
   ```

   启用后，你可以在你的电脑上的Chrome浏览器中输入 `chrome://inspect`，然后选择你的设备和应用程序中的WebView来进行调试。

4. 使用VConsole()
   参考：https://gitee.com/Tencent/vConsole/tree/master/
   
请注意，出于安全考虑，不建议在生产环境中启用远程调试或者将`addJavascriptInterface`暴露给不受信任的内容。确保在发布应用程序之前关闭远程调试，并且仅将JavaScript接口暴露给可信的内容。