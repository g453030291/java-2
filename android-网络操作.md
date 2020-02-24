Android9.0对访问http地址的限制解决方案：

手动在res文件夹下创建xml文件夹，创建network_security_config.xml文件

````xml
<?xml version="1.0" encoding="utf-8"?> 
<network-security-config>
	<base-config cleartextTrafficPermitted=”true” /> 
</network-security-config>
````

manifest配置中添加属性：

````xml
<application android:networkSecurityConfig="@xml/netwo rk_security_config" tools:targetApi=”n”... >
````

### Handler通信

#### 概念

​	多线程间的通信就是用handler的消息队列。

![android-handler](https://github.com/g453030291/java-2/blob/master/images/android-handler.png)

​	handler是android实现的消息队列。每个线程有自己的MessageQueue。MessageQueue中可以存放message或者runnable。Looper其实是一个死循环，不停地向MessageQueue中取出消息。只有MainActivity是自动开启loop循环的，如果是子线程，需要手动开启后，才可以使用MessageQueue。

#### 实例

1.下载示例（子线程下载文件，利用handler发送消息给主线程更新ProgressBar的进度条）

````java
public class DownloadActivity extends Activity {

    public static final int DOWNLOAD_MESSAGE_CODE = 100001;
    private static final int DOWNLOAD_MESSAGE_FAIL_CODE = 100002;
    public static final String APP_URL = "http://download.sj.qq.com/upload/connAssitantDownload/upload/MobileAssistant_1.apk";
    private Handler mHandler;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_download);

        final ProgressBar progressBar = (ProgressBar) findViewById(R.id.progressBar);


        /**
         * 主线程  -- > start
         * 点击按键 |
         * 发起下载 |
         * 开启子线程做下载 |
         * 下载过程中通知主线程 |  -- > 主线程更新进度条
         */

        findViewById(R.id.button2).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        download(APP_URL);
                    }
                }).start();

            }
        });

        mHandler = new Handler() {
            @Override
            public void handleMessage(Message msg) {
                super.handleMessage(msg);

                switch (msg.what) {
                    case DOWNLOAD_MESSAGE_CODE:
                        progressBar.setProgress((Integer) msg.obj);
                        break;
                    case DOWNLOAD_MESSAGE_FAIL_CODE:

                }
            }
        };

    }

    private void download(String appUrl) {
        try {
            URL url = new URL(appUrl);
            URLConnection urlConnection = url.openConnection();

            InputStream inputStream = urlConnection.getInputStream();

            /**
             * 获取文件的总长度
             */

            int contentLength = urlConnection.getContentLength();

            String downloadFolderName = Environment.getExternalStorageDirectory()
                    + File.separator + "imooc" + File.separator;

            File file = new File(downloadFolderName);
            if (!file.exists()) {
                file.mkdir();
            }

            String fileName = downloadFolderName + "imooc.apk";

            File apkFile = new File(fileName);

            if(apkFile.exists()){
                apkFile.delete();
            }

            int downloadSize = 0;
            byte[] bytes = new byte[1024];

            int length;

            OutputStream outputStream = new FileOutputStream(fileName);
            while ((length = inputStream.read(bytes)) != -1) {
                outputStream.write(bytes, 0, length);
                downloadSize += length;
                /**
                 * update UI
                 */

                Message message = Message.obtain();
                message.obj = downloadSize * 100 / contentLength;
                message.what = DOWNLOAD_MESSAGE_CODE;
                mHandler.sendMessage(message);

            }
            inputStream.close();
            outputStream.close();


        } catch (MalformedURLException e) {
            notifyDownloadFaild();
            e.printStackTrace();
        } catch (IOException e) {
            notifyDownloadFaild();
            e.printStackTrace();
        }
    }

    private void notifyDownloadFaild() {
        Message message = Message.obtain();
        message.what = DOWNLOAD_MESSAGE_FAIL_CODE;
        mHandler.sendMessage(message);
    }
}
````

2.解决handler内存泄漏的问题（使用弱引用）

````java
public static class TestHandler extends Handler{

        public final WeakReference<MainActivity> mWeakReference;
        public TestHandler(MainActivity activity) {
            mWeakReference = new WeakReference<>(activity);
        }

        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            MainActivity activity = mWeakReference.get();
            if(msg.what == CODE){
                int time = msg.arg1;
                activity.mTextview.setText(String.valueOf(time/1000));
                Message message = Message.obtain();
                message.what = CODE;
                message.arg1  = time - 1000;

                if (time > 0) {
                    sendMessageDelayed(message, 1000);
                }
            }

        }
    }
````

### AsyncTask

​	android中被封装过的多线程方法。

#### 使用

````java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        new DownLoadAsyncTask().execute("imooc","ssss");
    }

    public class DownLoadAsyncTask extends AsyncTask<String,Integer,Boolean>{
        @Override
        protected void onPreExecute() {
            super.onPreExecute();
            //(主线程)可操作UI
        }

        @Override
        protected Boolean doInBackground(String... strings) {
            return null;
            //(子线程)在另一个线程中处理事件
        }

        @Override
        protected void onPostExecute(Boolean aBoolean) {
            super.onPostExecute(aBoolean);
            //(主线程)在主线程中 执行处理结果
        }

        @Override
        protected void onProgressUpdate(Integer... values) {
            super.onProgressUpdate(values);
            //(主线程)收到进度 然后处理 也是在UI线程中
        }
    }

}
````

### CardView

常用属性：

。。。

### 屏幕适配

#### dp范围划分

| 名称    | 像素密度范围  |
| ------- | ------------- |
| mdpi    | 120dpi-160dpi |
| hdpi    | 160dpi-240dpi |
| xhdpi   | 240dpi-320dpi |
| xxhdpi  | 320dpi-480dpi |
| xxxhdpi | 480dpi-640dpi |

切图标准：2：3：4：6：8

#### 布局适配

1.禁用绝对布局

2.少用px（用dp、dip）

3.使用wrap_content、match_parent、layout_weight

4.重建布局文件

​	对于垂直方向，或者水平方向的布局。将相应方向的layout_width、layout_height设置为0dp。即可使layout_weight分配的权重生效。即可实现在各种屏幕大小都可按比例适配。

#### 图片适配

1.提供不同分辨率的备用位图

2.使用自动拉伸图

​	使用AS制作.9.png的图片，确定拉伸区域和文本显示区域。

### 数据存储

#### 1.SharedPreferences存储数据

使用：

​	用于存放一些登录信息

概念：

​	本质上是一个xml文件，是通过类似键值对的方式存放信息。

​	位于程序私有目录中，即data/data/[packageName]/shared_prefs

操作模式：

​	MODE_APPEND:追加方式存储（已过期，无法使用）

​	MODE_PRIVATE:私有方式存储，其他应用无法访问（主要使用此模式）

​	MODE_WORLD_READABLE:可被其他应用读取

​	MODE_WORLD_WRITEABLE:可被其他应用写入

````java
public class ShareActivity extends AppCompatActivity {

    private EditText accEdt,pwdEdt;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_share);

        accEdt = findViewById(R.id.acc_edt);
        pwdEdt = findViewById(R.id.pwd_edt);

        //SharePreference的读取
        //①获取SharePreference对象(参数1：文件名  参数2：模式)
        SharedPreferences share = getSharedPreferences("myshare",MODE_PRIVATE);
        //②根据key获取内容(参数1：key   参数2：当对应key不存在时，返回参数2的内容作为默认值)
        String accStr = share.getString("account","");
        String pwdStr = share.getString("pwd","");

        accEdt.setText(accStr);
        pwdEdt.setText(pwdStr);

        findViewById(R.id.login_btn).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                //1.获取两个输入框的内容
                String account = accEdt.getText().toString();
                String pwd = pwdEdt.getText().toString();
                //2.验证(admin  123)
                    if(account.equals("admin") && pwd.equals("123")){
                        //2.1存储信息到SharePreference
                        //①获取SharePreference对象(参数1：文件名  参数2：模式)
                        SharedPreferences share = getSharedPreferences("myshare",MODE_PRIVATE);
                        //②获取Editor对象
                        SharedPreferences.Editor edt = share.edit();
                        //③存储信息
                        edt.putString("account",account);
                        edt.putString("pwd",pwd);
                        //④指定提交操作
                        edt.commit();

                        Toast.makeText(ShareActivity.this,"登录成功",Toast.LENGTH_SHORT).show();
                    }else {
                        //2.2验证失败，提示用户
                        Toast.makeText(ShareActivity.this,"账号或密码错误",Toast.LENGTH_SHORT).show();
                    }
            }
        });
    }
}
````

#### 2.文件存储

##### 外部存储

外部存储ExternalStorage：

1.存储位置：storage或者mnt文件夹

2.获取存储路径：Environment.getExternalStorageDirectory();

3.公有目录常以功能名称命名（DCIM、DOWNLOAD）

4.私有目录（Android/data/应用包名）

获取读写权限（manifest.xml）

````xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
````

````java
public class ExternalActivity extends AppCompatActivity {

    EditText infoEdt;
    TextView txt;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_external);

        infoEdt = findViewById(R.id.info_edt);
        txt = findViewById(R.id.textView);

        int permisson = ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE);
        if(permisson!=PackageManager.PERMISSION_GRANTED){
            //动态去申请权限
            ActivityCompat.requestPermissions(this,new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE},1);
        }

    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if(requestCode == 1){
            //xxxxxxxxxxxxx
        }
    }

    public void operate(View v){
        String path = Environment.getExternalStorageDirectory().getAbsolutePath() + "/Download/imooc.txt";
        Log.e("TAG",path);
        //if(Environment.getExternalStorageState().equals("mounted"))查看是否是存储在sd卡中
        switch (v.getId()){
            case R.id.save_btn:
                File f = new File(path);
                try {
                    if (!f.exists()) {
                        f.createNewFile();
                    }

                    FileOutputStream fos = new FileOutputStream(path,true);
                    String str = infoEdt.getText().toString();
                    fos.write(str.getBytes());
                }catch (IOException ioe){
                    ioe.printStackTrace();
                }
                break;
            case R.id.read_btn:
                try {
                    FileInputStream fis = new FileInputStream(path);
                    byte[] b = new byte[1024];
                    int len = fis.read(b);
                    String str2 = new String(b,0,len);
                    txt.setText(str2);
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                break;
        }
    }
}

````

如果想让你的应用在被卸载时也能一并被删除，就存放在以下两个目录中。

````java
Context.getExternalFilesDir();
//获取到SDCard/Android/data/包名/files/目录
Context.getExternalCacheDir();
//获取到SDCard/Android/data/包名/cache/目录
````

外部存储注意确认真机是否有sd卡：

````java
if(Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED))
````

##### 内部存储

内部存储InternalStorage：

通过DDMS->File Explorer可以找到，文件夹叫data

文件夹中有两个文件：app，data

获取内部存储的目录：

````java
Context.getFilesDir();
//获取/data/data/包名/files
Context.getCacheDir();
//获取到data/data/包名/cache
````

````java
public class InternalActivity extends AppCompatActivity {
    EditText edt;
    TextView txt;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_internal);
        
        edt = findViewById(R.id.editText);
        txt = findViewById(R.id.textView);
    }

    public void operate(View v){

        //  data/data/包名/files
        //   getCacheDir()    data/data/包名/cache
        File f = new File(getFilesDir(),"getFilesDirs.txt");
        switch (v.getId()){
            case R.id.save_btn:
                try {
                    if (!f.exists()) {
                        f.createNewFile();
                    }
                    FileOutputStream fos = new FileOutputStream(f);
                    fos.write(edt.getText().toString().getBytes());
                    fos.close();
                }catch (IOException e){
                    e.printStackTrace();
                }
                break;
            case R.id.read_btn:
                try {
                    FileInputStream fis = new FileInputStream(f);
                    byte[] b = new byte[1024];
                    int len = fis.read(b);
                    String str2 = new String(b,0,len);
                    txt.setText(str2);
                } catch (FileNotFoundException e) {
                    e.printStackTrace();
                } catch (IOException e) {
                    e.printStackTrace();
                }
                break;
        }
    }
}

````

##### 对比

![内部-外部存储对比](https://github.com/g453030291/java-2/blob/master/images/内部-外部存储对比.png)

#### 3.SQLite数据库

​	SQLiteOpenHelper：android平台提供的数据库辅助类，用于创建和打开数据库。

​	SQLiteDataBase：用于管理和操作SQLite数据库，所有数据库操作最终都将由该类完成。

操作数据库主要有两种操作方法：

1.rawQuery()查询。execSQL()增删改和创建表。

````java
public class MainActivity extends Activity {

    private EditText nameEdt , ageEdt , idEdt;
    private RadioGroup genderGp;
    private ListView stuList;
    private RadioButton malerb;
    private String genderStr = "男";
    private SQLiteDatabase db;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //添加操作
        //数据库名称
        //如果只有一个数据库名称，那么这个数据库的位置会是在私有目录中
        //如果带SD卡路径，那么数据库位置则在指定的路径下
        String path = Environment.getExternalStorageDirectory() + "/stu.db";
        SQLiteOpenHelper helper = new SQLiteOpenHelper(this,path,null,2) {
            @Override
            public void onCreate(SQLiteDatabase sqLiteDatabase) {
                //创建
                Toast.makeText(MainActivity.this,"数据库创建",Toast.LENGTH_SHORT).show();
                //如果数据库不存在，则会调用onCreate方法，那么我们可以将表的创建工作放在这里面完成
                        /*
                        String sql = "create table test_tb (_id integer primary key autoincrement," +
                                "name varhcar(20)," +
                                "age integer)";
                        sqLiteDatabase.execSQL(sql);*/
            }

            @Override
            public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {
                //升级
                Toast.makeText(MainActivity.this,"数据库升级",Toast.LENGTH_SHORT).show();
            }
        };

        //用于获取数据库库对象
        //1.数据库存在，则直接打开数据库
        //2.数据库不存在，则调用创建数据库的方法，再打开数据库
        //3.数据库存在，但版本号升高了，则调用数据库升级方法
        db = helper.getReadableDatabase();

        nameEdt = (EditText) findViewById(R.id.name_edt);
        ageEdt = (EditText) findViewById(R.id.age_edt);
        idEdt = (EditText) findViewById(R.id.id_edt);
        malerb = (RadioButton) findViewById(R.id.male);

        genderGp = (RadioGroup) findViewById(R.id.gender_gp);
        genderGp.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup radioGroup, int i) {
                if(i == R.id.male){
                    //“男”
                    genderStr = "男";
                }else{
                    //"女"
                    genderStr = "女";
                }
            }
        });

        stuList = (ListView) findViewById(R.id.stu_list);
    }

//    SQLiteOpenHelper
//    SQLiteDatabase
    public void operate(View v){

        String nameStr = nameEdt.getText().toString();
        String ageStr = ageEdt.getText().toString();
        String idStr = idEdt.getText().toString();
        switch (v.getId()){
            case R.id.insert_btn:


//                db.rawQuery();        查询    select * from 表名
//                db.execSQL();         添加、删除、修改、创建表

                //String sql = "insert into info_tb (name,age,gender) values ('"+nameStr+"',"+ageStr+",'"+genderStr+"')";
                String sql = "insert into info_tb (name,age,gender) values (?,?,?)";
                db.execSQL(sql,new String[]{nameStr,ageStr,genderStr});
                Toast.makeText(this,"添加成功",Toast.LENGTH_SHORT).show();
                break;
            case R.id.select_btn:
                //SQLiteOpenHelper
                //select * from 表名 where _id = ?
                String sql2 = "select * from info_tb";
                if(!idStr.equals("")){
                    sql2 += " where _id=" + idStr;
                }
                //查询结果
                Cursor c = db.rawQuery(sql2,null);

                //SimpleCursorAdapter
                //SimpleAdapter a = new SimpleAdapter()
                //参数3：数据源
                SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                        this, R.layout.item,c,
                        new String[]{"_id","name","age","gender"},
                        new int[]{R.id.id_item,R.id.name_item,R.id.age_item,R.id.gender_item});
                stuList.setAdapter(adapter);
                break;
            case R.id.delete_btn:
                //' '   "23"   23
                String sql3 = "delete from info_tb where _id=?";
                db.execSQL(sql3,new String[]{idStr});
                Toast.makeText(this,"删除成功",Toast.LENGTH_SHORT).show();
                break;
            case R.id.update_btn:
                String sql4 = "update info_tb set name=? , age=? , gender=?  where _id=?";
                db.execSQL(sql4,new String[]{nameStr,ageStr,genderStr,idStr});
                Toast.makeText(this,"修改成功",Toast.LENGTH_SHORT).show();
                break;
        }
        nameEdt.setText("");
        ageEdt.setText("");
        idEdt.setText("");
        malerb.setChecked(true);
    }
}
````

2.直接操作insert，select，update，delete方法。

````java
public class MainActivity2 extends Activity {

    private EditText nameEdt , ageEdt , idEdt;
    private RadioGroup genderGp;
    private ListView stuList;
    private RadioButton malerb;
    private String genderStr = "男";
    private SQLiteDatabase db;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //添加操作
        //数据库名称
        //如果只有一个数据库名称，那么这个数据库的位置会是在私有目录中
        //如果带SD卡路径，那么数据库位置则在指定的路径下
        String path = Environment.getExternalStorageDirectory() + "/stu.db";
        SQLiteOpenHelper helper = new SQLiteOpenHelper(this,path,null,2) {
            @Override
            public void onCreate(SQLiteDatabase sqLiteDatabase) {
                //创建
                Toast.makeText(MainActivity2.this,"数据库创建",Toast.LENGTH_SHORT).show();
                //如果数据库不存在，则会调用onCreate方法，那么我们可以将表的创建工作放在这里面完成
                        /*
                        String sql = "create table test_tb (_id integer primary key autoincrement," +
                                "name varhcar(20)," +
                                "age integer)";
                        sqLiteDatabase.execSQL(sql);*/
            }

            @Override
            public void onUpgrade(SQLiteDatabase sqLiteDatabase, int i, int i1) {
                //升级
                Toast.makeText(MainActivity2.this,"数据库升级",Toast.LENGTH_SHORT).show();
            }
        };

        //用于获取数据库库对象
        //1.数据库存在，则直接打开数据库
        //2.数据库不存在，则调用创建数据库的方法，再打开数据库
        //3.数据库存在，但版本号升高了，则调用数据库升级方法
        db = helper.getReadableDatabase();

        nameEdt = (EditText) findViewById(R.id.name_edt);
        ageEdt = (EditText) findViewById(R.id.age_edt);
        idEdt = (EditText) findViewById(R.id.id_edt);
        malerb = (RadioButton) findViewById(R.id.male);

        genderGp = (RadioGroup) findViewById(R.id.gender_gp);
        genderGp.setOnCheckedChangeListener(new RadioGroup.OnCheckedChangeListener() {
            @Override
            public void onCheckedChanged(RadioGroup radioGroup, int i) {
                if(i == R.id.male){
                    //“男”
                    genderStr = "男";
                }else{
                    //"女"
                    genderStr = "女";
                }
            }
        });

        stuList = (ListView) findViewById(R.id.stu_list);
    }

//    SQLiteOpenHelper
//    SQLiteDatabase
    public void operate(View v){

        String nameStr = nameEdt.getText().toString();
        String ageStr = ageEdt.getText().toString();
        String idStr = idEdt.getText().toString();
        switch (v.getId()){
            case R.id.insert_btn:
                //在SqliteDatabase类下，提供四个方法
                //insert（添加）、delete（删除）、update（修改）、query（查询）
                //都不需要写sql语句
                //参数1：你所要操作的数据库表的名称
                //参数2：可以为空的列.  如果第三个参数是null或者说里面没有数据
                //那么我们的sql语句就会变为insert into info_tb () values ()  ，在语法上就是错误的
                //此时通过参数3指定一个可以为空的列，语句就变成了insert into info_tb (可空列) values （null）
                ContentValues values = new ContentValues();
                //insert into 表明(列1，列2) values（值1，值2）
                values.put("name",nameStr);
                values.put("age",ageStr);
                values.put("gender",genderStr);
                long id = db.insert("info_tb",null,values);
                Toast.makeText(this,"添加成功，新学员学号是：" + id,Toast.LENGTH_SHORT).show();
                break;
            case R.id.select_btn:
                //select 列名 from 表名 where 列1 = 值1 and 列2 = 值2
                //参数2：你所要查询的列。{”name","age","gender"},查询所有传入null/{“*”}
                //参数3：条件（针对列）
                //参数5:分组
                //参数6：当 group by对数据进行分组后，可以通过having来去除不符合条件的组
                //参数7:排序
                Cursor c = db.query("info_tb",null,null,null,null,null,null);

                //SimpleCursorAdapter
                //SimpleAdapter a = new SimpleAdapter()
                //参数3：数据源
                SimpleCursorAdapter adapter = new SimpleCursorAdapter(
                        this, R.layout.item,c,
                        new String[]{"_id","name","age","gender"},
                        new int[]{R.id.id_item,R.id.name_item,R.id.age_item,R.id.gender_item});
                stuList.setAdapter(adapter);
                break;
            case R.id.delete_btn:
                int count = db.delete("info_tb","_id=?",new String[]{idStr});
                if(count > 0) {
                    Toast.makeText(this, "删除成功", Toast.LENGTH_SHORT).show();
                }
                break;
            case R.id.update_btn:
                ContentValues values2 = new ContentValues();
                //update info_tb set 列1=xx , 列2=xxx where 列名 = 值
                values2.put("name",nameStr);
                values2.put("age",ageStr);
                values2.put("gender",genderStr);
                int count2 = db.update("info_tb",values2,"_id=?",new String[]{idStr});
                if(count2 > 0) {
                    Toast.makeText(this, "修改成功", Toast.LENGTH_SHORT).show();
                }
                break;
        }
        nameEdt.setText("");
        ageEdt.setText("");
        idEdt.setText("");
        malerb.setChecked(true);
    }
}
````

#### 4.ContentProvider存储数据（不同应用之间数据共享）

#### 5.服务端存储

### 广播

Boradcast和BroadcastReceiver

#### 静态注册：

AndroidManifest.xml

8.0后静态注册的限制

Manifest.xml

````xml
<!-- 静态注册广播接收器 -->
        <receiver
            android:name=".ImoocBroadcastReceiver">

            <!-- 接收哪些广播 -->

            <intent-filter>
                <!-- 开机广播 -->
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
                <!-- 电量低广播 -->
                <action android:name="android.intent.action.BATTERY_LOW"/>
                <!-- 应用被安装广播 -->
                <action android:name="android.intent.action.PACKAGE_INSTALL"/>
                <!-- 应用被卸载广播 -->
                <action android:name="android.intent.action.PACKAGE_REMOVED"/>
                <!-- 数据类型 -->
                <data android:scheme="package"/>

                <!-- 自定义广播 -->
                <action android:name="com.imooc.demo.afdsabfdaslj"/>

            </intent-filter>

        </receiver>
````

````java
public class ImoocBroadcastReceiver extends BroadcastReceiver {

    TextView mTextView;
    public ImoocBroadcastReceiver() {
    }

    public ImoocBroadcastReceiver(TextView textView) {
        mTextView = textView;
    }

    private static final String TAG = "ImoocBroadcastReceiver";
    @Override
    public void onReceive(Context context, Intent intent) {
        // 接收广播
        if(intent != null){
            // 接收到的是什么广播
            String action  = intent.getAction();
            Log.d(TAG, "onReceive: " + action);

            // 判断是什么广播（是不是我们自己发送的自定义广播）
            if(TextUtils.equals(action, MainActivity.MY_ACTION)){
                // 获取广播携带的内容， 可自定义的数据
                String content = intent.getStringExtra(MainActivity.BROADCAST_CONTENT);
                if(mTextView != null){
                    mTextView.setText("接收到的action是："+ action + "\n接收到的内容是：\n" + content);
                }
            }
        }
    }
}
````

#### 动态注册：

````java
public class MainActivity extends AppCompatActivity {

    public static final String MY_ACTION = "com.imooc.demo.afdsabfdaslj";
    public static final String BROADCAST_CONTENT = "broadcast_content";
    private ImoocBroadcastReceiver mBroadcastReceiver;
    private EditText mInputEditText;
    private Button mSendBroadcastButton;
    private TextView mResultTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 用包名做title
        setTitle(getPackageName());

        mInputEditText = findViewById(R.id.inputEditText);
        mSendBroadcastButton = findViewById(R.id.sendBroadcastButton);
        mResultTextView = findViewById(R.id.resultTextView);



        // 新建广播接收器
        mBroadcastReceiver = new ImoocBroadcastReceiver(mResultTextView);

        // 注册广播接收器

        // 为广播接收器添加Action
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction("android.intent.action.PACKAGE_REMOVED");
        intentFilter.addAction(MY_ACTION);

        // 注册广播接收器
        registerReceiver(mBroadcastReceiver, intentFilter);


        mSendBroadcastButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 新建广播
                Intent intent = new Intent(MY_ACTION);
                // 放入广播要携带的数据
                intent.putExtra(BROADCAST_CONTENT, mInputEditText.getText().toString());
                sendBroadcast(intent);
            }
        });

    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        // 取消注册广播接收器，不然会导致内存泄露
        if(mBroadcastReceiver != null){
            unregisterReceiver(mBroadcastReceiver);
        }
    }
}
````

#### 自定义广播：

。。。

### Application

​	Application类是一个维护应用全局状态的基类。Android系统在启动时会创建一个Application对象。

​	我们可以扩展Application类，让Android系统使用我们自定义的Application类来创建Application对象。1.创建Application子类。2.manifest文件中为application标签添加name属性。

回调函数：

onCreate、onConfigurationChanged、onLowMemory

作用：

共享全局状态（在不同activity之间）

初始化全应用所需的服务

#### otto

尽量使用单例的otto