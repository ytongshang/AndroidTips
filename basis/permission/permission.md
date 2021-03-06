# Permission

## 签名

- 签名的唯一性，一方面保证了signature-level permissions，另一方面一个应用可以和另一个应用具有相同的linux identity的应用共享数据，通过指定androidManifest.xml中manifest的名为shardUserId属性

 ```xml
 <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.shareusertesta"
    android:versionCode="1"
    android:versionName="1.0"
    android:sharedUserId="com.example">
 ```

- 默认情况下getSharedPreferences(String, int), openFileOutput(String, int),
 或 openOrCreateDatabase(String, int, SQLiteDatabase.CursorFactory),所产生的数据只能被当前应用读写，
 但是如果设置了MODE_WORLD_READABLE and/or MODE_WORLD_WRITEABLE，虽然产生的数据仍然是当前应用的，
 但是可以被其它应用读写

## 自定义权限

- 首先在AndroidManifest.xml中定义权限：
    - 其中android:protectionLevel是必须的;
    - android:permissionGroup是可选的，可以加入现有的permission group,也可以加入自定义的permission group，
    - android:label和android:description一般都要加上
    - label是简单描述
    - 而description是详细描述，一般由这个权限干什么的和获得了这个权限可能会出现什么问题组成

 ```java
<permission
 android:protectionLevel="signature"
 android:name="com.bignerdranch.android.photogallery.PRIVATE"/>
<uses-permission android:name="com.bignerdranch.android.photogallery.PRIVATE" />

public static final String PERM_PRIVATE ="com.bignerdranch.android.photogallery.PRIVATE";
sendBroadcast(new Intent(ACTION_SHOW_NOTIFICATION), PERM_PRIVATE);

@Override
public void onResume() {
    super.onResume();
    IntentFilter filter = new IntentFilter(PollService.ACTION_SHOW_NOTIFICATION);
    getActivity().registerReceiver(mOnShowNotification, filter,PollService.PERM_PRIVATE, null);
}
 ```

## 权限

- 权限可以显式的分为normal permissions 和 dangerous permissions
- 对于列在androidmanifest.xml中的normal permissions，android系统会在安装应用时自己确保获得这些权限
- 对于列在androidmanifest.xml中的dangerous permissions,android 系统会显式的向用户请求这些权限

### 权限group

- **android所有的权限被分为不同的group**

Group      | Permissions
-----------|-------------------------------------------------------------------------------------------------------
CALENDAR   | READ_CALENDAR WRITE_CALENDAR
CAMERA     | CAMERA
CONTACTS   | READ_CONTACTS WRITE_CONTACTS GET_ACCOUNTS
LOCATION   | ACCESS_FINE_LOCATION ACCESS_COARSE_LOCATION
MICROPHONE | RECORD_AUDIO
PHONE      | READ_PHONE_STATE CALL_PHONE READ_CALL_LOG WRITE_CALL_LOG ADD_VOICEMAIL USE_SIP PROCESS_OUTGOING_CALLS
SENSORS    | BODY_SENSORS
SMS        | SEND_SMS RECEIVE_SMS READ_SMS RECEIVE_WAP_PUSH RECEIVE_MMS
STORAGE    | READ_EXTERNAL_STORAGE WRITE_EXTERNAL_STORAGE

### Android 6.0

- 对于android 6.0，如果系统请求一个列在AndroidManifest.xml中的危险权限：
    - **如果系统对于该权限所属的group的任何其它权限都没有获得权限时，会显示一个对话框，向用户请求权限;**
    - **如果系统已经有了该权限所属group的任何其它权限，则系统会立即授予该权限，而不会提示用户**,比如原来已经获得了READ_CONTACTS的权限，如果再次请求WRITE_CONTACTS时，则会立即获得写联系人的权限，而不会展示请求对话框

### Android 8.0

- [Android8.0运行时权限策略变化和适配方案](http://blog.csdn.net/yanzhenjie1003/article/details/76719487)

- **在 Android O 之前，如果应用在运行时请求权限并且被授予该权限，系统会错误地将属于同一权限组并且在清单中注册的其他权限也一起授予应用**
- **对于针对Android O的应用，此行为已被纠正。系统只会授予应用明确请求的权限。然而一旦用户为应用授予某个权限，则所有后续对该权限组中权限的请求都将被自动批准**
- 例如：假设某个应用在其清单中列出READ_EXTERNAL_STORAGE和WRITE_EXTERNAL_STORAGE。
    - 应用请READ_EXTERNAL_STORAGE，并且用户授予了该权限，
    - 如果该应用针对的是API级别24或更低级别，系统还会同时授予WRITE_EXTERNAL_STORAGE，因为该权限也属于STORAGE权限组并且也在清单中注册过。
    - 如果该应用针对的是Android O，则系统此时仅会授予READ_EXTERNAL_STORAGE，**不过在该应用以后申请WRITE_EXTERNAL_STORAGE权限时，系统会立即授予该权限，而不会提示用户**，这一点与6.0的是一样的
    - 区别在于8.0如果只请求了READ_EXTERNAL_STORAGE，系统只会授予READ_EXTERNAL_STORAGE，而**如果要使用WRITE_EXTERNAL_STORAGE必须再次申请**，虽然只要请求就会授予，所以对于权限一般最好一个权限组的申请

### 请求权限

- 对于android系统是6.0以下或者应用的targetSdkVersion<=22时，只会在安装或者升级时才会请求授予对应的权限，
 并且也是以一个一个权限group请求的.并且一旦安装后，唯一的修改权限的方法就是卸载app
- 如果**android系统是6.0并且targetSdkVersion>=23**,系统会在运行时动态的向用户请求权限，而且用户可以随时取消已经授予过的权限，因而需要在每一次运行特定权限代码时，需要动态的检测是否已经拥有了对应的权限
- 一般情况下，权限失败，会导致SecurityException，但是这个不一定会在所有的地方发生，比如sendBroadCast(intent)，一般请求权限失败会打印在log中
- 常见的会出现请求权限的时候：
    - 调用需要特定权限的系统函数时;
    - startActivity,可以防止启动其它应用的activities;
    - send和receive broadcast，可以控制谁可以接受自己的广播，也可以控制谁可以向自己发送广播；
    - 当操作content provider的时候；
    - bind或者start 一个service的时候

### shouldShowRequestPermissionRationale()

- 系统提供了一个shouldShowRequestPermissionRationale()函数，这个函数的作用是帮助开发者找到需要向用户额外解释权限的情况:
    - 应用安装后第一次访问，直接返回false；
    - 第一次请求权限时，用户拒绝了，下一次shouldShowRequestPermissionRationale()返回 true，这时候可以显示一些为什么需要这个权限的说明；
    - 第二次请求权限时，用户拒绝了，并选择了“不再提醒”的选项时：shouldShowRequestPermissionRationale()返回 false；
    - 设备的系统设置中禁止当前应用获取这个权限的授权，shouldShowRequestPermissionRationale()返回false；
    - 注意：第二次请求权限时，才会有“不再提醒”的选项，如果用户一直拒绝，并没有选择“不再提醒”的选项，下次请求权限时，会继续有“不再提醒”的选项，并且shouldShowRequestPermissionRationale()也会一直返回true

### URI permission

- 对于content provider要保证读写的权限，但是有的时候更要保证只对特定的uri持有权限，而不会影响到其它的的内容，
 比如邮件，对于一封邮件有权限，并不应当保证对所有的邮件有权限，因而出现了uri permission，只对特定uri资源有权限
- 解决办法是在startActivity或者从一个activity获得结果时，为intent设置flag,Intent.FLAG_GRANT_READ_URI_PERMISSION and/or Intent.FLAG_GRANT_WRITE_URI_PERMISSION,这样就保证了只有有了对应的权限，才能读取对应的内容
- content provider实现支持uri permission的机制，通过 <grant-uri-permissions>tag,和android:grantUriPermissions属性
- 常用方法：Context.grantUriPermission(), Context.revokeUriPermission(), 和Context.checkUriPermission

```java
public static final String NOTE_ACTION_VIEW = "notes.intent.action.NOTE_VIEW";

Uri uri = Uri.pars("content://com.granturipermissiontest.providers.NotesContentProvider/notes");
Intent intent = new Intent();
intent.setAction(NOTE_ACTION_VIEW);
intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
intent.addFlags(Intent.FLAG_GRANT_WRITE_URI_PERMISSION);
intent.setData(uri);
startActivity(intent);
```

## Enforcing Permissions in AndroidManifest.xml

- High-level权限会影响整个应和相关组件的运行
- activity permission,在Context.startActivity() 和Activity.startActivityForResult()的时候会检测对应的权限，如果没有满足，会抛SecurityException
- service permission,会在Context.startService(), Context.stopService() 和Context.bindService()的时候检查，如果不满足，会抛SecurityException
- BroadcastReceiver权限，限制了谁可以向对应的receiver发送intent的权限，会在Context.sendBroadcast()之后检查，如果不满足，不会抛异常，只会导致不会发送对应的Intent
- 对于Context.sendBroadcast(),可以提供权限控制哪些对应的receiver可以接受到对应的inent
- 对于Context.registerReceiver(),可以提供权限哪些可以向它发送Intent
- 对于content provider，相关的权限控制了对应数据的读写（URI permisson）,而且对于数据的读写分别有 android:readPermisson和android:writePermission,并且很重要的一点是，不像其它的权限，有了android： writePermission就一定会有对应的android:readPermission,**两者要分别获取**，在执行ContentResolver.query() 时需要读的权限，而使用ContentResolver.insert(), ContentResolver.update(), ContentResolver.delete()则需要写的权限

## FileProvider

- [Android开发之深入理解Android 7.0系统权限更改相关文档](https://www.cnblogs.com/dazhao/p/6547811.html)

## 悬浮窗

## IPC权限检查（跨进程通信）

- 在ipc中，可以通过checkCallingPermission (String permission)，处理调用的process是否有对应的权限，并且这一函数只能在ipc调用的过程中使用，其它的情况下都会返回没有对应的权限
- 如果知道了一个应用的pid,可以能过Context.checkPermission(String, int, int)检查它是否有对应的权限
- 如果知道了一个应用的packagename,可以通过PackageManager.checkPermission(String, String)检查它是否有对应的权限

## permission与feature

- 有些permission需要特定的feature支持，当使用对应特性时，需要要androidManifest.xml指定使用对应的feature
