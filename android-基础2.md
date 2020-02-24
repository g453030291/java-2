### service

四大组件之一，没有界面，生命周期和activity类似，适合在后台自动运行。

生命周期：

![android-service生命周期](https://github.com/g453030291/java-2/blob/master/images/android-service生命周期.png)

示例代码：

````java
public class MyService extends Service {
    public MyService() {
    }

    private int i;
    //创建[34                                                                                                                                                                                                                                  
    @Override
    public void onCreate() {
        super.onCreate();
        Log.e("TAG","服务创建了");
        //开启一个线程（从1数到100），用于模拟耗时的任务
        new Thread(){
            @Override
            public void run() {
                super.run();
                try {
                    for (i = 1; i <= 100; i++) {
                        sleep(1000);
                    }
                }catch (Exception e){
                    e.printStackTrace();
                }
            }
        }.start();
    }

    //启动
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.e("TAG","服务启动了");
        return super.onStartCommand(intent, flags, startId);
    }

    //绑定
    //IBinder：在android中用于远程操作对象的一个基本接口
    @Override
    public IBinder onBind(Intent intent) {
        Log.e("TAG","服务绑定了");
        return new MyBinder();
    }

    //对于onBind方法而言，要求返回IBinder对象
    //实际上，我们会自己定义一个内部类，集成Binder类

    class MyBinder extends Binder{
        //定义自己需要的方法（实现进度监控）
        public int getProcess(){
            return i;
        }
    }

    //解绑
    @Override
    public boolean onUnbind(Intent intent) {
        Log.e("TAG","服务解绑了");
        return super.onUnbind(intent);
    }

    //摧毁
    @Override
    public void onDestroy() {
        super.onDestroy();
        Log.e("TAG","服务销毁了");
    }
}
````

#### ServiceConnection

两个重要方法：

onServiceConnected()、onServiceDisconnected()

#### IBinder

利用ibinder对象，实现activity和service的通信。

### AIDL

进程间的通信接口

远程启动、绑定、解绑服务。1.manifest文件声明包名。2.intent引入远程包名即可远程控制。

远程通信数据。（查询任务的进度）创建aidl文件，两个项目同时引入。

### ContentProvider

​	四大组件之一，为存储和获取数据提供统一接口。可以在不同应用之间共享数据。对于ContentProvider而言，他认为所有数据都是表，然后把数据组织成表格。（ContentProvider其实就是对外暴露一个接口，供外部调用）

应用场景：微信调用手机联系人。

### Socket&Https

Socket两种通信模型：

TCP：类似于打电话，是持续的

UDP：类似于Email，是一个一个的包来回发送

### 动画

#### 逐帧动画

操作的对象是UI切出的原图

AnimationDrawable

`<animation-list>`

````java
public class FrameAnimationActivity extends AppCompatActivity {

    private AnimationDrawable animationDrawable;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_frame_animation);

        View view = findViewById(R.id.view);
        animationDrawable = (AnimationDrawable) view.getBackground();
        animationDrawable.setOneShot(true);
    }

    public void onClick(View view) {
        switch (view.getId()) {
            case R.id.btnStart:
                animationDrawable.start();
                break;
            case R.id.btnStop:
                animationDrawable.stop();
                break;
        }
    }
}
````

#### 视图动画

操作的对象是android中的视图对象

类：android.view.animation

接口：Animation

​				AlphaAnimation：透明度动画

​				ScaleAnimation：缩放动画

​				TranslateAnimation：位移动画

​				RotateAnimation：旋转动画

​				AnimationSet：集合动画

Interpolator：设定动画速率。（插值器）

#### 属性动画

操作的是对象的值。视图动画，其实并没有改变属性值。属性动画是不断改变属性值。

ValueAnimator：改变属性的值

ObjectAnimator：改变哪个对象的属性

ViewPropertyAnimator：调节视图属性的值

### 转场动画

### 自定义View

### Surface View

### Android中的事件分发机制

