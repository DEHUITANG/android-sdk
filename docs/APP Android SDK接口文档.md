©2016  **云智易**物联云平台（http://www.xlink.cn）


# XLINK Android SDK集成文档

本文档是面向智能硬件的APP开发者，通过Xlink Android SDK接入云智易物联平台。

## 一.开发前的准备

### 1. 功能概述

云智易android sdk 帮助开发者基于android系统上和xlink模块智能设备进行通讯。

### 2.集成准备

* 注册云智易企业帐号 http://admin.xlink.cn/#!/register
  在注册界面填写相关信息后,点击注册按钮,系统会向注册邮箱发一份激活邮件，    点击激活链接进行激活
* 激活之后在 http://admin.xlink.cn/#!/login 进行登陆
* 云平台首页中左侧栏点击添加产品进行产品的添加，输入产品描述并点击添加
* 获取到产品PID,产品密钥,企业ID(corp_id  点击右上角用户昵称，企业信息)

#####这样，前期的准备工作就算完成了。

### 3.配置程序
(参考Demo程序AndroidManifest.xml文件)

#### 3.1.在AndroidManifest.xml文件中application 标签下配置添加：

	<!-- XLINK 内网服务 -->
	<service android:name="io.xlink.wifi.sdk.XlinkUdpService" />               
	<!-- XLINK 公网服务 -->
	<service android:name="io.xlink.wifi.sdk.XlinkTcpService" />
	
	注意：如果缺少以上配置会造成sdk服务不能正常启动

#### 3.2. 添加sdk所需要权限

	<!-- 联网权限 -->                  
	<uses-permission android:name="android.permission.INTERNET" />
	<!-- 网络状态 -->                 
	<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
	<!-- wifi状态 -->                  
	<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
	<!-- wifi状态改变 -->
	<uses-permission android:name="android.permission.CHANGE_WIFI_MULTICAST_STATE" />



## 二.配置SDK  
(可参考Demo程序MyApp.java文件)

请先把文件夹内 xlink-wifi-sdk-v***.jar 文件拷贝到自己工程libs下。

### 1.初始化SDK
在自定义Application 下的onCreate()函数调用：

	  XlinkAgent.init(applicationContext);

参数会被长期引用，最好使用application的context。

### 2.设置监听器     

请确保使用sdk的功能前最少设置一个通用监听器：

	XlinkAgent.getInstance().addXlinkListener(XlinkNetListener);
	
	private XlinkNetListener XlinkNetListener = new XlinkNetListener() {
        
        @Override
        public void onStart(int code) {
            //监听启动sdk成功/失败回调
        }

        // 回调登录xlink状态
        @Override
        public void onLogin(int code) {
           
            if (code == XlinkCode.SUCCEED) {
                //云端网络连接成功回调
               
            } else if (code == XlinkCode.CLOUD_CONNECT_NO_NETWORK) {
             //   XlinkUtils.shortTips("网络不可用，请检查网络连接");
            }
        }

        @Override
        public void onLocalDisconnect(int code) {
            if (code == XlinkCode.LOCAL_SERVICE_KILL) {
                // 这里是xlink服务被异常终结了（第三方清理软件，或者进入应用管理被强制停止应用/服务）
                XlinkAgent.getInstance().start();//进行重启操作
            }
            XlinkUtils.shortTips("本地网络已经断开");
        }

        @Override
        public void onDisconnect(int code) {
            //云端连接断开回调
        }

        @Override
        public void onRecvPipeData(XDevice xdevice, byte[] data){
          //收到 局域网设备推送的pipe数据
        }

        @Override
        public void onRecvPipeSyncData(XDevice xdevice, byte[] data) {
        //收到设备通过云端服务器推送的pipe数据
        }

        @Override
        public void onDeviceStateChanged(XDevice xDevice, int i) {
          //设备状态改变回调
        }

        @Override
        public void onDataPointUpdate(XDevice xDevice, List<DataPoint> var2,int i2) {
            //数据端点的更新回调
        }

        @Override
        public void onEventNotify(EventNotify eventNotify) {

           //云端推送数据 
        }
    };

该监听器的功能有：(具体回调请参见XlinkNetListener说明)

value


函数：|说明：									
---| ---| ---	 				
onStart(int code);| 监听启动sdk成功/失败回调 			
onLogin(int code);|监听login登录服务器成功/失败回调
onLocalDisconnect(int code);|监听本地局域网断开回调
onDisconnect(int code);|监听和云端服务器连接状态回调
onRecvPipeSyncData(XDevice device,byte[] data);|收到设备发送同步Pipe数据
onDeviceStateChanged(XDevice xdevice, int state);|设备状态改变
onDataPointUpdate(XDevice xDevice, List<DataPoint> dataPionts, int channel);|设备数据端点改变
onEventNotify(EventNotify eventNotify)|应用内推送回调

注意事项:

1.当不需要该监听器时，可以调用
		 
	XlinkAgent.getInstance().removeListener(XlinkNetListener)
进行移除。

2.回调前需要调用 

     XlinkAgent.getInstance().login(appid, authKey); 
进行云端登陆才能正常回调，appId和authKey是调用登陆接口返回的用户数据


## 三.登录/注册 厂商自己的用户体系

### 1. 概述
参考文档:[http://support.xlink.cn/hc/kb/article/89925/](http://support.xlink.cn/hc/kb/article/89925/) 调用HTTP接口进行用户注册和用户认证,注册用户可通过企业管理后台进行用户管理
通过HTTP接口进行用户认证后, 可以获取该用户在云智易平台的唯一user_id auth_key。获取到 user_id 后，调用：

	XlinkAgent.getInstance().login(user_id,auth_Key);

才能使用云端网络和设备功能。

### 2.使用

* 在企业管理平台查看企业ID(COMPANY_ID),并参考文档:[http://support.xlink.cn/hc/kb/article/89925/](http://support.xlink.cn/hc/kb/article/89925/)和 Demo程序的注册登录流程(Demo里HttpManage.java对HTTP接口调用进行了封装,替换COMPANY_ID后可直接使用)。

### 3.注意事项

* 用户也可使用邮箱进行注册，如果使用邮箱进行注册,调用注册接口进行邮箱注册后，会发出一封注册邮件到用户邮箱上，用户通过点击邮箱中的激活链接完成注册,才能正常使用。



## 四.XlinkAgent类 方法说明

同步错误码(有INT返回值的函数)：

XlinkCode常量|int实际值|说明|使用的函数
---- | ---- | ---- |-----
`SUCCEED`|0|调用方法成功|ALL
`NO_INIT`|-1|未初始化SDK/未调用init(Context mContext)函数|ALL
`NO_HANDSHAKE`|-2|未和设备在局域网内连接成功|connectDevice(),setDeviceAuthorizeCode()，setDataPoint()，sendPipeData()
`ERROR_DEVICE`|-3|错误的设备（有传入设备参数的方法都有该错误码）|connectDevice(),setDeviceAuthorizeCode()，setDataPoint()，sendPipeData(),subscribeDevice()
`NO_CONNECT_SERVER`|-4|未连接服务器/未绑定UDP端口|scanDeviceByProductID函数回调该code是未调用start方法  
`NO_DEVICE`|-6|该设备在sdk未找到（操作设备前，需要调用initDevice(XDevice device)）|connectDevice(),setDeviceAuthorizeCode()，setDataPoint()，sendPipeData(),subscribeDevice()
`ALREADY_EXIST`|-7|任务重复，正在执行中，不允许重复调用；函数有: start  login connectDevice|start(), login(),connectDevice()
`INVALID_PARAM`|-8|参数有误（参数为空/密码长度太长/等等...）|所有带参数的函数
`INVALID_DEVICE_ID`|-9|无效的设备ID|所有带XDevice参数的函数
`NETWORD_UNAVAILABLE`|-10	|网络不可用|所有有联网操作的方法
`INVALID_POINT`|-11|非法的数据端点（1.该设备数据端点索引越界；2.value类型不匹配； 3.未添加正确的数据模版；|setDataPoint()
...	| ... | ... | ...

注意：这里的返回值只是表示调用成功，如果小于0则表示该函数未调用成功，自然也就不会回调监听器里面的函数。


### SDK功能函数：

### 1.void init(Context mContext)

#### 说明：

* 初始化xlink sdk,使用sdk前，必须调用

#### 参数：

| 参数 | 说明 |
| --- | --- |
mContext | ApplicationContext实例  

### 2.int start()

#### 说明：

* 启动内网服务，绑定udp 如第一次使用 需要连接到和wifi设备同一局域网
* 该链接会自己重连维护
* 如果确保不使用局域网，该方法可以不调用（可判断是否是移动网络）

#### 返回值:

| 值 | 说明 |
| --- | --- |  
| = 0 | 调用成功
| < 0 | 调用失败,失败code请观看同步错误码;

#### 对应回调:

    XlinkNetListener onStart(int code) 
 
>详情请参见XlinkNetListener 说明

### 3.int login(int user_id, String app_key)

#### 说明：

* 使用user_id和appkey登录到CM服务器。user_id和appkey的获取，请查看demo代码及用户HTTP接口开发文档 
* 获取到的user_id和app_key，由外部APP缓存维护。
* 该方法不用重复调用，调用一次后，会自己断线重连
* 只有login成功后，才能使用跟云端有关的服务

#### 参数：

| 参数 | 说明 |
| --- | --- |
| user_id | 通过HTTP接口获取到的连接云端的用户ID |
| app_key | 连接云端认证码 |

#### 返回值：

| 值 | 说明 |
| --- | --- |  
| = 0 | 调用成功
| < 0 | 调用失败,失败code请观看同步错误码;

#### 对应回调:

	XlinkNetListener onLogin(int code)

>详情请参见XlinkNetListener 说明

### 4.boolean initDevice(XDevice device)

#### 说明：

* 向SDK中初始化设备节点
* APP可以缓存设备节点，在下次程序启动后，可以自行初始化设备节点到SDK，用于跳过Scan步骤。
* SDK把设备节点放入内部队列，用于设备的定位和回调时的参数。
* 初始化设备时SDK不要向设备发送任何消息包，包括handshake。

#### 参数：

| 参数 | 说明 |
| --- | --- |
 XDevice | Device实体对象

#### 返回值：

| 值 | 说明 |
| --- | --- |
true  | 向SDK添加设备成功
false | 添加设备失败，设备属性错误


### 5.boolean setDataTemplate(String productID, String product)

#### 说明：

* 添加数据模版,在企业管理台定义的设备数据端点
* **V2版本SDK已废弃数据端点功能**
* 
#### 参数：

| 值 | 说明 |
| --- | --- |
prodctID | 产品id
product | 数据端点描述 是jsonarray格式.

#### 返回值：

| 值 | 说明 |
| --- | --- |
true  | 添加成功
false | 添加失败（product 不是json array格式）

### 6.XDevice JsonToDevice(JSONObject jsonObject)

#### 说明：

* 把jsonObject 序列化成Xdevice对象（除了scan，是获取XDevice实例的唯一接口）

#### 参数：

| 参数 | 说明 |
| --- | --- |
jsonObject | 设备的json对象

#### 返回值：

| 值 | 说明 |
| --- | --- |
XDevice | 序列化成功后的Device对象
NULL | jsonObject错误

### 7.JSONObject   deviceToJson(XDevice device)

#### 说明：

* 如果需要存储设备，通过此接口把device序列话成JSONObject对象，然后存储

#### 参数：

| 参数 | 说明 |
| --- | --- |
device | XDevice实例

### 8.void  stop()

#### 说明：

* 停止SDK，释放SDK所有占用的资源,在APP退出,或者不再使用SDK时调用。
* 该函数会清空initDevice()后设备列表;
* 清空addXlinkNetListener监听器列表；

### 9.void addXlinkListener(XlinkNetListener listener)

#### 说明：

* 添加XlinkNetListener 监听器，请确保全局至少有一个监听器。

### 10.void debug(boolean debug)

#### 说明：

* 设置日志是否打印

### 11.boolean isConnectedOuterNet()

#### 说明：

* 判断是否连接上xlink服务器

#### 返回值：

| 值 | 说明 |
| --- | --- |
true | 已连接 
false | 未连接

### 12.boolean isConnectedLocal()

#### 说明：

* 判断sdk 是否启动本地内网服务

#### 返回值：

| 值 | 说明 |
| --- | --- |
true | 启用 
false | 未启用

### 下面是操作设备的函数

### 1.int scanDeviceByProductID(String productId,ScanDeviceListener listener)

#### 说明：

* 通过productid扫描内网内所有对应的设备;需开启wifi 并连接到设备所在的wifi网络

#### 参数：
| 参数 | 说明 |
| --- | --- |
productid | 设备的产品id
ScanDeviceListener | listener 监听器

#### 返回值：
| 值 | 说明 |
| --- | --- |
| = 0 | 调用成功
| < 0 | 调用失败,失败code参见 同步错误码;

#### 对应回调:

   	ScanDeviceListener onGotDeviceByScan(XDevice device)
在扫描回调中, 可以通过device.getAccessKey()属性判断设备是否已设置设备授权码, 如果没设置,需要设置一下. 后面的用户通过判断是否设置AccessKey以及AccessKey是否一致,来控制是否需要把此设备添加到APP中


### 2.int setDeviceAccessKey(XDevice device, final int accessKey, final SetDeviceAccessKeyListener listener)

#### 说明：

* 修改设备密码(授权码)，必须在内网环境才能设置
* 已经设置过AccessKey的设备需要硬件复位清除AccessKey后才能重新设置

#### 参数：

| 参数 | 说明 |
| --- | --- |
| XDevice | Device实体对象
| accessKey | 设备密码(授权码) 支持9位数字
| listener | 监听器

#### 返回值：

| 值 | 说明 |
| --- | --- |
| = 0 | 调用成功 |
| < 0 | 调用失败,失败code参见 同步错误码 |

#### 结果回调：

	SetDeviceAccessKeyListener onSetLocalDeviceAccessKey(XDevice device, int code, int messageId)


### 3. int connectDevice(XDevice device,final int accessKey, ConnectDeviceListener connectListener)

#### 说明：

* 调用该函数用于连接设备，确定设备处于内网还是外网;

#### 参数：

| 参数 | 说明 |
| --- | --- |
| XDevice | Device实体兑现 | 
| accessKey | 设备授权码 | 
| connectListener | 监听器 | 

#### 返回值：

| 值 | 说明 |
| --- | --- |
| = 0 | 调用成功 |
| < 0 | 调用失败,失败code参见 同步错误码 |

#### 结果回调：

    ConnectDeviceListener  onConnectDevice(XDevice xDeivce, int ret)


### 4.int setDataPoint(XDevice xdevice, int key, Object value,SetDataPointListener listener)

#### 说明：

* 设置设备的数据端点

#### 参数：

| 参数 | 说明 |
| --- | --- |
| DeviceObject| Device实体对象
| key | 端点索引
| value | 端点值

#### 返回值：

| 值 | 说明 |
| --- | --- |
| = 0 | 调用成功 |
| < 0 | 调用失败,失败code参见 同步错误码 |

#### 结果回调：

	SetDataPointListener onSetDataPoint();

### 5.int subscribeDevice(XDevice device, int accessKey, SubscribeDeviceListener listener)

#### 说明：

* 订阅设备(必须有公网环境)(如果在公网环境下使用
* 公网环境调用 XlinkAgent#connectDevice()会自动调用该函数;
* 设置设备密码后，订阅关系会清空
* 解除订阅关系请调用HTTP接口(/v2/user/{user_id}/unsubscribe) 参考:[http://support.xlink.cn/hc/kb/article/89925/](http://support.xlink.cn/hc/kb/article/89925/)

#### 参数：

| 参数 | 说明 |
| --- | --- |
| DeviceObject | Device实体对象
| accessKey | 设备授权码
| listener | 监听器

#### 返回值：

| 值 | 说明 |
| --- | --- |
| = 0 | 调用成功；
| < 0 | app本地错误;详情参见同步错误码;

#### 结果回调：

	onSubscribeDevice()

### 6.int sendPipeData(XDevice device, byte[] data, SendPipeListener listener)

#### 说明：

* 向设备发送pipe数据包.

#### 参数：

| 参数 | 说明 |
| --- | --- |
| DeviceObject | Device实体对象
| byte[] | Pipe数据

#### 返回值：

| 值 | 说明 |
| --- | --- |
| = 0 | 调用成功；
| < 0 | app本地错误;详情参见同步错误码;


#### 结果回调：

	onSendPipeData

## 五.XlinkNetListener 回调说明：

### 1. onStart(int code)

#### 说明：

* 调用XlinkAgent.start()的回调

#### 返回值：

XlinkCode常量 | int实际值 | 说明
---- | ---- | ----
`SUCCEED`|0|成功
`LOCAL_CONNECT_ERROR`	|-1	|绑定端口失败
...|...|...

### 2. onLogin(int code)  

#### 说明：

* 调用XlinkAgent.login的回调(如果已经login 服务器成功，是不会再回调该函数)

#### 返回值 
XlinkCode 常量|int实际值|说明
---- | ---- | ----
`SUCCEED`|0|登录服务器成功
`CLOUD_CONNECT_ERROR`|-1|连接公网服务器失败（解析域名失败/无网络连接/网络响应超时）
`CLOUD_CONNECT_NO_NETWORK`|-2|无物理网络连接
`TIMEOUT`|-100|登录服务器超时（原因：手机网络不稳定)
`SERVER_CODE_INVALID_PARAM`|1|参数错误（sdk内部错误）
`SERVER_CODE_INVALID_KEY`|2|app key不正确
`SERVER_CODE_UNAVAILABLE_ID`|3|非法的 appid
`SERVER_CODE_SERVER_ERROR`|4|服务器内部错误
... | ...	| ...

### 3.  onDisconnect(int code)

#### 说明：

* 对应于login，当app从xlink服务掉线时，会回调该方法。云端连接会自己断线重连(网络异常，心跳异常才会，其他异常需要处理)，不用重复调用Login()方法

#### 返回值:
XlinkCode 常量|int实际值|说明
---- | ---- | ----
`CLOUD_STATE_DISCONNECT`|-1|网络问题导致和服务器连接中端(不需要处理，会自动重连)
`CLOUD_KEEPALIVE_ERROR`|-2|和服务器心跳异常，导致从服务器掉线(不需要处理，会自动重连)
`CLOUD_SERVICE_KILL`|-3|XlinkTcpServrce服务被异常杀死（如360等安全软件）.（需要重新调用login函数）
`CLOUD_USER_EXTRUSION`|-4|该用户在其他地方登录(提示用户，帐号被挤)


### 4. onLocalDisconnect(int code);

#### 说明：

* 本地服务断开

#### 返回值：
XlinkCode 常量|int实际值|说明
---- | ---- | ----
`LOCAL_THREAD_ERROR`|-1|无物理网络
`LOCAL_SERVICE_KILL`|-2|XlinkUdpServrce服务被异常杀死（如360等安全软件),需要重新调用start函数。
...|...|...

### 5.  onRecvPipeData(XDevice device, byte flags, byte[] data);

#### 说明：

* 收到局域网内设备推送的pipe数据 （跟设备直连，返回的数据）

#### 参数 :

| 参数 | 说明 |
| --- | --- |
| device | 设备实体
|flags|标识|
| data | byte数据

### 6. onRecvPipeSyncData(XDevice device, byte flags, byte[] data);

#### 说明：

＊ 收到服务器推送的同步pipe数据

#### 参数 :

| 参数 | 说明 |
| --- | --- |
| device | 该设备的 pipe数据
|flags|标识|
| data | byte数据

### 7. onDataPointUpdate(XDevice xDevice, int key,Object value, int channel,int type);

#### 说明：

* 设备数据节点发生改变，会回调此方法
* **V2版本SDK已废弃数据端点功能**

#### 参数 :

| 参数 | 说明 |
| --- | --- |
| device | 设备实例
| key | 端点索引
| value | 端点值
| channel |  通道，区分是在局域网直连设备的更新端点，还是云端更新端点、`channel == XlinkCode.CHANGED_UPDATAPOINT_LOCAL`  为本地局域网通道`channel == XlinkCode.CHANGED_UPDATAPOINT_CLOUD`  为yun'd云端网络通道
type | value的类型

type 定义|具体int值|说明
---- | ---- | ----
`POINT_TYPE_BOOLEAN`|1|布尔值
`POINT_TYPE_BYTE`|2|byte单字节
`POINT_TYPE_SHORT`|3|int16 (short)
`POINT_TYPE_INT`|4|int32 (int)
`POINT_TYPE_STRING`|5|string


### 9.onDeviceStateChanged(XDevice xdevice, int state);

#### 说明：

* 设备当连接成功/掉线，如果设备有状态变化会回调该方法.
* 每当设备掉线，会自动调用connectDevice 一次。

#### 参数 :
| 参数 | 说明 |
| --- | --- |
| device | 设备实例
| state | 状态

state 定义 | int实际值 |	说明
-----|-----|-----
`DEVICE_CHANGED_CONNECTING`|	-1|	设备重新连接中
`DEVICE_CHANGED_OFFLINE`|	-2|	设备掉线
`DEVICE_CHANGED_CONNECT_SUCCEED`|	-3|	设备重新连接成功
...|	...|	...

## 六.XLINK 设备的操作

### 1.设备和用户的关系:
#### ①.内网模式:

	内网通讯不经过云智易服务器，在无互联网环境下也能使用设备;
	在这种模式下，用户和设备唯一凭证是设备密码
	内网控制之前，必须先调用XlinkAgent connectDevice()函数，否则进行设备操作，会同步返回 NO_HANDSHAKE -2的错误码

#### ②.公网模式:

	设备激活后，设备有属性 deviceID,该属性是每个设备在云智易后台唯一不重复的标识
	在公网环境，调用XlinkAgent connectDevice()函数连接设备成功后，SDK会自动把当前user 跟 deviceID 进行订阅绑定;

#### ③.设备密码:

	每个设备都有一个设备密码，设备需要设置密码后才能使用,该密码是设备的唯一凭证;
	可调用 XlinkAgent setDeviceAccessKey() 进行修改设备密码（也是通过该方法设置初始密码）
	修改设备密码成功后，服务器会清空以前所有的订阅关系，这时需要重新订阅设备才能使用公网进行控制设备
    修改设备密码需要有内网环境(设备跟APP在同一局域网)
    已经设置过AccessKey的设备需要硬件复位清除AccessKey后才能重新设置

#### ③.设备的分享:

	调用deviceTojson把需要分享的设备序列号成功json字符串，然后把该json字符串分享（分享方式有二维码，等等）给被分享人，
	在被分享人中调用jsonToDevice把该字符串反序列化成XDevice实体类。
    详细请查考文档:http://support.xlink.cn/hc/kb/article/108289/

### 2.设备的获取:

#### ①.配置设备上网

	配置设备上网是和wifi厂商有关，不同的wifi芯片对应的配置程序和SDK不同，一般是由WIFI厂商提供
	这个不属于XLINK SDK的一部分。需要APP开发者找到对应的   WIFI配置SDK，自行将功配置功能集成到APP中。
	比如HF的Smartlink，庆科的Easylink等


#### ②.当用户第一次使用SDK时，通过扫描获取

	XlinkAgent scanDeviceByProductId(String productId,   ScanDeviceListener listener) 获取设备实例

#### ③.如果有设备实例序列化对象，通过该接口获取：

	XlinkAgent  JsonToDevice(JSONObject jsonObject) 把jsonObject 反序列化成设备实例

### 3.设备的存储:  

#### ①.SDK不提供存储设备,SDK有提供了设备序列化接口:

    XlinkAgent  deviceToJson(XDevice device)
	可以获取该设备的jsonObject对象，而后可转换为String字符串进行存储(云端/本地)
	Demo程序的存储方式是数据库本地存储；

#### ②.如果需要云端同步设备，可使用云智易Http接口, 使用用户扩展属性和设备扩展属性进行同步

	(详情请查阅文档 http://support.xlink.cn/hc/kb/article/89925/  )
	注意：在app运行周期过程中，设备的属性可能会被更改（如 deviceID, 设备版本号等信息）;
	当退出程序之前，需要重新调用deviceToJson 接口，获取最新的设备json对象;
	然后匹配下之前的设备json对象，如果有修改，则需要把最新的设备json对象替换。            

### 4.设备的操作:

#### ①. 扫描到设备后可通过isInit() 属性判断设备是否初始化过，如果未初始化设备需要设置初始授权码才能使用设备

(1).流程如下：      

* Scan Device
	* isInit()==true
		* 连接设备需要提示，让用户输入之前设置过的设备密码
		* 连接设备成功
		* 控制设备
	* isInit()==false
		* 设置设备初始密码(设置成功后，自行存储)
		* 连接设备
		* 连接设备成功
		* 控制设备


(2). 序列化接口不提供存储设备密码，设备密码存储由第三方自行决定，可参考Demo;

	如果设备安全等级不高，上述步骤可由APP自动实现；
	（Demo就是这种方式：扫描设备成功后，判断isInit==false则设置内置的设备密码，然后连接设备都是传入进行连接）

#### ②. 如果使用 XlinkAgent  JsonToDevice(JSONObject jsonObject) 方法获取的设备实例需要调用：

	XlinkAgent  initDevice(XDevice device) 
	向SDK添加设备；否则调用和设备相关的接口会返回错误码:NO_DEVICE

#### ③. 如果需要扩展设备属性,可参考Demo程序

	io.xlink.wifi.js.bean/Device.java

### 5.设备的控制:

#### ①. 数据透传:

	XlinkAgent sendPipeData(XDevice device, byte[] data, SendPipeListener listener) ;

#### ②. 数据端点控制：

	XlinkAgent setDataPoint(XDevice xdevice, int key, Object value,SetDataPointListener listener)

### 6.设备的推送数据:

#### ①. 数据透传:

	XlinkNetListener  onRecvPipeData(XDevice device, byte flags, byte[] data);
	该接口回调出设备对当前app发送的数据。
	XlinkNetListener onRecvPipeSyncData(XDevice device, byte flags, byte[] data);
	该接口回调出设备对所以在线订阅过xdevice的app发送的数据

注意：有可能回调出来的XDevice 对象只有 deviceId一个属性,是因为在sdk内部未找到当前XDevice实例,只能把带deviceId一个属性的设备回调出来

#### ②. 数据端点:

	XlinkNetListener  onDataPointUpdate(XDevice xdevice,int key,Object value,int hannel,int type)
	如果设备定义了数据端点，当设备数据端点有变化时，会通过该函数回调出设备当前的数据端点

### 1.XDevice暴露属性：

boolean isInit()

	说明返回该设备是否第一次使用(第一次使用原密码，可直接设置新密码，使用设备必须先设置设备密码)
boolean isHandshake()

	说明返回该设备是否可用户局域网直连
boolean isValidId()

	说明返回该设备是否激活
int getDevcieConnectStates()

	说明返回该设备当前连接状态XlinCode.DEVICE_STATE_OFFLIEN等
String getProductId

	说明：返回该设备产品ID
String getAddress

	说明：返回该设备在局域网的ip地址
String  getMacAddress

	说明：返回该设备mac地址,唯一的设备标识量

String  getAccessKey

	说明：返回该设备密码,扫描回调中获取的对象此属性才准确
    
其他不必要属性请见XDevice API文档

### 2.EventNotify属性：
 * byte notyfyFlags:
        bit0:来自server的事件
        bit1:来自其他device的事件
        bit2:来自其他APP的事件
        bit3:收到事件后要不要应答, 默认都不需要应答
        bit4-7:预留 Reserved
 * int formId :发送者ID 如果是服务端发送的消息, id为0
 * int messageType:
        1:设备端点变化发送的通知
        2:设备端点变化引起的警报
        3:设备管理员推送的分享消息
        4:厂商推送的消息广播
        5:设备属性变化通知
        当 messageType=1 or 2 时,
        notifyData: 前2个字节为字符串长度,后面的所有数据为UTF8格式的字符串
 * byte[] notifyData 推送数据



## 七.监听器返回Code说明

###返回码都在XlinkCode类下  

### 1.ConnectDeviceListener onConnectDevice(XDevice xDevice, int result);

	连接设备监听器
	retsult有:

XlinkCode 常量|	int返回值|	说明
-----|-----|-----
`DEVICE_STATE_OUTER_LINK`|	1|	连接设备成功，设备处于公网环境
`DEVICE_STATE_LOCAL_LINK`|	0|	连接设备成功，设备处于局域网环境
`CONNECT_DEVICE_TIMEOUT`|	200	|连接设备超时，设备假在线/手机网络差/设备网络差
`CONNECT_DEVICE_SERVER_ERROR`|	104|	app内部错误
`CONNECT_DEVICE_INVALID_KEY`|	102	|设备认证失败，密码错误
`CONNECT_DEVICE_OFFLINE`|	110	|设备不在线
`CONNECT_DEVICE_NO_ACTIVATE`|	109	|设备未激活
`CONNECT_DEVICE_OFFLINE_NO_LOGIN`|	111|	设备局域网不在线，且当前app只有局域网环境，无法云端连接设备
...|	...	|...


### 2.ScanDeviceListener onGotDeviceByScan(XDevice device);

#### 说明
	扫描设备监听器
	
#### 参数
	device：扫描到的设备对象;

### 3.SendPipeListener onSendLocalPipeData(XDevice device, int code,messageId);

#### 说明

	发送pipe数据监听器
	
#### code定义

XlinkCode 常量|	INT返回值|	说明
-----|-----|-----
`SUCCEED`	|0|	成功（除了SUCCEED，其他的都是发送失败）
`TIMEOUT`|	-100|	发送pipe数据超时
`SERVER_CODE_INVALID_PARAM`|	1	|公网环境下：参数错误。内网环境下：会话ID已经失效，需要重新连接设备，一般出现在设备重启过。
`SERVER_CODE_UNAVAILABLE_ID`|	3|	设备id错误（设备未激活）
`SERVER_CODE_SERVER_ERROR`|	4|	服务器内部错误
`SERVER_CODE_UNAUTHORIZED`|	5|	设备未授权（无订阅关系，需要调用函数connectDevice/subscribeDevice 重新在服务器建立订阅关系）
`SERVER_DEVICE_OFFLIEN`|	10|	设备不在线
...|	...|	...


> messageId  : 跟调用sendPipe接口返回的 msgId 一一对应


### 4.SetDataPointListener onSetDataPoint(XDevice xdevice, int code，int messageId);

#### 说明
	设置设备数据端点
	注：V2版本SDK已废弃数据端点功能

#### code定义：
> 跟SendPipeListener 一样

> messageId：跟调用sendPipe接口返回的 msgId 一一对应

### 5.SetDeviceAccessKeyListener  onSetLocalDeviceAccessKey(XDevice device, int code,messageId);

#### 说明：
	设置设备密码
	
#### code定义：
> 返回的code 跟SendPipeListener 一样，新增以下错误码：

XlinkCode | INT值 | 说明
---- | ---- | ----
`SERVER_CODE_INVALID_KEY`	|2|	设置密码失败，认证设备失败（oldAuth错误）
> messageId  : 跟调用sendPipe接口返回的 msgId 一一对应


### 6.SubscribeDeviceListener onSubscribeDevice(XDevice device, int code);

#### 说明：
	订阅设备
	
#### code定义：
	返回的code 跟SendPipeListener  一样
	
##八.数据端点
### 3.设置设备数据模版

如果设备使用了数据端点，需要根据在后端定义的设备数据端点在SDK进行配置，这样SDK才能解析该设备端点。 

例如下面表格样的数据端点：

索引（key）|变量名|  备注  |数据类型（type）
----|----|----|-----
1|LED|三色灯|单字节
2|KEY|KEY1(按键1)|布尔
3|inr|红外感应距离|单字节
4|Temp|温湿度|短整型
5|RF_IN|无线遥控码|长整型
6|Name|同步名称|字符串
...|...|...|...

转化成json串为：

	[{"key":1,"type":"byte"},{"key":2,"type":"bool"},{"key":3,"type":"byte"},{"key":4,"type":"int16"},{"key":5,"type":"int32"},{"key":6,"type":"string"}]

通过函数：

	XlinkAgent.setDataTemplate(“设备的产品id”, prodctid_value);

进行设置数据端点（可参考demo程序MyApp.java）

### 注意事项:

* 扫描,操作设备之前需先设置该设备的数据端点；(必须设置，不然解析不出数据端点)
* 如该设备未定义数据端点也需要这样设置： XlinkAgent.setDataTemplate(“该设备的产品id”,“[]”);
* 该设备的产品id 可在企业管理平台查看；
* SDK支持多产品ID；


##九.进行打包混淆

### 1.混淆打包

如果的项目使用了Proguard混淆打包，为了避免SDK被二次混淆导致无法正常使用SDK，请务必在proguard-project.txt中添加以下代码：

	-libraryjars libs/xlink-wifi-sdk-v*.jar #（具体请查看jar名称）
	-dontwarn io.xlink.wifi.sdk.**
	-keep class io.xlink.wifi.sdk.**{
	         *;
	}



©2016 云智易物联云平台（http://www.xlink.cn）
