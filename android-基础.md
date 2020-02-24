## Android

四大组件：activity、service、broadcast receive、content provider

### Activity

#### 声明一个activity

​	继承AppCompatActivity类重写onCreat方法，通过setContentView配置layout（布局）文件。一个activity绑定一个layout，activity文件控制逻辑，layout的xml控制页面布局。

​	AndroidManifest.xml（重要）。指定所有的activity在这里声明，并且需要指定默认启动的activity。

#### Activity跳转

```java
Intent intent = new Intent(this,NewActivity.class);
startActivity(intent);
```

#### 四种启动模式

standard、singleTop、singleTask、singleInstance

可以在AndroidManifest.xml的activity标签中，设置属性android:launchMode="standard"。

##### standard

​	默认栈模式

##### singTop

​	顶部复用模式

##### singleTask

​	整个任务栈中只有他一个，如果它上面还有其它activity，会被直接清除掉

##### singleInstance

​	独占一个任务栈，类似于单例模式，来电话的activity就是典型的singleInstance模式

#### Activity启动方式

##### 显式启动

​	就是使用intent引入跳转的class方式启动

##### 隐式启动

````java
Intent it = new Intent(Intent.ACTION_VIEW,Uri.parse("www.baidu.com"));//以协议决定具体调用什么系统Activity，以上就会自动调用浏览器，打电话可以使用tel:18888888888
startActivity(it);
````

##### startActivityForResult

````java
Intent it6 = new Intent(this,Main2Activity.class);
//参数：请求码
startActivityForResult(it6,1000);
-----------------------------------------------------------------
//如果时通过startActivityForResult  的方式启动了第二个Activity
    //当第二个Activity处理结束后，再回到的当前Activity时，一定会自动回调onActivityResult
    //在该方法中我们可以处理第二个Activity返回的结果（如：拍照后得到的照片，从图库中选取的图片）
    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        //resultCode:  0 , RESULT_CANCEL  取消    -1，RESULT_OK 正确处理完后返回
        if(resultCode == RESULT_OK) {
            //requestCode:用来区分该结果是来自于哪个Activity
            if (requestCode == 1000) {
                Log.e("TAG", "自动进入了onActivityResult，requestCode=" + requestCode + ",resultCode=" + resultCode);
                Log.e("TAG","返回的数据是："+data.getStringExtra("myMsg"));
            }
        }
    }
````

#### Activity之间信息传递

##### 传递简单字符串

​	使用intent.putExtra("msg","数据");

##### 传递Java对象

​	实体类实现Serializable接口即可

#### Activity生命周期

![activity生命周期](https://github.com/g453030291/java-2/blob/master/images/activity生命周期.png)

````java
//单个Activity的生命周期：
    //1.正常启动onCreate-->onStart-->onReusme , 正常退出onPause-->onStop-->onDestory,
    // 再次启动onCreate-->onStart-->onResume
    //2.已经处于前台的Activity，点击主页按钮离开当前Activity，OnPause-->onStop,
    // 回到Activity:onRestart-->onStart-->onResume
    //3.Activity不可操作onPause-->onStop（如：息屏，打开了其他Activity），而应用被强行杀死了，
    //再回到Activity，onCreate-->onStart-->onResume

    //多个Activity切换时
    //当启动另一个Activity时，当前Activity：onPause-->onStop,当点击返回按钮，
    //使另一个Activity退出时，当前Activity：onRestart-->onStart-->onResume

    //对话框存在时
    //1.普通对话框对生命周期没有任何影响
    //2.如果有个Activity伪装成对话框模式，那么当它启动时，之前的Activity：onPause
    //“对话框”消失后，回调onResume再次回到前台
````

### Fragment

​	主要是适配大屏设备

Fragment和Activity的关系：

1.Fragment是3.0以后出现的

2.一个Activity可以运行多个Fragment

3.Fragment不能脱离Activity存在

4.Activity是屏幕主体，而Fragment是Activity的一个组成元素

#### 生命周期

![fragment生命周期](https://github.com/g453030291/java-2/blob/master/images/fragment生命周期.png)

#### 静态加载

​	创建fragment直接右键new即可。然后在xml文件中的fragment标签内，设置属性android:name="fragment的类路径"。

#### 动态加载

​	xml文件中声明一个framelayout，待使用java代码来动态将fragment加载到framelayout中。

​	有两个关键的对象，FragmentManager、FragmentTransaction。(以下代码为动态加载fragment和修改fragment的文本)

````java
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Log.i("TAG","Activity--onCreate");

        //FragmentManager   FragmentTransaction

        //1.获取Fragment管理器
        FragmentManager manager = getSupportFragmentManager();

        //2.获取Fragment事务（/开启事务）
        FragmentTransaction transaction = manager.beginTransaction();

        //3.动态添加Fragment
        //参数1：容器id
        //参数2：Fragment对象
        final Fragment f2 = new Fragment2();
        transaction.add(R.id.container,f2);

        //4.提交事务
        transaction.commit();

        findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //1.获取Fragment管理器
                FragmentManager manager = getSupportFragmentManager();
                //2.获取Fragment对象
                Fragment f1 = manager.findFragmentById(R.id.fragment1);
                //3.获取Fragment对象的视图
                View v1 = f1.getView();

                //4.再视图里找到指定的控件
                TextView txt1 = v1.findViewById(R.id.txt1);

                //5.修改控件内容
                txt1.setText("这是动态生成的内容");


                View v2 = f2.getView();
                TextView txt2 = v2.findViewById(R.id.txt2);
                txt2.setText("这个Fragment2的动态内容");
            }
        });

    }
````

#### 传值

##### Activity传值到Fragment

第一种方法：

````java
//传值
//setArguments
                //1.实例化Fragment
                Fragment f1 = new Fragment1();
                //2.实例化一个Bundle对象
                Bundle bundle = new Bundle();
                //3.存入数据到Bundle对象中
                bundle.putString("msg1","这是由Activity发往Fragment的数据");
                //4.调用Fragment的setArguments方法，传入Bundle对象
                f1.setArguments(bundle);
                //5.添加/替换显示的Fragment
                transaction.replace(R.id.container,f1);
-----------------------------------------------------------------
//取值
Bundle b = getArguments();
        String msg1 = b.getString("msg1");
````

第二种方法：

````java
//取值
@Override
    public void onAttach(Context context) {
        super.onAttach(context);
        String msg = ((TabFragmentActivity)context).sendMsg();
        Toast.makeText(context,msg,Toast.LENGTH_SHORT).show();
        Log.e("TAG","onAttach--Fragment与Activity关联");
    }
````

##### Fragment传值到Activity

````java
//传值
//Fragment像Activity传值（接口回调）
    //1.定义一个接口，在该接口中声明一个用于传递数据的方法
    //2.让Activity实现该接口，然后重写回调方法，获取传入的值，然后做处理
    //3.在自定义Fragment中，声明一个回调接口的引用
    //4.在onAttach中法中，为第三步的引用赋值
    //5.用引用调用传递数据的方法
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.fragment_fragment3, container, false);
    }

    private MyListener ml;

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        ml = (MyListener) getActivity();
        ml.sendMsg("消息");
    }

    public interface  MyListener{
        public void sendMsg(String msg);
    }
-----------------------------------------------------------------
//取值
    @Override
    public void sendMsg(String msg) {
        Log.e("TAG","Fragment传回的数据："+msg);
    }
````

##### Fragment传值到Fragment

​	由于 Fragment跳转过程中都需要fragment实例，就可以在fragment中直接声明一个变量，另一边取值即可。

动态加载实例

​	实现一个app首页底部4个导航切换的界面。底部放置RadioGroup配置4个单选按钮，设置android:drawableTop= ""。然后使用selector来获取选中状态，选中/未选中状态的样式在这类声明，xml的布局文件相应标签引入即可 。（selector是声明在drawable文件夹下）。至于切换fragment，只需要给RadioButton添加onClick事件即可。

### UI基础

#### View(UI基础控件)

##### TextView

![textview继承关系](https://github.com/g453030291/java-2/blob/master/images/textview继承关系.png)

| 通用属性                                     | 可选值                                                       |
| -------------------------------------------- | ------------------------------------------------------------ |
| Android:layout_width\android:layout_height   | Match_parent\wrap_content\dp                                 |
| Android:id                                   | @id/valName\@+id/valName                                     |
| Android:layout_margin                        | dp                                                           |
| Android:padding                              | dp                                                           |
| Android:background                           | 十六进制颜色值、@mipmap/resourceId                           |
| Android:layout_gravity（相对于父容器的偏向） | Center_horizontal\center_vertical\center\left\right\top\bottom |
| Android:gravity（控件内部组件偏向）          | Center_horizontal\center_vertical\center\left\right\top\bottom |
| Android:visibility                           | Visible\invisible\gone                                       |

##### EditText

常用属性：android:inputType、android:hint、android:maxLength

##### Button

注册点击事件的方法：

1.自定义内部类

2.匿名内部类

3.当前Activity去实现接口

4.在布局文件中添加点击事件属性

​	前三种都是实现接口，View.OnClickListener。不过是实现接口的class不同而已。最后一种可以用xml直接声明android:onclick=xxx属性，activity中声明一个同名xxx方法，形参为View即可。

##### ImageView

指定前景图片资源：android:src（前景会一直保持图片长宽比例，图片不会变形）

指定背景图片资源：android:background（背景会优先适配设备屏幕，图片会变形）

##### ProgressBar

​	默认样式：圆形旋转，修改样式需要设置style=”?android:attr/progressBarStyleHorizontal“。设置进度：android:progress。设置最大值（默认100）:android:max。设置永恒滚动：android:indeterminate="true"。

#### Layout

线性布局（LanearLayout）、相对布局（RelativeLayout）、帧布局（FrameLayout）、表格布局（TableLayout）、网格布局（GridLayout）、约束布局（ConstraintLayout） 

##### LanearLayout

重要属性：

| 属性                   | 含义 |
| ---------------------- | ---- |
| android:orientation    | 方向 |
| android:layout_weight  | 权重 |
| android:layout_gravity | 重力 |

##### RelativeLayout

| 属性                            | 含义                                                 |
| ------------------------------- | ---------------------------------------------------- |
| android:layout_alignParentRight | 相对于父容器（true、false）                          |
| android:layout_toRightOf        | 相对于其它控件（其它控件id）（在参照物的某边）       |
| android:layout_alignTop         | 相对于其它控件（其它控件id）（和参照物的某边线对齐） |

##### FrameLayout

重要属性：

控件重力：android:layout_gravity

前景：android:foreground

前景重力：android:foregroundGravity

##### TableLayout

重要属性：android:stretchColumns(设置可伸展的列，直接传列的索引，如果有多列，以逗号作为分隔，*表示全部)、android:shrinkColumns（设置可缩小的列）、android:collapseColumns（设置可隐藏的列）

##### GridLayout

重要属性：android:columnCount(列数量)、android:layout_columnSpan(跨几列)、android:layout_columnWeight(列权重)

column表示列，可更换为row即更换为行属性

##### ConstraintLayout

属性分为两类：位置设置（类似于相对布局，需要找参照物 ）、偏移量设置

位置设置：

app:layout_constraint方位_to方位Of="?"

app:layout_constraintLeft_toLeftOf="?"

?:可填parent或其他控件id

偏移量设置：取值0~1

app:layout_constraintVertical_bias="0.53" 垂直偏移量，0.5在正中间

app:layout_constraintHorizontal_bias="0.53" 水平偏移量，0.5在正中间

### UI常用组件

#### menu

注意：

1.xml定义的Menu不显示 onCreateOptionsMenu()方法必须返回true

2.onOptionsItemSelected方法返回true

3.调用父类的默认实现(swich语句中可能不能覆盖所有情况，一般都会在结尾加上一个default：super.onOptionsItemSelected(item))

##### 选项菜单（OptionMenu）

​	是一个应用的主菜单项，用于放置对应用产生全局影响的操作。

​	res新建menu文件夹，新建option.xml文件，选择布局为menu，其中根布局默认为menu，子菜单项为item标签。item中最多可以再嵌套一层menu。然后在activity中，重写onCreateOptionsMenu方法。使用`getMenuInflater().inflate(R.menu.option,menu)`。方法返回值为true即可显示右上角三个点的OptionMenu。item有app:showAsAction="always"，即可将菜单快捷显示在顶部操作栏。 OptionMenu的选中方为activity中重写onOptionsItemSelected方法。

##### 上下文菜单（ContextMenu）

​	长按某个view不放，就会在屏幕中间弹出ContextMenu。

​	menu文件夹下新建context.xml文件，文件类型为menu resource file。activity中重写onCreateContextMenu方法，`getMenuInflater().inflate(R.menu.context,menu)`。并在onCreate方法中注册contextMenu（`registerForContextMenu(findViewById(R.id.ctx_btn))`）。重写onContextItemSelected()方法获取选中项。

​	还有一种上下文操作模式，即长按后将选项显示在顶部ActionBar中。

##### 弹出菜单（PopupMenu）

​	模态形式展示的弹出风格菜单，绑在某个view上，一般出现在被绑定的view下方。

````java
//popup_btn:演示PopupMenu
        final Button popupBtn = findViewById(R.id.popup_btn);
        popupBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //①实例化PopupMenu对象 (参数2：被锚定的view)
                final PopupMenu menu = new PopupMenu(MainActivity.this,popupBtn);
                //②加载菜单资源：利用MenuInflater将Menu资源加载到PopupMenu.getMenu()所返回的Menu对象中
                //将R.menu.xx对于的菜单资源加载到弹出式菜单中
                menu.getMenuInflater().inflate(R.menu.popup,menu.getMenu());
                //③为PopupMenu设置点击监听器
                menu.setOnMenuItemClickListener(new PopupMenu.OnMenuItemClickListener() {
                    @Override
                    public boolean onMenuItemClick(MenuItem menuItem) {
                        switch (menuItem.getItemId()){
                            case R.id.copy:
                                Toast.makeText(MainActivity.this,"复制",Toast.LENGTH_SHORT).show();
                                break;
                            case R.id.paste:
                                Toast.makeText(MainActivity.this,"粘贴",Toast.LENGTH_SHORT).show();
                                break;
                        }
                        return false;
                    }
                });
                //④千万不要忘记这一步,显示PopupMenu
                menu.show();
            }
        });
````

#### 对话框

##### AlertDialog

常用方法：setTitle、setMessage、creat、show

````java
AlertDialog.Builder builder = new AlertDialog.Builder(this);
                builder.setTitle("提示");
                builder.setMessage("确定退出程序吗");
                builder.setPositiveButton("确定", new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialogInterface, int i) {
                        finish();
                    }
                });
                builder.setNegativeButton("ȡ?",null);
                builder.show();
````

##### 自定义DiaLog

##### PopupWindow

#### 适配器

##### ArrayAdapter

​	用来处理单一的文本信息。

##### SimpleAdapter

。。。

##### BaseAdapter

。。。

##### SimpleCursorAdapter

。。。

#### ListView

。。。

#### **ExpandableListView**

​	继承自listview，就是一个可分组的listview。只要涉及到列表，并且列表需要分组就可以使用listview。

常用属性：

groupIndicator、childIndicator、childDivider

常用方法：

setAdapter(ExpandableListAdapter)

setOnGroupClickListener

setOnChildClickListener

setOnGroupCollapseListener

setOnGroupExpandListener

#### ViewPager2

包：`androidx.viewpager2:viewpager2`

。。。

