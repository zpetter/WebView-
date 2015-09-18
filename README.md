# WebView-
概览：
          Android WebView在Android平台上是一个特殊的View， 他能用来显示网页，这个类可以被用来在你的app中仅仅显示一张在线的网页，还可以用来开发浏览器。WebView内部实现是采用渲染引擎来展示view的内容，提供网页前进后退，网页放大，缩小，搜索，前端开发者可以使用web inspector(Android 4.4系统支持，4.4一下可以采用http://developer.android.com/guide/webapps/debugging.html)调试HTML,CSS,Javascript等等功能。在Android 4.3系统及其一下WebView内部采用Webkit渲染引擎，在Android 4.4采用chromium 渲染引擎来渲染View的内容。

1.WebView的基本使用
 （1）创建WebView的实例加入到Activity view tree中   

[java] view plaincopy在CODE上查看代码片派生到我的代码片
WebView webview = new WebView(this);  
setContentView(webview);  
  （2）在xml中配置WebView

[html] view plaincopy在CODE上查看代码片派生到我的代码片
<Webview  
    android:layout_width="match_parent"  
    android:layout_height="match_parent" >  
</Webview>  
  （3）访问网页
[html] view plaincopy在CODE上查看代码片派生到我的代码片
webview.loadUrl("http://developer.android.com/");  

2.WebView API使用详解
   1）请求加载网页部分
        
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void loadData (String data, String mimeType, String encoding)  
加载指定的data数据

参数说明：

data  字符串String形式的数据 可以通过base64编码而来

mineType data数据的 MIME类型， e.g. 'text/html'

encoding data数据的编码格式

Tips：

1.Javascript有同源限制，同源策略限制了一个源中加载文本或者脚本与来自其他源中的数据交互方式。避免这种限制可以使用loadDataWithBaseURL()方法。

2.encoding参数制定data参数是否为base64或者 URL 编码，如果data是base64编码那么 encoding必须填写 "base64“。

http://developer.android.com/reference/android/webkit/WebView.html

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void loadDataWithBaseURL (String baseUrl, String data, String mimeType, String encoding, String historyUrl)  
使用baseUrl加载base URL的网页内容，baseUrl解决相关url使用Javascript相同源问题。
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void loadUrl (String url)  
加载制定url的网页内容
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void loadUrl (String url, Map<String, String> additionalHttpHeaders)  

加载制定url并携带http header数据。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void reload ()  
重新加载页面

Tip（重要）

页面所有资源会重新加载


[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void stopLoading ()  


2) 前进后退

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void goBack ()  
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void goForward ()  
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void goBackOrForward (int steps)  
以当前的index为起始点前进或者后退到历史记录中指定的steps，  如果steps为负数则为后退，正数则为前进
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean canGoForward ()  
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean canGoBack ()  


3)JavaScript操作

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void addJavascriptInterface (Object object, String name)  

当网页需要和App进行交互时，可以注入Java对象提供给JavaScritp调用.  Java对象提供相应的方法供js使用.
Tips（重要）

问题：在Android 4.2以下使用这个api会涉及到JavaScript安全问题， javascript可以通过反射这个java对象的相关类进行攻击。

解决：可以采用白名单的机制调用这个方法.

在Android4.2极其以上系统需要给提供js调用的方法前加入一个注视：@JavaScriptInterface; 在虚拟机当中 Javascript调用Java方法会检测这个anotation，如果方法被标识@JavaScriptInterface则Javascript可以成功调用这个Java方法，否则调用不成功。

example：

[java] view plaincopy在CODE上查看代码片派生到我的代码片
class JsObject {  
   @JavascriptInterface  
   public String toString() { return "injectedObject"; }  
}  
webView.addJavascriptInterface(new JsObject(), "injectedObject");  

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void evaluateJavascript (String script, ValueCallback<String> resultCallback)  
这个方法在Android 4.4系统引入，因此只能在Android4.4系统中才能使用，提供在当前页面显示上下文中异步执行javascript代码
Tips（重要）

这个方法必须在UI线程调用，这个函数的回调也会在UI线程执行。

那么在Android4.4一下如何执行javascrit代码呢

可以通过 WebView提供的loadUrl方法：具体格式如下：

[java] view plaincopy在CODE上查看代码片派生到我的代码片
webView.loadUrl("javascript:alert(injectedObject.toString())");  
其中javascript: 是执行javascript代码的标识 ， 后面是javascript语句。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void removeJavascriptInterface (String name)  
删除addJavascripInterface时对webview注入的java对象. 此方法在不同的Android系统WebView会有问题，会存在失效情况。

4）网页查找功能

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public int findAll (String find)  
这个API在Android 4.1 就已经被去除， 在Android 4.1极其以上系统使用findAllAsync方法
这个API还存在bug 具体请见我的之前一篇博文Android WebView findAll bug

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void findAllAsync (String find)  
异步执行查找网页内包含的字符并设置高亮，查找结果会回调.
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void findNext (boolean forward)  
查找下一个匹配的字符
使用example：

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public class TestFindListener implements android.webkit.WebView.FindListener {  
    private FindListener mFindListener;  
  
    public TestFindListener(FindListener findListener) {  
        mFindListener = findListener;  
    }  
  
    @Override  
    public void onFindResultReceived(int activeMatchOrdinal, int numberOfMatches,  
            boolean isDoneCounting) {  
        mFindListener.onFindResultReceived(activeMatchOrdinal, numberOfMatches, isDoneCounting);  
    }  
}  
  
   public void findAllAsync(String searchString) {  
       if (android.os.Build.VERSION_CODES.JELLY_BEAN <= Build.VERSION.SDK_INT)  
           mWebView.findAllAsync(searchString);  
       else {  
           int number = mWebView.findAll(searchString);  
           if (mIKFindListener !=null)  
               mIKFindListener.onFindResultReceived(number);  
           fixedFindAllHighLight();  // 参见我之前一篇博文Android WebView API findAll bug  
       }  
   }  
     
   mWebView.findNext(forward);  

5）数据清除部分

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void clearCache (boolean includeDiskFiles)  
清除网页访问留下的缓存，由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序.
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void clearFormData ()  
这个api仅仅清除自动完成填充的表单数据，并不会清除WebView存储到本地的数据。
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void clearHistory ()  
清除当前webview访问的历史记录，只会webview访问历史记录里的所有记录除了当前访问记录.
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void clearMatches ()  
清除网页查找的高亮匹配字符
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void clearView ()  
在Android 4.3及其以上系统这个api被丢弃了， 并且这个api大多数情况下会有bug，经常不能清除掉之前的渲染数据。官方建议通过loadUrl("about:blank")来实现这个功能，阴雨需要重新加载一个页面自然时间会收到影响。
6）WebView的状态

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onResume ()  
激活WebView为活跃状态，能正常执行网页的响应
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onPause ()  
当页面被失去焦点被切换到后台不可见状态，需要执行onPause动过， onPause动作通知内核暂停所有的动作，比如DOM的解析、plugin的执行、JavaScript执行。并且可以减少不必要的CPU和网络开销，可以达到省电、省流量、省资源的效果。
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void pauseTimers ()  

当应用程序被切换到后台我们使用了webview， 这个方法不仅仅针对当前的webview而是全局的全应用程序的webview，它会暂停所有webview的layout，parsing，javascripttimer。降低CPU功耗。
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void resumeTimers ()  
恢复pauseTimers时的动作。
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void destroy ()  
Tips（重要）
这个方法必须在webview从view tree中删除之后才能被执行， 这个方法会通知native释放webview占用的所有资源。

7) WebView 事件回调监听

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void setWebChromeClient (WebChromeClient client)  
主要通知客户端app加载当前网页的 title，Favicon，progress,javascript dialog等事件，通知客户端处理这些相应的事件。
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void setWebViewClient (WebViewClient client)  
主要通知客户端app加载当前网页时的各种时机状态，onPageStart,onPageFinish,onReceiveError等事件。
8) Android 5.0 Lollipop 新API

public static void enableSlowWholeDocumentDraw ()

Android 5.0 Webview默认提供减少内存占用支持，并且智能选择需要绘制的HTML document部门来提供性能。 当然开发者可以在自己应用程序需要时关闭这个选项(enableSlowWholeDocumentDraw)。

3. WebView Demo
[java] view plaincopy在CODE上查看代码片派生到我的代码片
package com.example.webviewdemo;  
  
import android.annotation.SuppressLint;  
import android.app.Activity;  
import android.content.Context;  
import android.graphics.Bitmap;  
import android.os.Message;  
import android.webkit.WebChromeClient;  
import android.webkit.WebSettings;  
import android.webkit.WebView;  
import android.webkit.WebViewClient;  
  
public class WebViewBase extends WebView {  
    private static final String DEFAULT_URL = "http://www.ijinshan.com/";  
    private Activity mActivity;  
    public WebViewBase(Context context) {  
        super(context);  
        mActivity = (Activity) context;  
        init(context);  
    }  
      
    @SuppressLint("SetJavaScriptEnabled")  
    private void init(Context context) {  
        WebSettings webSettings = this.getSettings();  
        webSettings.setJavaScriptEnabled(true);  
        webSettings.setSupportZoom(true);  
        //webSettings.setUseWideViewPort(true);  
        this.setWebViewClient(mWebViewClientBase);  
        this.setWebChromeClient(mWebChromeClientBase);  
        this.loadUrl(DEFAULT_URL);  
        this.onResume();  
    }  
      
    private WebViewClientBase mWebViewClientBase = new WebViewClientBase();  
      
    private class WebViewClientBase extends WebViewClient {  
  
        @Override  
        public boolean shouldOverrideUrlLoading(WebView view, String url) {  
            // TODO Auto-generated method stub  
            return super.shouldOverrideUrlLoading(view, url);  
        }  
  
        @Override  
        public void onPageStarted(WebView view, String url, Bitmap favicon) {  
            // TODO Auto-generated method stub  
            super.onPageStarted(view, url, favicon);  
        }  
  
        @Override  
        public void onPageFinished(WebView view, String url) {  
            // TODO Auto-generated method stub  
            super.onPageFinished(view, url);  
        }  
  
        @Override  
        public void onReceivedError(WebView view, int errorCode,  
                String description, String failingUrl) {  
            // TODO Auto-generated method stub  
            super.onReceivedError(view, errorCode, description, failingUrl);  
        }  
  
        @Override  
        public void doUpdateVisitedHistory(WebView view, String url,  
                boolean isReload) {  
            // TODO Auto-generated method stub  
            super.doUpdateVisitedHistory(view, url, isReload);  
        }  
    }  
      
    private WebChromeClientBase mWebChromeClientBase = new WebChromeClientBase();  
      
    private class WebChromeClientBase extends WebChromeClient {  
  
        @Override  
        public void onProgressChanged(WebView view, int newProgress) {  
            mActivity.setProgress(newProgress * 1000);  
        }  
  
        @Override  
        public void onReceivedTitle(WebView view, String title) {  
            // TODO Auto-generated method stub  
            super.onReceivedTitle(view, title);  
        }  
  
        @Override  
        public void onReceivedTouchIconUrl(WebView view, String url,  
                boolean precomposed) {  
            // TODO Auto-generated method stub  
            super.onReceivedTouchIconUrl(view, url, precomposed);  
        }  
  
        @Override  
        public boolean onCreateWindow(WebView view, boolean isDialog,  
                boolean isUserGesture, Message resultMsg) {  
            // TODO Auto-generated method stub  
            return super.onCreateWindow(view, isDialog, isUserGesture, resultMsg);  
        }  
          
    }  
}  

[html] view plaincopy在CODE上查看代码片派生到我的代码片
<uses-permission android:name="android.permission.INTERNET" />  
