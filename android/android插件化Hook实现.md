#### Android 插件化 Hook 实现
Hook的方式需要基本了解系统启动一个Activity的过程，一般来说系统会先检查Activity是否注册，然后再去生成该Activity，那么我们只需要在检查的时候用一个已经注册的Activity（桩，通常表示为StubActivity）来给系统检查，检查通过后在生成的时候再替换成插件的就可以了。

##### 一、hook Instrumentation 
主要工作：要自己实现一个Instrumentation，hook execStartActivity 和 newActivity，修改 intent 绕过 activity 检查，再复原 intent 创建 activity.

```java
public class HookedInstrumentation extends Instrumentation implements Handler.Callback {
    public static final String TAG = "HookedInstrumentation";
    protected Instrumentation mBase;
    private PluginManager mPluginManager;

    public HookedInstrumentation(Instrumentation base, PluginManager pluginManager) {
        mBase = base;
        mPluginManager = pluginManager;
    }

    /**
     * 覆盖掉原始Instrumentation类的对应方法,用于插件内部跳转Activity时适配
     *
     * @Override
     */
    public ActivityResult execStartActivity(
            Context who, IBinder contextThread, IBinder token, Activity target,
            Intent intent, int requestCode, Bundle options) {

        if (Constants.DEBUG) Log.e(TAG, "execStartActivity");
        mPluginManager.hookToStubActivity(intent);

        try {
            Method execStartActivity = Instrumentation.class.getDeclaredMethod(
                    "execStartActivity", Context.class, IBinder.class, IBinder.class,
                    Activity.class, Intent.class, int.class, Bundle.class);
            execStartActivity.setAccessible(true);
            return (ActivityResult) execStartActivity.invoke(mBase, who,
                    contextThread, token, target, intent, requestCode, options);
        } catch (Exception e) {
            e.printStackTrace();
            throw new RuntimeException("do not support!!!" + e.getMessage());
        }
    }

    @Override
    public Activity newActivity(ClassLoader cl, String className, Intent intent) throws InstantiationException, IllegalAccessException, ClassNotFoundException {
        if (Constants.DEBUG) Log.e(TAG, "newActivity");

        if (mPluginManager.hookToPluginActivity(intent)) {
            String targetClassName = intent.getComponent().getClassName();
            PluginApp pluginApp = mPluginManager.getLoadedPluginApk();
            Activity activity = mBase.newActivity(pluginApp.mClassLoader, targetClassName, intent);
            activity.setIntent(intent);
            ReflectUtil.setField(ContextThemeWrapper.class, activity, Constants.FIELD_RESOURCES, pluginApp.mResources);
            return activity;
        }

        if (Constants.DEBUG) Log.e(TAG, "super.newActivity(...)");
        return super.newActivity(cl, className, intent);
    }

    @Override
    public boolean handleMessage(Message message) {
        if (Constants.DEBUG) Log.e(TAG, "handleMessage");
        return false;
    }


    @Override
    public void callActivityOnCreate(Activity activity, Bundle icicle) {
        if (Constants.DEBUG) Log.e(TAG, "callActivityOnCreate");
        super.callActivityOnCreate(activity, icicle);
    }
}
在负责启动的execStartActivity设置为启动已注册的Activity，再在newActivity设置为实际要启动的插件的Activity。然后去Hook系统持有的该字段：

public class ReflectUtil {
    public static final String METHOD_currentActivityThread = "currentActivityThread";
    public static final String CLASS_ActivityThread = "android.app.ActivityThread";
    public static final String FIELD_mInstrumentation = "mInstrumentation";
    public static final String TAG = "ReflectUtil";


    private static Instrumentation sInstrumentation;
    private static Instrumentation sActivityInstrumentation;
    private static Field sActivityThreadInstrumentationField;
    private static Field sActivityInstrumentationField;
    private static Object sActivityThread;

    public static boolean init() {
        //获取当前的ActivityThread对象
        Class<?> activityThreadClass = null;
        try {
            activityThreadClass = Class.forName(CLASS_ActivityThread);
            Method currentActivityThreadMethod = activityThreadClass.getDeclaredMethod(METHOD_currentActivityThread);
            currentActivityThreadMethod.setAccessible(true);
            Object currentActivityThread = currentActivityThreadMethod.invoke(null);


            //拿到在ActivityThread类里面的原始mInstrumentation对象
            Field instrumentationField = activityThreadClass.getDeclaredField(FIELD_mInstrumentation);
            instrumentationField.setAccessible(true);
            sActivityThreadInstrumentationField = instrumentationField;

            sInstrumentation = (Instrumentation) instrumentationField.get(currentActivityThread);
            sActivityThread = currentActivityThread;


            sActivityInstrumentationField =  Activity.class.getDeclaredField(FIELD_mInstrumentation);
            sActivityInstrumentationField.setAccessible(true);
            return true;
        } catch (ClassNotFoundException
                | NoSuchMethodException
                | IllegalAccessException
                | InvocationTargetException
                | NoSuchFieldException e) {
            e.printStackTrace();
        }

        return false;
    }

    public static Instrumentation getInstrumentation() {
        return sInstrumentation;
    }

    public static Object getActivityThread() {
        return sActivityThread;
    }

    public static void setInstrumentation(Object activityThread, HookedInstrumentation hookedInstrumentation) {
        try {
            sActivityThreadInstrumentationField.set(activityThread, hookedInstrumentation);
            if (Constants.DEBUG) Log.e(TAG, "set hooked instrumentation");
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public static void setActivityInstrumentation(Activity activity, PluginManager manager) {
        try {
            sActivityInstrumentation = (Instrumentation) sActivityInstrumentationField.get(activity);
            HookedInstrumentation instrumentation = new HookedInstrumentation(sActivityInstrumentation, manager);
            sActivityInstrumentationField.set(activity, instrumentation);
            if (Constants.DEBUG) Log.e(TAG, "set activity hooked instrumentation");
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
    }


    public static void setField(Class clazz, Object target, String field, Object object) {
        try {
            Field f = clazz.getDeclaredField(field);
            f.setAccessible(true);
            f.set(target, object);
        } catch (Exception e) {
            e.printStackTrace();
        }

    }

}
```

##### 二、实现 PluginManager
PluginManager同样负责加载插件的类和资源等。
```java
public class PluginManager {
    private final static String TAG = "PluginManager";
    private static PluginManager sInstance;
    private Context mContext;
    private PluginApp mPluginApp;


    public static PluginManager getInstance(Context context) {
        if (sInstance == null && context != null) {
            sInstance = new PluginManager(context);
        }
        return sInstance;
    }

    private PluginManager(Context context) {
        mContext = context;
    }


    public void hookInstrumentation() {
        try {
            Instrumentation baseInstrumentation = ReflectUtil.getInstrumentation();

            final HookedInstrumentation instrumentation = new HookedInstrumentation(baseInstrumentation, this);

            Object activityThread = ReflectUtil.getActivityThread();
            ReflectUtil.setInstrumentation(activityThread, instrumentation);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public void hookCurrentActivityInstrumentation(Activity activity) {
        ReflectUtil.setActivityInstrumentation(activity, sInstance);
    }


    public void hookToStubActivity(Intent intent) {
        if (Constants.DEBUG) Log.e(TAG, "hookToStubActivity");

        if (intent == null || intent.getComponent() == null) {
            return;
        }
        String targetPackageName = intent.getComponent().getPackageName();
        String targetClassName = intent.getComponent().getClassName();

        if (mContext != null
                && !mContext.getPackageName().equals(targetPackageName)
                && isPluginLoaded(targetPackageName)) {
            if (Constants.DEBUG) Log.e(TAG, "hook " +  targetClassName + " to " + Constants.STUB_ACTIVITY);

            intent.setClassName(Constants.STUB_PACKAGE, Constants.STUB_ACTIVITY);
            intent.putExtra(Constants.KEY_IS_PLUGIN, true);
            intent.putExtra(Constants.KEY_PACKAGE, targetPackageName);
            intent.putExtra(Constants.KEY_ACTIVITY, targetClassName);
        }
    }

    public boolean hookToPluginActivity(Intent intent) {
        if (Constants.DEBUG) Log.e(TAG, "hookToPluginActivity");
        if (intent.getBooleanExtra(Constants.KEY_IS_PLUGIN, false)) {
            String pkg = intent.getStringExtra(Constants.KEY_PACKAGE);
            String activity = intent.getStringExtra(Constants.KEY_ACTIVITY);
            if (Constants.DEBUG) Log.e(TAG, "hook " + intent.getComponent().getClassName() + " to " + activity);
            intent.setClassName(pkg, activity);
            return true;
        }
        return false;
    }

    private boolean isPluginLoaded(String packageName) {
        // TODO 检查packageNmae是否匹配
        return mPluginApp != null;
    }



    public PluginApp loadPluginApk(String apkPath) {
        String addAssetPathMethod = "addAssetPath";
        PluginApp pluginApp = null;
        try {
            AssetManager assetManager = AssetManager.class.newInstance();
            Method addAssetPath = assetManager.getClass().getMethod(addAssetPathMethod, String.class);
            addAssetPath.invoke(assetManager, apkPath);
            Resources pluginRes = new Resources(assetManager,
                    mContext.getResources().getDisplayMetrics(),
                    mContext.getResources().getConfiguration());
            pluginApp = new PluginApp(pluginRes);
            pluginApp.mClassLoader = createDexClassLoader(apkPath);
        } catch (IllegalAccessException
                | InstantiationException
                | NoSuchMethodException
                | InvocationTargetException e) {
            e.printStackTrace();
        }
        return pluginApp;
    }

    private DexClassLoader createDexClassLoader(String apkPath) {
        File dexOutputDir = mContext.getDir("dex", Context.MODE_PRIVATE);
        return new DexClassLoader(apkPath, dexOutputDir.getAbsolutePath(),
                null, mContext.getClassLoader());

    }

    public boolean loadPlugin(String apkPath) {
        File apk = new File(apkPath);
        if (!apk.exists()) {
            return false;
        }
        mPluginApp = loadPluginApk(apkPath);
        return mPluginApp != null;
    }

    public PluginApp getLoadedPluginApk() {
        return mPluginApp;
    }
}
```

##### 三、使用
宿主 app：
```java
public class MainActivity extends Activity implements View.OnClickListener {
    // https://zhuanlan.zhihu.com/p/33017826

    public static final boolean DEBUG = true;
    public static final String TAG = "MainActivity";

    private String mPluginPackageName = "top.vimerzhao.image";
    private String mPluginClassName = "top.vimerzhao.image.MainActivity";

    //读写权限
    private static String[] PERMISSIONS_STORAGE = {Manifest.permission.READ_EXTERNAL_STORAGE,
            Manifest.permission.WRITE_EXTERNAL_STORAGE};
    //请求状态码
    private static int REQUEST_PERMISSION_CODE = 1;

    private PluginManager mPluginManager;


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initData();
        initView();
        initPlugin();
    }


    private void initPlugin() {
        // !! must first
        ReflectUtil.init();

        mPluginManager = PluginManager.getInstance(getApplicationContext());
        mPluginManager.hookInstrumentation();
        mPluginManager.hookCurrentActivityInstrumentation(this);
    }

    private void initData() {
        if (Build.VERSION.SDK_INT > Build.VERSION_CODES.LOLLIPOP) {
            if (ActivityCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                ActivityCompat.requestPermissions(this, PERMISSIONS_STORAGE, REQUEST_PERMISSION_CODE);
            }
        }
    }

    private void initView() {
        (findViewById(R.id.tv_launch)).setOnClickListener(this);
    }

    @Override
    protected void attachBaseContext(Context newBase) {
        super.attachBaseContext(newBase);
        // !!! 不要在此Hook,看源码发现mInstrumentaion会在此方法后初始化
    }

    @Override
    public void onClick(View view) {
        if (Constants.DEBUG) Log.e(TAG, "click view id: " + view.getId());
        if (view.getId() == R.id.tv_launch) {
            // TODO launch plugin app
            if (mPluginManager.loadPlugin(Constants.PLUGIN_PATH)) {
                Intent intent = new Intent();
                intent.setClassName(mPluginPackageName, mPluginClassName);
                startActivity(intent);
            }
        }

    }
}
```

插件 app：
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

    }

    @Override
    protected void onResume() {
        super.onResume();

    }
}
```

#### 参考
https://zhuanlan.zhihu.com/p/33017826

