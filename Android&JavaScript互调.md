## Android & JavaScript互调
```
// javaScript 需要调用的方法
private InJavaScript {
     Context context;

     InJavaScript () {}

     InJavaScript (Context context) {
     this.context  = context
     }
     
     @JavaScriptInterface // 添加注解，很重要的一步
     public void runOnAndroidJavaScript (final String str) {
          runOnUiThread (new Runnable () {
               @override
               public void run () {

                    yourOwnFunction(str);
               }
          }
     }
}

// 相关配置
webView.getSettings.setJavaScriptEnabled (true);
// javaScript 调用的接口，随便用了一个”Android“的标志 
webView.addJavaScriptInterface(new InJavaScript(this), "Android");
```
```
/*
     H5 调用方式
     window.Android.runOnAndroidJavaScript("param");
*/
```
```
// Android 调起javaScript方法
webView.load("javascript:functionName("param")");
```
