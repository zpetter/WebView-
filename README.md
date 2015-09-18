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

<uses-permission android:name="android.permission.INTERNET" />  
概览：
   Android WebView 做为承载网页的载体控件，他在网页显示的过程中会产生一些事件，并回调给我们的应用程序，以便我们在网页加载过程中做应用程序想处理的事情。比如说客户端需要显示网页加载的进度、网页加载发生错误等等事件。 WebView提供两个事件回调类给应用层，分别为WebViewClient,WebChromeClient开发者可以继承这两个类，接手相应事件处理。WebViewClient 主要提供网页加载各个阶段的通知，比如网页开始加载onPageStarted，网页结束加载onPageFinished等；WebChromeClient主要提供网页加载过程中提供的数据内容，比如返回网页的title,favicon等。

1.WebViewClient的基本使用
 创建WebViewClient实例并设置到WebView对象中，具体代码参考如下：
[java] view plaincopy在CODE上查看代码片派生到我的代码片
class MyAndroidWebViewClient extends WebViewClient {  
     @Override  
    public void onPageStarted(WebView view, String url, Bitmap favicon) {  
       // TODO  
    }  
  
    @Override  
    public void onPageFinished(WebView view, String url) {  
      // TODO  
    }  
[java] view plaincopy在CODE上查看代码片派生到我的代码片
}  
webview.setWebViewClient(new MyAndroidWebViewClient ());  

2.WebViewClient API详解
1）网页加载时机部分
     
[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean shouldOverrideUrlLoading(WebView view, String url)  
当加载的网页需要重定向的时候就会回调这个函数告知我们应用程序是否需要接管控制网页加载，如果应用程序接管，并且return true意味着主程序接管网页加载，如果返回false让webview自己处理。

参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param url    即将要被加载的url
@return  true 当前应用程序要自己处理这个url， 返回false则不处理。

Tips
(1) 当请求的方式是"POST"方式时这个回调是不会通知的。
(2) 当我们访问的地址需要我们应用程序自己处理的时候，可以在这里截获，比如我们发现跳转到的是一个market的链接，那么我们可以直接跳转到应用市场，或者其他app。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onPageStarted(WebView view, String url, Bitmap favicon)  
当内核开始加载访问的url时，会通知应用程序，对每个main frame这个函数只会被调用一次，页面包含iframe 或者framesets 不会另外调用一次onPageStarted，当网页内内嵌的frame 发生改变时也不会调用onPageStarted。

参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param url    即将要被加载的url
@param favicon 如果这个favicon已经存储在本地数据库中，则会返回这个网页的favicon，否则返回为null。

Tips：
(1) iframe 可能不少人不知道什么含义，这里我解释下，iframe 我们加载的一张，下面有很多链接，我们随便点击一个链接是即当前host的一个iframe.
(2) 有个问题可能是开发者困惑的，onPageStarted和shouldOverrideUrlLoading 在网页加载过程中这两个函数到底哪个先被调用。
     当我们通过loadUrl的方式重新加载一个网址时候，这时候会先调用onPageStarted再调用shouldOverrideUrlLoading，当我们在打开的这个网址点击一个link，这时候会先调用shouldOverrideUrlLoading 再调用onPageStarted。不过shouldOverrideUrlLoading不一定每次都被调用，只有需要的时候才会被调用。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onPageFinished(WebView view, String url)  

当内核加载完当前页面时会通知我们的应用程序，这个函数只有在main frame情况下才会被调用，当调用这个函数之后，渲染的图片不会被更新，如果需要获得新图片的通知可以使用@link WebView.PictureListener#onNewPicture。

参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param url    即将要被加载的url

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onLoadResource(WebView view, String url)  
通知应用程序WebView即将加载url 制定的资源

参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param url    即将加载的url 资源

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public WebResourceResponse shouldInterceptRequest(WebView view,  
            String url)  
通知应用程序内核即将加载url制定的资源，应用程序可以返回本地的资源提供给内核，若本地处理返回数据，内核不从网络上获取数据。

参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param url    raw url 制定的资源
@return 返回WebResourceResponse包含数据对象，或者返回null

Tips
这个回调并不一定在UI线程执行，所以我们需要注意在这里操作View或者私有数据相关的动作。
如果我们需要改变网页的背景，或者需要实现网页页面颜色定制化的需求，可以在这个回调时机处理。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onReceivedError(WebView view, int errorCode,  
            String description, String failingUrl)  
当浏览器访问制定的网址发生错误时会通知我们应用程序，比如网络错误。

参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param errorCode 错误号可以在WebViewClient.ERROR_* 里面找到对应的错误名称。
@param description 描述错误的信息
@param failingUrl  当前访问失败的url，注意并不一定是我们主url

Tips
在onReceiveError我们可以自定义网页的错误页面。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onFormResubmission(WebView view, Message dontResend,  
            Message resend)  
如果浏览器需要重新发送POST请求，可以通过这个时机来处理。默认是不重新发送数据。
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param dontResent 当浏览器不需要重新发送数据时，可以使用这个参数。
@param resent 当浏览器需要重新发送数据时， 可以使用这个参数。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void doUpdateVisitedHistory(WebView view, String url,  
            boolean isReload)  

通知应用程序可以将当前的url存储在数据库中，意味着当前的访问url已经生效并被记录在内核当中。这个函数在网页加载过程中只会被调用一次。注意网页前进后退并不会回调这个函数。
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param url 当前正在访问的url 
@ param isReload 如果是true 这个是正在被reload的url


[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onReceivedSslError(WebView view, SslErrorHandler handler,  
            SslError error)  

当网页加载资源过程中发现SSL错误会调用此方法。我们应用程序必须做出响应，是取消请求handler.cancel(),还是继续请求handler.proceed();内核的默认行为是handler.cancel();

参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param handler 处理用户请求的对象。
@param error  SSL错误对象

Tips
内核会记住本次选择，如果下次还有相同的错误，内核会直接执行之前选择的结果。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onReceivedHttpAuthRequest(WebView view,  
            HttpAuthHandler handler, String host, String realm)  

通知应用程序WebView接收到了一个Http auth的请求，应用程序可以使用supplied 设置webview的响应请求。默认行为是cancel 本次请求。
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param handler 用来响应WebView请求的对象
@param host  请求认证的host
@param realm 认真请求所在的域

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean shouldOverrideKeyEvent(WebView view, KeyEvent event)   
提供应用程序同步一个处理按键事件的机会，菜单快捷键需要被过滤掉。如果返回true，webview不处理该事件，如果返回false， webview会一直处理这个事件，因此在view 链上没有一个父类可以响应到这个事件。默认行为是return false；
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param  event 键盘事件名
@return  如果返回true,应用程序处理该时间，返回false 交有webview处理。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onScaleChanged(WebView view, float oldScale, float newScale)  
通知应用程序webview 要被scale。应用程序可以处理改事件，比如调整适配屏幕。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onReceivedLoginRequest(WebView view, String realm,  
            String account, String args)  

通知应用程序有个自动登录的帐号过程
参数说明：
@param view 请求登陆的webview
@param realm 账户的域名，用来查找账户。
@param account 一个可选的账户，如果是null 需要和本地的账户进行check， 如果是一个可用的账户，则提供登录。
@param  args  验证制定参数的登录用户

3.WebChromeClient 基本使用

4. WebChromeClient API详解
创建WebChromeClient实例并设置到WebView对象中，具体代码参考如下：

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onProgressChanged(WebView view, int newProgress)  
通知应用程序当前网页加载的进度。
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onReceivedTitle(WebView view, String title)  

当document 的title变化时，会通知应用程序
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param title  document新的title
Tips
这个函数调用时机不确定，有可能很早，有可能很晚，取决于网页把title设置在什么位置，大多数网页一般把title设置到页面的前面，因此很多情况会比较早回调到这个函数。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onReceivedIcon(WebView view, Bitmap icon)  

当前页面有个新的favicon时候，会回调这个函数。
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param icon 当前页面的favicon

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onReceivedTouchIconUrl(WebView view, String url,  
            boolean precomposed)  


通知应用程序 apple-touch-icon的 url 
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param url  apple-touch-icon 的服务端地址
@param precomposed  如果precomposed 是true 则touch-icon是预先创建的
Tips 
如果应用程序需要这个icon的话， 可以通过这个url获取得到 icon。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onShowCustomView(View view, CustomViewCallback callback)  

通知应用程序webview需要显示一个custom view，主要是用在视频全屏HTML5Video support。
参数说明：
@param view 即将要显示的view
@param callback  当view 需要dismiss 则使用这个对象进行回调通知。


[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onHideCustomView()  
退出视频通知

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean onCreateWindow(WebView view, boolean isDialog,  
            boolean isUserGesture, Message resultMsg)  
请求创建一个新的窗口，如果我们应用程序接管这个请求，必须返回true，并且创建一个新的webview来承载主窗口。
如果应用程序不处理，则需要返回false，默认行为和返回false表现一样。
参数说明：
@param view 请求创建新窗口的webview
@param isUserGesture 如果是true，则说明是来自用户收拾操作行为，比如用户点击链接
@param isDialog true 请求创建的新窗口必须是个dialog，而不是全屏的窗口。
@param resultMsg 当webview创建时需要发送一个消息。WebView.WebViewTransport.setWebView(WebView)
Tips 具体例子如下：
[java] view plaincopy在CODE上查看代码片派生到我的代码片
  private void createWindow(final Message msg) {  
WebView.WebViewTransport transport = (WebView.WebViewTransport) msg.obj;  
final Tab newTab = mWebViewController.openTab(null, Tab.this, true,  
        true);  
transport.setWebView(newTab.getWebView());  
msg.sendToTarget();  
  }  

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onRequestFocus(WebView view)  
webview请求得到focus，发生这个主要是当前webview不是前台状态，是后台webview。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onCloseWindow(WebView window)  

通知应用程序从关闭传递过来的webview并从view tree中remove。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean onJsAlert(WebView view, String url, String message,  
            JsResult result)  

通知应用程序显示javascript alert对话框，如果应用程序返回true内核认为应用程序处理这个消息，返回false，内核自己处理。
参数说明：
@param view 接收WebViewClient的那个实例，前面看到webView.setWebViewClient(new MyAndroidWebViewClient())，即是这个webview。
@param url 当前请求弹出javascript 对话框webview 加载的url地址。
@param message 弹出的内容信息
@result 用来响应用户的处理。

Tips
如果我们应用接管处理， 则必须给出result的结果，result.cancel,result.comfirm必须调用其中之后，否则内核会hang住。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean onJsConfirm(WebView view, String url, String message,  
            JsResult result)  

通知应用程序提供confirm 对话框。
参数说明同上onJsAlert

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean onJsPrompt(WebView view, String url, String message,  
            String defaultValue, JsPromptResult result)  

通知应用程序显示一个prompt对话框。 
Tips
必须调用result.confirm 方法如果应用程序接管这个方法。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public boolean onJsBeforeUnload(WebView view, String url, String message,  
            JsResult result)  
通知应用程序显示一个对话框，让用户选择是否离开当前页面，这个回调是javascript中的onbeforeunload事件，如果客户端返回true，内核会认为客户端提供对话框。默认行为是return false。
参数说明和之前介绍的onJsAlert()相同。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onExceededDatabaseQuota(String url, String databaseIdentifier,  
            long quota, long estimatedDatabaseSize, long totalQuota,  
            WebStorage.QuotaUpdater quotaUpdater)  

通知应用程序webview内核web sql 数据库超出配额，请求是否扩大数据库磁盘配额。默认行为是不会增加数据库配额。
参数说明：

@param url 触发这个数据库配额的url地址
@param databaseIdentifier 	指示出现数据库超过配额的标识。
@param quota   原始数据库配额的大小，是字节单位bytes
@param estimatedDatabaseSize  到达底线的数据大小 bytes
@param totalQuota 总的数据库配额大小 bytes
@param quotaUpdater 更新数据库配额的对象，可以使用 quotaUpdater.updateQuota(newQuota);配置新的数据库配额大小。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onReachedMaxAppCacheSize(long requiredStorage, long quota,  
            WebStorage.QuotaUpdater quotaUpdater)  

 通知应用程序内核已经到达最大的appcache。
appcache是HTML5针对offline的一个数据处理标准。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void onGeolocationPermissionsShowPrompt(String origin,  
            GeolocationPermissions.Callback callback)  

当前页面请求是否允许进行定位。

GeolocationPermissions.Callback的使用

public void invoke(String origin, boolean allow, boolean retain);

参数说明：
@param origin 权限设置的源地址
@param allow 是否允许定位
@retain 当前的选择是否让内核记住。


public void onGeolocationPermissionsHidePrompt()  


public void openFileChooser(ValueCallback<Uri> uploadFile, String acceptType, String capture)  
这个回调是私有回调， 当页面需要请求打开系统的文件选择器，则会回调这个方法，比如我们需要上传图片，请求拍照，邮件的附件上传等等操作。
如果不实现这个私有API，则上面的请求都将不会执行。
一个普通网页的加载cache会被检查，内容也会被重新校验，第一次访问网页时，会存储cache到本地，设置策略可以让网页加载方式发生变化，cache模式有如下几种:
LOAD_DEFAULT: 如果我们应用程序没有设置任何cachemode， 这个是默认的cache方式。 加载一张网页会检查是否有cache，如果有并且没有过期则使用本地cache，否则                                   从网络上获取。
LOAD_CACHE_ELSE_NETWORK: 使用cache资源，即使过期了也使用，如果没有cache才从网络上获取。
LOAD_NO_CACHE: 不使用cache 全部从网络上获取
LOAD_CACHE_ONLY:  只使用cache上的内容。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public void setLoadWithOverviewMode(boolean overview)  
概览模式的设置，默认指是false。

[java] view plaincopy在CODE上查看代码片派生到我的代码片
public synchronized void setJavaScriptEnabled (boolean flag)  
默认值是false.  如果我们网页需要javascript时，需要开启这个设置，否则网页加载不全。

2.WebSettings Webkit中的实现
    WebSeetings 的API在Android 系统各个版本变化不大只是增加API或者deprecate一些API，但其内部是实现在不同版本中确有些差异，目前主流android系统版本主要为Android 4.0以上，分析4.0以上各系统版本得出webseetings的实现分为三块Android 4.0, Android 4.1---4.3,Android 4.4。下面分析下Android 以上各版本间的实现：

1）Android 4.0系统 主要分为两部分，一部分是API层，另一部分Settings的存储位置。
    Settings存储位置大部分的setting最终设置到WebCore当中的Settings.cpp， 比如javaScriptEnable等
    还有一部分根据模块相关存储在模块内部，比如CacheMode存储在FrameLoader当中。
2）Android 4.1--4.3系统对WebView的 framework进行重构，WebSettings相应也跟着变化。
      中间引入了桥阶层WebSettingsClassc。
 Settings存储位置大部分的setting最终设置到WebCore当中的Settings.cpp， 比如javaScriptEnable等
   还有一部分跟平台相关的存储在WebCoreSupport层相应模块中，比如在4.1---4.3上CacheMode存储在WebRequestContext



在Android 4.4上WebView底层实现换成了chromium，为了兼容老的WebSettings的接口，Android 4.4做了chromium 的桥阶层，主要涉及的WebSettings相关代码在
ContentSettingsAdapter,AwSettings中。
和前面的一些版本相同的是大部分settings还是存储在Webkit的Settings.cpp中，这边简单介绍下chromium 使用的blink渲染引擎，而blink是从webkit当中剥离出来的，还保留了webkit的parsing等。因此和我们之前看到的Settings.cpp存储在WebCore目录，目录结构会有所不同。
还有一部分settings在Android 4.4上存储方式也是存储在platform porting层。 下面是一个关于cachemode这个设置的分析:




