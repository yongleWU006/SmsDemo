# mob短信验证（无UI最简demo）
官方文档http://wiki.mob.com/sdk-sms-android-3-0-0/

## 集成步骤
1. 注册mob平台开发者，获取appkey
2. 下载sdk http://www.mob.com/downloadDetail/SMS/android, 在/SMSSDK/SMSSDK/lib下把3个jar包拷出来放到你的工程中，添加依赖
```
compile name: 'SMSSDK-3.0.0', ext: 'aar'
compile files('libs/MobTools-2017.0607.1736.jar')
compile files('libs/MobCommons-2017.0607.1736.jar')
```
3. 声明一个Application，在AndroidManifest文件中添加appkey和appsecret(以下是引用官方的MobApplication，但最好还是使用自己定义的)
```
<application
        android:name="com.mob.MobApplication"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <meta-data android:name="Mob-AppKey" android:value="your-appkey"/>
        <meta-data android:name="Mob-AppSecret" android:value="your-appsecret"/>
        
</application>
```
4. 然后在Application的onCreate方法中调用
```
MobSDK.init(this);
```
5. 在activity中调用请求验证码
```
SMSSDK.getVerificationCode("86", "用户手机号码");
```
6. 验证手机和验证码
```
SMSSDK.submitVerificationCode("86","用户手机号码","验证码");
```
7. 注册事件回调，用于处理提交验证码操作的结果
```
EventHandler eh=new EventHandler(){

            @Override
            public void afterEvent(int event, int result, Object data) {
                Message msg = new Message();
                msg.arg1 = event;
                msg.arg2 = result;
                msg.obj = data;
                mHandler.sendMessage(msg);
            }

        };
        SMSSDK.registerEventHandler(eh);
        
 Handler mHandler = new Handler(){
        public void handleMessage(Message msg) {

            // TODO Auto-generated method stub
            super.handleMessage(msg);
            int event = msg.arg1;
            int result = msg.arg2;
            Object data = msg.obj;
            Log.e("event", "event=" + event);
            if (result == SMSSDK.RESULT_COMPLETE) {
                System.out.println("--------result"+event);
                //短信注册成功后，返回MainActivity,然后提示新好友
                if (event == SMSSDK.EVENT_SUBMIT_VERIFICATION_CODE) {//提交验证码成功
                    Toast.makeText(getApplicationContext(), "提交验证码成功", Toast.LENGTH_SHORT).show();

                } else if (event == SMSSDK.EVENT_GET_VERIFICATION_CODE){
                    //已经验证
                    Toast.makeText(getApplicationContext(), "验证码已经发送", Toast.LENGTH_SHORT).show();
                } 

            } else {
                // 处理错误的结果
//				((Throwable) data).printStackTrace();
//				Toast.makeText(MainActivity.this, "验证码错误", Toast.LENGTH_SHORT).show();
//					Toast.makeText(MainActivity.this, "123", Toast.LENGTH_SHORT).show();
                int status = 0;
                try {
                    ((Throwable) data).printStackTrace();
                    Throwable throwable = (Throwable) data;

                    JSONObject object = new JSONObject(throwable.getMessage());
                    String des = object.optString("detail");
                    status = object.optInt("status");
                    if (!TextUtils.isEmpty(des)) {
                        Toast.makeText(MainActivity.this, des, Toast.LENGTH_SHORT).show();
                        return;
                    }
                } catch (Exception e) {
                    SMSLog.getInstance().w(e);
                }
           }


        }
    };
```
8. 获取验证码回调,重写onReadVerifyCode
```
public void onReadVerifyCode(String s) {
        textV.setText(s);
    }
```
9. 用完回调要注销，否则会造成泄露
```
protected void onDestroy() {
        super.onDestroy();
        SMSSDK.unregisterAllEventHandler();
    };
```
