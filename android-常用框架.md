​	Android程序中，对于线程的管理是非常严格的。大的来说，分为两大类，一类是MAIN(ui)线程，一类是background线程。MAIN线程是主线程，主要用于修改页面显示，跳转等，所有不适合运行耗时的代码，可能会发生程序无响应（卡死）的情况。background线程，主要用于后台操作，不能修改页面显示，适合耗时的操作。

### OKio

#### ByteString

。。。

#### BufferedSource

。。。

#### BufferedSink

。。。

### OKHttp

。。。

### EventBus

组件间的通信，事件订阅和发布。

不使用EventBus的情况下，组件间的事件通信有两种方案。

1.自定义监听器，自定义回调函数。

2.使用本地广播。（LocalBroadcastManager）

​	使用EventBus，1.声明事件对象。2.添加订阅回调函数（@Subscribe）。3.发布事件。只需要添加依赖后，声明事件对象，以及使用相关注解即可使事件间通信。

#### ThreadMode

​	因为android对线程管理非常严格，EventBus提供了可以根据需求来调整事件运行的线程位置。

POSTING：指的是你希望你的订阅者运行的线程和发布者相同。（在@Subscribe(threadMode = ThreadMode.POSTING)声明。）

​	如果希望你的代码稳定运行在ui线程，可以调用runOnUiThread来修改页面显示。

MAIN：EventBus将订阅回调方法，执行在MAIN线程中。（此方法会被阻塞，也就是发布者后续代码，必须等待订阅方代码执行完成后，再执行后续操作。）

MAIN_ORDERED：订阅者回调执行代码速度的快慢，不阻塞发布者代码运行。

BACKGROUND：订阅回调运行在非UI线程上

ASYNC：异步订阅回调，一定会运行在一个新的线程上。这种模式适合耗时操作。

#### *粘性事件*(Sticky)

事件先发布，再订阅。

@Subscribe(sticky = true)

#### 混淆规则(ProGuard)

​	因为在apk打包时，程序会尝试优化apk包，可能会把打包时没有调用到的代码或者方法删除。因为EventBus是通过反射调用的，所以声明了@Subscribe的方法，在编译器检查时，都是没有调用的方法，可能会在打包时被删除。这个时候，需要使用官方提供的proguard。

proguard-rules.pro文件中加入：

````properties
-keepattributes *Annotation*
-keepclassmembers class * {
    @org.greenrobot.eventbus.Subscribe <methods>;
}
-keep enum org.greenrobot.eventbus.ThreadMode { *; }

# Only required if you use AsyncExecutor
-keepclassmembers class * extends org.greenrobot.eventbus.util.ThrowableFailureEvent {
    <init>(java.lang.Throwable);
}
````

### RecyclerView

线性布局-网格布局-瀑布流布局切换view。

### Glide

使用

````java
/**
     * 使用Glide加载网络图片
     * @param img 网络图片地址
     */
    private void glideLoadImage (String img) {
//      通过 RequestOptions 对象来设置Glide的配置
        RequestOptions options = new RequestOptions()
//                设置图片变换为圆角
                .circleCrop()
//                设置站位图
                .placeholder(R.mipmap.loading)
//                设置加载失败的错误图片
                .error(R.mipmap.loader_error);

//      Glide.with 会创建一个图片的实例，接收 Context、Activity、Fragment
        Glide.with(this)
//                指定需要加载的图片资源，接收 Drawable对象、网络图片地址、本地图片文件、资源文件、二进制流、Uri对象等等
                .load(img)
//                指定配置
                .apply(options)
//                用于展示图片的ImageView
                .into(mIv);
    }

    /**
     * 使用GlideApp进行图片加载
     * @param img
     */
    private void glideAppLoadImage (String img) {
        /**
         * 不想每次都通过 .apply(options) 的方式来进行配置的时候，可以使用GlideApp的方式来进行全局统一的配置
         * 需要注意以下规则：
         * 1、引入 repositories {mavenCentral()}  和 dependencies {annotationProcessor 'com.github.bumptech.glide:compiler:4.8.0'}
         * 2、集成 AppGlideModule 的类并且通过 @GlideModule 进行了注解
         * 3、有一个使用了 @GlideExtension 注解的类 MyGlideExtension，并实现private的构造函数
         * 4、在 MyGlideExtension 可以通过被 @GlideOption 注解了的静态方法来添加可以被GlideApp直接调用的方法，该方法默认接受第一个参数为：RequestOptions
         */
       GlideApp.with(this)
               .load(img)
//               调用在MyGlideExtension中实现的，被@GlideOption注解的方法，不需要传递 RequestOptions 对象
               .injectOptions()
               .into(mIv);
    }
````

使用Generated API全局配置glide

````java
@GlideExtension
public class MyGlideExtension{

    /**
     * 实现private的构造函数
     */
    private MyGlideExtension() {
    }

    @GlideOption
    public static void injectOptions (RequestOptions options) {
        options
//                设置图片变换为圆角
                .circleCrop()
//                设置站位图
                .placeholder(R.mipmap.loading)
//                设置加载失败的错误图片
                .error(R.mipmap.loader_error);

    }
}
````

````java
/**
 * 帮助我们生成 GlideApp 对象
 */
@GlideModule
public class MyAppGlideModule extends AppGlideModule {
}
````

### GreenDao

使用步骤：

1.创建实体类

2.生成对应的DaoMaster、DaoSession、Dao

3.通过Dao对象完成增删改查

#### 数据库加密

sqlcipher

### 极光推送

。。。

### WebView

#### nodejs

安装nodejs作为服务端：

1.https://nodejs.org/zh-cn/（node -v测试安装成功）

2.使用淘宝镜像cnpm代替默认的npm。`$ npm install -g cnpm --registry=https://registry.npm.taobao.org`

3.安装http-server：`cnpm install http-server -g`

4.启动http-server：到html文件所在目录，输入命令：`http-server`即可启动服务器。默认运行在8080端口。如果8080被占用，可以使用：`http-server -p 8888`改端口号。

#### 加载网页的4种方式

1.loadUrl(String url)

2.loadUrl(String url, Map additionalHttpHeaders)

3.loadData(String data, String mimeType, String encoding)

4.loadDataWithBaseURL(String baseUrl, String data, String mimeType, String encoding, String historyUrl)

#### 控制网页前进后退

canGoBack、goBack、canGoForward、goForward、canGoBackOrForward(int steps) 、goBackOrForward(int steps) 、clearHistory() 

#### webview状态管理

（需要配合activity的生命周期使用）

onResume() ：激活WebView为 活跃状态，能正常执⾏⽹⻚的响 应

onPause()：当⻚⾯被失去焦点被切换到后台不可⻅ 状态，需要执⾏onPause ，通过onPause动作通知内 核暂停所有的动作，⽐如DOM的解析、plugin的执 ⾏、JavaScript执⾏。

pauseTimers()：当应⽤程序(存在webview)被切换到后台时， 这个⽅法不仅仅针对当前的webview⽽是全局的全应⽤程序的 webview ，它会暂停所有webview的layout，parsing， javascript。降低CPU功耗。

resumeTimers()：恢复 pauseTimers状态

destroy()：销毁Webview ,在关闭了Activity时 , 如果不调⽤ webview的destroy()⽅法，那么在你activity关闭时，webview 本身并不会被销毁，就会出现内存溢出的问题。

#### 常用类

##### WebSettings

对webview进行配置管理

控制js代码运行

````java
WebSettings webSettings = mWebView.getSettings();
        webSettings.setJavaScriptEnabled(true);
````

控制网页缩放

````java
webSettings.setSupportZoom(true);
webSettings.setBuiltInZoomControls(true);
webSettings.setDisplayZoomControls(true);
````

控制缓存策略

````java
webSettings.setCacheMode(WebSettings.LOAD_CACHE_ONLY);
webSettings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);
webSettings.setCacheMode(WebSettings.LOAD_DEFAULT);//默认
webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);
````

##### WebViewClient

处理网页加载时各种回调通知。如果没有设置webviewclient，android会自动调用系统的浏览器。就会跳出当前app。

6种回调函数

````java
mWebView.setWebViewClient(new WebViewClient() {

            @Override
            public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
                super.onReceivedError(view, errorCode, description, failingUrl);
                Log.e("WebViewActivity","webview-》onReceivedError : 加载了url：" + failingUrl + " - 错误描述：" + description+ " - 错误代码：" + errorCode);
                view.loadUrl("http://192.168.2.124:3000/");
            }


            @RequiresApi(api = Build.VERSION_CODES.M)
            @Override
            public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
                super.onReceivedError(view, request, error);
                Log.e("WebViewActivity","webview-》onReceivedError (android6.0以上调用) : 加载了url：" + request.getUrl().toString() + " - 错误描述：" + error.getDescription()+ " - 错误代码：" + error.getErrorCode());
            }

            @Override
            public boolean shouldOverrideUrlLoading(WebView view, String url) {
                Log.e("WebViewActivity","webview-》shouldOverrideUrlLoading : 加载了url：" + url);
                if ("http://www.baidu.com/".equals(url)) {
//                    view.loadUrl("http://www.sogou.com/");
                    Toast.makeText(WebViewActivity.this, "webview-》shouldOverrideUrlLoading : 加载了url：" + url, Toast.LENGTH_SHORT).show();
                    return true;
                }

                Uri uri = Uri.parse(url);
                if ("android".equals(uri.getScheme())) {
                    String functionName = uri.getAuthority();
                    if ("print".equals(functionName)) {
                        String msg = uri.getQueryParameter("msg");
                        print(msg);
                        return true;
                    }
                }

                return super.shouldOverrideUrlLoading(view, url);
            }

            @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
            @Override
            public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
                Log.e("WebViewActivity","webview-》shouldOverrideUrlLoading(Android7.0以上调用) : 加载了url：" + request.getUrl().toString());
                return super.shouldOverrideUrlLoading(view, request);
            }

            @Override
            public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
                WebResourceResponse result = super.shouldInterceptRequest(view, url);
                Log.e("WebViewActivity","webview-》shouldInterceptRequest请求了url：" + url);
                Log.e("WebViewActivity", "result = " + result);
//                if ("http://www.baidu.com/".equals(url)) {
//                    return new WebResourceResponse("text/html", "utf-8", null);
//                }
                return result;
            }

            @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
            @Override
            public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
                Log.e("WebViewActivity","webview-》shouldInterceptRequest请求了(android5.0之上调用)url：" + request.getUrl().toString());
                return super.shouldInterceptRequest(view, request);
            }

            @Override
            public void onPageStarted(WebView view, String url, Bitmap favicon) {
                super.onPageStarted(view, url, favicon);
                Log.e("WebViewActivity","webview-》onPageStarted 网页开始进行加载url：" + url);
            }

            @Override
            public void onLoadResource(WebView view, String url) {
                super.onLoadResource(view, url);
                Log.e("WebViewActivity","webview-》onLoadResource 网页开始加载资源url：" + url);
            }

            @Override
            public void onPageFinished(WebView view, String url) {
                super.onPageFinished(view, url);
                Log.e("WebViewActivity","webview-》onPageFinished 网页已经加载完成url：" + url);
            }
        });
````

##### WebChromeClient

辅助webview处理javascript对话框、标题进度等

主要有以下5个回调方法

````java
        mWebView.setWebChromeClient(new WebChromeClient() {
            @Override
            public void onProgressChanged(WebView view, int newProgress) {
                super.onProgressChanged(view, newProgress);
                Log.e("webViewActivity", "newProgress:" + newProgress);
            }

            @Override
            public void onReceivedTitle(WebView view, String title) {
                super.onReceivedTitle(view, title);
                Log.e("webViewActivity", "title:" + title);
            }

            @Override
            public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
                boolean res = super.onJsAlert(view, url, message, result);
                res = true;
                Log.e("webViewActivity", "onJsAlert - url : " + url + " - message : " + message + "  - res : " + res);
                Toast.makeText(WebViewActivity.this, message, Toast.LENGTH_SHORT).show();
                result.confirm();
                return res;
            }

            @Override
            public boolean onJsConfirm(WebView view, String url, String message, final JsResult result) {
                boolean res = super.onJsConfirm(view, url, message, result);
                res = true;
                Log.e("webViewActivity", "onJsConfirm - url : " + url + " - message : " + message + "  - res : " + res);
                AlertDialog.Builder builder = new AlertDialog.Builder(WebViewActivity.this);
                builder.setMessage(message);
                builder.setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        result.confirm();
                    }
                });

                builder.setNegativeButton("取消", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        result.cancel();
                    }
                });
                builder.create().show();
                return res;
            }

            @Override
            public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, final JsPromptResult result) {
                boolean res = super.onJsPrompt(view, url, message, defaultValue, result);
                res = true;
                Log.e("webViewActivity", "onJsConfirm - url : " + url + " - message : " + message + " - defaultValue : " + defaultValue + "  - res : " + res);
                AlertDialog.Builder builder = new AlertDialog.Builder(WebViewActivity.this);
                builder.setMessage(message);
                builder.setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        result.confirm("这是点击了确定按钮之后的输入框内容");
                    }
                });

                builder.setNegativeButton("取消", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        result.cancel();
                    }
                });
                builder.create().show();
                return res;
            }
        });
````

#### android调用js代码

有两种方法：

````java
//1.
mWebView.loadUrl("javascript:alert(sum(2, 3))");
//2.
public void onSumFromEVJS (View v) {
        mWebView.evaluateJavascript("javascript:sum(2, 3)", new ValueCallback<String>() {
            @Override
            public void onReceiveValue(String s) {
                Toast.makeText(WebViewActivity.this, "evaluateJavascript - " + s, Toast.LENGTH_SHORT).show();
            }
        });
    }
````

#### js调用android代码

1.通过拦截JavaScript请求的回调方法

2.对象映射

````java
public class DemoJsObject {

    @JavascriptInterface
    public String print (String msg) {
        Log.e("DemoJsObject", "msg ：" + msg);
        return "这是android的返回值";
    }
}
````

### ButterKnife

#### ButterKnife Zelezny插件