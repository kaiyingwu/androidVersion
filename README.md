# androidVersion
记录安卓各个版本的功能和配置方面的变化

# 通知类方面的变化 （Notification）

## 安卓8.0开始加入了ChannelId，即通知的类别，通过创建的通知类别，让用户能在设置中自定义通知的方式，而且将应用的各个通知都罗列出来

通知的创建方式
```
//安卓8.0之前可以这样设定，并直接使用可以，直接实现通知效果
 notification = new NotificationCompat.Builder(context,initChannelId())
                .setSmallIcon(R.mipmap.ic_launcher)     //设置通知图标。
                .setVisibility(NotificationCompat.VISIBILITY_PUBLIC)
                .setContentIntent(pi)            //设置通知点击事件
                .setTicker(taskBean.getMessage())        //通知时在状态栏显示的通知内容
                .setContentInfo("时间提醒")        //内容信息
                .setContentTitle(taskBean.getMessage())        //设置通知标题。
                .setContentText(taskBean.getRemarks())        //设置通知内容。
                .setAutoCancel(false)                //点击通知后通知消失
               // .setDefaults(Notification.DEFAULT_ALL)        //设置系统默认的通知音乐、振动、LED等。
                .setLights(Color.RED,1500,1000)   //灯光
                .setVibrate(new long[]{0, 1000, 500, 1000})  //震动
                .setPriority(NotificationManager.IMPORTANCE_HIGH)  //优先级
                //  .setFullScreenIntent(pi, true)
                .addAction(0,"关闭",getPendingIntent(context))   //添加一个关闭按钮
                .setPublicVersion(notification)  
                .setSound(Uri.parse("android.resource://" + getPackageName() + "/" +R.raw.default_tones))//自定义的通知音频文件
                //  .setTimeoutAfter(3000)  //一段时间后自动关闭通知
                .build();

//8.0之后需要设置ChannelId部分配置设置是一致的

 /**
     * 创建Notification ChannelID
     *
     * @return 频道id
     */
    private String initChannelId() {
        // 通知渠道的id
        String id = "LazyRecord";
        // 用户可以看到的通知渠道的名字.
        CharSequence name = "任务通知";
        // 用户可以看到的通知渠道的描述
        String description = LazyRecordApplication.getInstance().getResources().getString(R.string.play_description);
        if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.O) {
            int importance = NotificationManager.IMPORTANCE_HIGH;
            NotificationChannel mChannel;
            mChannel = new NotificationChannel(id, name, importance);
            mChannel.setDescription(description);
            //LED灯
            mChannel.enableLights(true);
            mChannel.setLightColor(Color.RED);
            //震动
            mChannel.enableVibration(true);
            mChannel.setVibrationPattern(new long[]{0, 1000, 500, 1000});
            mChannel.setImportance(importance);
            mChannel.setSound(Uri.parse("android.resource://" + getPackageName() + "/" +R.raw.default_tones),Notification.AUDIO_ATTRIBUTES_DEFAULT);
            //最后在notificationmanager中创建该通知渠道
            manager.createNotificationChannel(mChannel);
        }
        return id;
    }
    
    //这里有个设置自定义的通知音频文件注意点
    1.设定setSound时，需要取消setDefaults，若都存在的话系统会优先使用系统默认的，即setDefaults的设置
    2.安卓8.0 之后需要在ChannelId中设置setSound
    3.由于ChannelId是在第一次创建就确定了，所以后期如果需要更换某个配置，只能用户自己去配置，除非进行删除重建（id设定不能一样），是否能够更新不确定
    
```

# 系统闹钟方面的设定变化

## 安卓API小于19的情况

```

//基本的set方式可以直接生效
 am.set(AlarmManager.RTC_WAKEUP,timeSmallMillions,pi);
 
 //几个参数的使用案例
AlarmManager.ELAPSED_REALTIME表示闹钟在手机睡眠状态下不可用，该状态下闹钟使用相对时间（相对于系统启动开始），状态值为3；

AlarmManager.ELAPSED_REALTIME_WAKEUP表示闹钟在睡眠状态下会唤醒系统并执行提示功能，该状态下闹钟也使用相对时间，状态值为2；

AlarmManager.RTC表示闹钟在睡眠状态下不可用，该状态下闹钟使用绝对时间，即当前系统时间，状态值为1；

AlarmManager.RTC_WAKEUP表示闹钟在睡眠状态下会唤醒系统并执行提示功能，该状态下闹钟使用绝对时间，状态值为0；

AlarmManager.POWER_OFF_WAKEUP表示闹钟在手机关机状态下也能正常进行提示功能，所以是5个状态中用的最多的状态之一，该状态下闹钟也是用绝对时间，状态值为4；不过本状态好像受SDK版本影响，某些版本并不支持；

```

## 安卓API大于19的情况，由于安卓6.0之后，系统锁屏由于进入Doze模式，闹铃会延后提醒，并且随时间推迟，延迟会越来越久

```

//这时候就不能用set方法设置闹铃了，需要用setExact
am.setExact(AlarmManager.RTC_WAKEUP,timeSmallMillions,pi);

```

## 安卓API大于23的情况，由于部分厂商的锁屏情况会完全杀死系统闹钟的设定，这个时候上面两个方法已经完全不行，需要setAlarmClock方法设置闹钟

```

//需要用setAlarmClock的方式设置闹钟（但是通知栏会出现一个闹钟的小图标）
AlarmManager.AlarmClockInfo alarmClockInfo = new AlarmManager.AlarmClockInfo(timeSmallMillions,pi);
am.setAlarmClock(alarmClockInfo, pi);

```

# 网络访问方面,安卓10的适配

```

//在AndroidManifest.xml 的application中加入android:networkSecurityConfig="@xml/network_security_config"

//network_security_config内容

<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="true">
        <trust-anchors>
            <certificates src="system" />
        </trust-anchors>
    </base-config>
</network-security-config>

```

# 权限方面

```

//主要安卓6.0之后的部分权限的申请
//几个重要的接口

ContextCompact.checkSelfPermission() 检测是否拥有权限

ActivityCompact.requestPermission() 申请授权

onRequestPermissionsResult() 用户是否授权

ActivityCompat.shouldShowRequestPermissionRationale() 权限解释（用户拒绝后出现）

//例子

  /**
     * 权限申请
     */
    private String[] permissions = {
            Manifest.permission.WRITE_EXTERNAL_STORAGE,
            //Manifest.permission.LOCATION_HARDWARE,
            Manifest.permission.READ_PHONE_STATE,
            Manifest.permission.WRITE_SETTINGS,
            Manifest.permission.READ_EXTERNAL_STORAGE,
            Manifest.permission.RECORD_AUDIO,
            //Manifest.permission.READ_CONTACTS,
            Manifest.permission.ACCESS_FINE_LOCATION,
            Manifest.permission.SYSTEM_ALERT_WINDOW};
    private boolean isNeedPermission = true;
    private void requestPermissions(){
        try {
            if (Build.VERSION.SDK_INT >= 23) {
                for (int i=0;i<permissions.length;i++){
                    if (ActivityCompat.checkSelfPermission(this,permissions[i])!=PackageManager.PERMISSION_GRANTED){
                        isNeedPermission = true;
                        break;
                    }else {
                        isNeedPermission = false;
                    }
                }

                if (isNeedPermission){
                    ActivityCompat.requestPermissions(this,permissions,0x0010);
                }else {
                    
                }

            }else {
               
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

```

# 安卓7.0之后调用系统相册或者相机的问题

## 清单文件注册

```

<provider
      android:name="android.support.v4.content.FileProvider"
      android:authorities="包名.fileprovider"
      android:grantUriPermissions="true"
      android:exported="false">
      <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/file_paths" />
 </provider>

```

## 应用，调用相机为例

```

Intent intent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
 
  if (Build.VERSION.SDK_INT >= 24) {
          intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
         //24以上使用FileProvider
          intent.putExtra(MediaStore.EXTRA_OUTPUT, 
          FileProvider.getUriForFile(getContext(), "包名.fileprovider", mTmpFile));
  }else{
          //24以下
          intent.putExtra(MediaStore.EXTRA_OUTPUT, Uri.fromFile(mTmpFile));
       }

```
