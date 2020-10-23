### 图片为证
<hr/>

<table>
    <tr>
       <td><center>
        <img src="https://img-blog.csdnimg.cn/20201023112914358.gif" width="95%"/>
        </center>
        <center>
        图1 程序自动拉活
        </center></td> 
        <td><center>
        <img src="https://img-blog.csdnimg.cn/20201023105610399.gif" />
        </center>
        <center>
        图2 打不开的进程页面
        </center></td> 
    </tr>
</table>



### 保活思路
<hr/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;传统的套路咱就不再累赘（[详情可看这里](https://blog.csdn.net/qq_35350654/article/details/84062395)），这里分享一个流氓做法，具体看下面代码。

```java
	private static android.os.Handler handler = new android.os.Handler(Looper.getMainLooper());
	/**
     * 返回主页
     *
     * @param context
     * @param delay
     */
    public static void perforGlobalHome(final Context context, long delay) {
        Intent home = new Intent(Intent.ACTION_MAIN);
        home.addCategory(Intent.CATEGORY_HOME);
        home.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        context.startActivity(home);

        handler.postDelayed(new Runnable() {
            @Override
            public void run() {
                Intent home = new Intent(Intent.ACTION_MAIN);
                home.addCategory(Intent.CATEGORY_HOME);
                home.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                context.startActivity(home);
            }
        }, delay);
    }
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很多大佬看到这里可能就不耐烦了，这不就是通过代码拉回主页嘛，跟保活有毛关系。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不着急，再配上一个广播监听基本就实现杀不死了，具体看代码。

```java
public class BroadcastReceiver extends android.content.BroadcastReceiver {
    private final String TAG = BroadcastReceiver.class.getSimpleName();

    @Override
    public void onReceive(final Context context, Intent intent) {
        // TODO Auto-generated method stub
        Log.e(TAG, "接收到广播：" + intent.getAction());
        
        if (intent.getAction() == Intent.ACTION_CLOSE_SYSTEM_DIALOGS) {
            perforGlobalHome(context, 500);
        }
     }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color=#FF0000>到这里就很多大佬应该能看出来了，实际上就是监听ACTION_CLOSE_SYSTEM_DIALOGS，当触发时直接通过代码拉回桌面，这样用户打不开进程页面自然就杀不死程序进程了。</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;哦，别忘了注册广播，详情看下面代码。

```java
 IntentFilter filter = new IntentFilter();
 filter.addAction(Intent.ACTION_CLOSE_SYSTEM_DIALOGS);//home
 registerReceiver(new BroadcastReceiver(), filter);
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;简单而又粗暴，无需任何权限即可实现应用保活，妈妈再也不用担心我Android O（8）到Android Q（10）无法实现应用保活的问题了。

### 拉活权限
<hr/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然了，高标准需求的小伙伴都应该知道，接下来才是正菜，仅仅防止用户手动杀掉进程还是远远不够的，我们并不能排除应用被系统干掉的可能性。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;那么我们又是如何来解决这个问题呢？详情请看下面具体代码。

```java
public class MyAccessibilityService extends AccessibilityService {
    private String TAG = MyAccessibilityService.class.getSimpleName();
    protected void onServiceConnected() {
        super.onServiceConnected();       
    }

    @Override
    protected boolean onKeyEvent(KeyEvent event) {
        return false;
    }

    @Override
    public void onAccessibilityEvent(AccessibilityEvent event) {
    	//在这里重新拉活自己的程序即可
    }
    
	@Override
    public void onInterrupt() {
    }

    @Override
    public boolean onUnbind(Intent intent) {
        return super.onUnbind(intent);
    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不要忘了在AndroidManifest中注册服务完成配置具体看下面代码。

```java
		<service
            android:name=".servise.MyAccessibilityService"
            android:label="@string/app_name"
            android:permission="android.permission.BIND_ACCESSIBILITY_SERVICE">
            <intent-filter>
                <action android:name="android.accessibilityservice.AccessibilityService" />
            </intent-filter>
            <meta-data
                android:name="android.accessibilityservice"
                android:resource="@xml/accessibility_service_config" />
        </service>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然了，还有xml下的配置文件，具体看代码。

```java
	<accessibility-service xmlns:android="http://schemas.android.com/apk/res/android"
    android:accessibilityEventTypes="typeWindowStateChanged"
    android:accessibilityFeedbackType="feedbackGeneric"
    android:accessibilityFlags="flagIncludeNotImportantViews"/>
```

### 完整代码
<hr/>

gitee:[https://gitee.com/yunianvh/employee-management-android](https://gitee.com/yunianvh/employee-management-android)
github:[https://github.com/Life1412378121/Android-2020-APP-](https://github.com/Life1412378121/Android-2020-APP-)
CSDN:https://blog.csdn.net/qq_35350654/article/details/109235852
