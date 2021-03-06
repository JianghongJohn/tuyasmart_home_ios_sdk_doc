# 蓝牙 BLE




单点蓝牙设备指的是具有和手机终端**通过蓝牙一对一连接**的设备，例如蓝牙手环、蓝牙耳机、蓝牙音箱等。每个设备最多同时和一个手机进行蓝牙连接，每个手机终端目前**同时蓝牙连接数量控制在 6～7 个**内

|           类名            |        说明        |
| :-----------------------: | :----------------: |
|    TuyaSmartBLEManager    |   单点蓝牙相关类   |
| TuyaSmartBLEWifiActivator | 双模设备配网相关类 |

`TuyaSmartBLEManager`包含单点蓝牙 SDK 的所有相关功能，包含扫描设备、单点设备配网、双模设备配网、蓝牙设备操作、固件升级、错误码等功能

 `TuyaSmartBLEWifiActivator`包含设备双模配网所需方法



## 准备工作


### 系统蓝牙状态监测

**接口说明**

SDK 提供了对系统蓝牙的状态监测的方法，在蓝牙状态变化（如开启或关闭）时，可以通过设置代理收到具体的消息

**示例代码**

Objc:

```objective-c
// 设置代理
[TuyaSmartBLEManager sharedInstance].delegate = self;


/**
 * 蓝牙状态变化通知
 *
 * @param isPoweredOn 蓝牙状态，开启或关闭
 */
- (void)bluetoothDidUpdateState:(BOOL)isPoweredOn {
    NSLog(@"蓝牙状态变化: %d", isPoweredOn ? 1 : 0);
}
```

Swift:

```swift
// 设置代理
TuyaSmartBLEManager.sharedInstance().delegate = self

/**
 * 蓝牙状态变化通知
 *
 * @param isPoweredOn 蓝牙状态，开启或关闭
 */
func bluetoothDidUpdateState(_ isPoweredOn: Bool) {
        
}

```



## 扫描设备


###开始扫描

**接口说明**

处于待连接的蓝牙设备都会不停的向四周发蓝牙广播包，客户端作为终端可以发现这些广播包，根据广播包中包含涂鸦设备信息规则作为目标设备的过滤条件

```objective-c
/**
 * 开始扫描
 *
 * 如果扫描到未激活设备，结果会通过 `TuyaSmartBLEManagerDelegate` 中的 `- (void)didDiscoveryDeviceWithDeviceInfo:(TYBLEAdvModel *)deviceInfo` 返回;
 *
 * 如果扫描到激活设备，会自动进行连接入网，不会返回扫描结果
 *
 * @param clearCache 是否清理已扫描到的设备
 */
- (void)startListening:(BOOL)clearCache;
```

**参数说明**

| 参数       | 说明                   |
| ---------- | ---------------------- |
| clearCache | 是否清理已扫描到的设备 |

**示例代码**

Objc:

```objective-c
// 设置代理
[TuyaSmartBLEManager sharedInstance].delegate = self;

// 开始扫描
[[TuyaSmartBLEManager sharedInstance] startListening:YES];


/**
 * 扫描到未激活的设备
 *
 * @param deviceInfo 未激活设备信息 Model 
 */
- (void)didDiscoveryDeviceWithDeviceInfo:(TYBLEAdvModel *)deviceInfo {
    // 成功扫描到未激活的设备
    // 若设备已激活，则不会走此回调，且会自动进行激活连接
}
```

Swift:

```swift
TuyaSmartBLEManager.sharedInstance().delegate = self
TuyaSmartBLEManager.sharedInstance().startListening(true)

/**
 * 扫描到未激活的设备
 *
 * @param uuid 未激活设备 uuid
 * @param productKey 未激活设备产品 key
 */
func didDiscoveryDevice(withDeviceInfo deviceInfo: TYBLEAdvModel) {
    // 成功扫描到未激活的设备
    // 若设备已激活，则不会走此回调，且会自动进行激活连接
}
```


### 停止扫描

停止扫描设备，比如退出配网页面 或者 在执行设备入网时，建议停止扫描，以防止扫描影响到配网过程

**接口说明**

```objective-c
/**
 * 停止扫描
 *
 * @param clearCache 是否清理已扫描到的设备
 */
- (void)stopListening:(BOOL)clearCache;
```

**参数说明**

| 参数       | 说明                   |
| ---------- | ---------------------- |
| clearCache | 是否清理已扫描到的设备 |
**示例代码**

Objc:

```objective-c
[[TuyaSmartBLEManager sharedInstance] stopListening:YES];
```

Swift:

```swift
TuyaSmartBLEManager.sharedInstance().stopListening(true)
```



## 单点设备配网


### 单点设备入网

**接口说明**

扫描到未激活的设备后，可以进行设备激活并且注册到涂鸦云，并记录在家庭下

```objective-c
/**
 * 激活设备
 * 激活过程会将设备信息注册到云端
 */
- (void)activeBLE:(TYBLEAdvModel *)deviceInfo
           homeId:(long long)homeId
          success:(void(^)(TuyaSmartDeviceModel *deviceModel))success
          failure:(TYFailureHandler)failure;
```

**参数说明**

| 参数       | 说明                                         |
| ---------- | -------------------------------------------- |
| deviceInfo | 设备信息 Model，来源于扫描代理方法返回的结果 |
| homeId     | 当前家庭 Id                                  |
| success    | 成功回调                                     |
| failure    | 失败回调                                     |

**示例代码**

Objc:

```objective-c
[[TuyaSmartBLEManager sharedInstance] activeBLE:deviceInfo homeId:homeId success:^(TuyaSmartDeviceModel *deviceModel) {
        // 激活成功
        
    } failure:^{
        // 激活中的错误
    }];
```

Swift:

```swift
TuyaSmartBLEManager.sharedInstance().activeBLE(<deviceInfo: deviceInfo, homeId: homeId, success: { (deviceModel) in
        // 激活成功  
        }) {
        // 激活中的错误
        }
```



## 双模设备配网


### 双模设备入网

**接口说明**

扫描到未激活的设备后，可以进行设备激活并且注册到涂鸦云，并记录在家庭下

```objective-c
/**
 *  connect ble wifi device
 *  连接蓝牙 Wifi 设备
 */
- (void)startConfigBLEWifiDeviceWithUUID:(NSString *)UUID
                                  homeId:(long long)homeId
                               productId:(NSString *)productId
                                    ssid:(NSString *)ssid
                                password:(NSString *)password
                                timeout:(NSTimeInterval)timeout
                                 success:(TYSuccessHandler)success
                                 failure:(TYFailureHandler)failure;
```

**参数说明**

| 参数      | 说明           |
| --------- | -------------- |
| UUID      | 设备 uuid      |
| homeId    | 当前家庭 Id    |
| productId | 产品 Id        |
| ssid      | 路由器热点名称 |
| password  | 路由器热点密码 |
| timeout   | 轮询时间       |
| success   | 成功回调       |
| failure   | 失败回调       |

**示例代码**

Objc:

```objective-c
  [[TuyaSmartBLEWifiActivator sharedInstance] startConfigBLEWifiDeviceWithUUID:TYBLEAdvModel.uuid homeId:homeId productId:TYBLEAdvModel.productId ssid:ssid password:password  timeout:100 success:^{
     // 激活成功
        } failure:^{
     // 激活失败
        }];
```

Swift:

```swift
  TuyaSmartBLEWifiActivator.sharedInstance() .startConfigBLEWifiDevice(withUUID: TYBLEAdvModel.uuid, homeId: homeId, productId:TYBLEAdvModel.productId, ssid: ssid, password: password, timeout: 100, success: {
            // 激活成功
        }) {
            // 激活失败
        }
```


### 双模配网设备激活回调

**接口说明**

配网的结果通过 delegate 方法回调

**示例代码**

Objc:

```objective-c
- (void)bleWifiActivator:(TuyaSmartBLEWifiActivator *)activator didReceiveBLEWifiConfigDevice:(TuyaSmartDeviceModel *)deviceModel error:(NSError *)error {
    if (!error && deviceModel) {
			// 配网成功
    }
  
    if (error) {
    	// 配网失败
    }
}
```

Swift:

```swift
func bleWifiActivator(_ activator: TuyaSmartBLEWifiActivator, didReceiveBLEWifiConfigDevice deviceModel: TuyaSmartDeviceModel, error: Error) {
    if (!error && deviceModel) {
			// 配网成功
    }

    if (error) {
    	// 配网失败
    }
}
```


### 取消双模设备入网

**接口说明**

停止发现双模设备

**示例代码**

Objc:

```objective-c
//在配网结束后调用
[[TuyaSmartBLEWifiActivator sharedInstance] stopDiscover];
```

Swift:

```swift
//在配网结束后调用
TuyaSmartBLEWifiActivator.sharedInstance() .stopDiscover
```



## 蓝牙设备操作


### 判断设备在线状态

**接口说明**

查询设备蓝牙是否本地连接

```objective-c
- (BOOL)deviceStatueWithUUID:(NSString *)uuid;
```

**参数说明**

| 参数       | 说明      |
| ---------- | --------- |
| uuid    | 设备 uuid      |


**示例代码**

Objc:

```objective-c
  BOOL isOnline = [TuyaSmartBLEManager.sharedInstance deviceStatueWithUUID:uuid];
```

Swift:

```swift
  var isOnline:BOOL = TuyaSmartBLEManager.sharedInstance().deviceStatue(withUUID: "uuid")
```


### 连接设备

**接口说明**

若设备处于离线状态 可以调用连接方法进行设备连接

```objective-c
- (void)connectBLEWithUUID:(NSString *)uuid
                productKey:(NSString *)productKey
                   success:(TYSuccessHandler)success
                   failure:(TYFailureHandler)failure;
```

**参数说明**

| 参数    | 说明           |
| ------- | -------------- |
| uuid    | 设备 uuid      |
| productKey     | 产品 Id        |
| success | 成功回调       |
| failure | 失败回调       |

**示例代码**

Objc:

```objective-c
  [[TuyaSmartBLEManager sharedInstance] connectBLEWithUUID:@"your_device_uuid" productKey:@"your_device_productKey" success:success failure:failure];
```

Swift:

```swift
  TuyaSmartBLEManager.sharedInstance().connectBLE(withUUID: @"your_device_uuid", productKey: @"your_device_productKey", success: success, failure: failure)
```


### 设备 DP 下发

控制发送请参考[设备功能点发送](https://tuyainc.github.io/tuyasmart_home_ios_sdk_doc/zh-hans/resource/Device.html#%E8%AE%BE%E5%A4%87%E5%8A%9F%E8%83%BD%E7%82%B9)


### 查询设备名称

**接口说明**

当扫描到设备广播包并拿到设备广播包对象后，可以通过该方法查询设备名称

```objective-c
/**
 * 查询设备名称
 */
- (void)queryNameWithUUID:(NSString *)uuid
               productKey:(NSString *)productKey
                  success:(void(^)(NSString *name))success
                  failure:(TYFailureError)failure;
```

**参数说明**

| 参数       | 说明      |
| ---------- | --------- |
| uuid       | 设备 uuid |
| productKey | 产品 Id   |
| success    | 成功回调  |
| failure    | 失败回调  |

**示例代码**

Objc:

```objective-c
[[TuyaSmartBLEManager sharedInstance] queryNameWithUUID:bleAdvInfo.uuid productKey:bleAdvInfo.productId success:^(NSString *name) {
        // 查询设备名称成功
        
    } failure:^{
        // 查询设备名称失败
    }];
```

Swift:

```swift
TuyaSmartBLEManager.sharedInstance().queryName(withUUID: bleAdvInfo.uuid, productKey: bleAdvInfo.productId, success: { (name) in
        // 查询设备名称成功                                                                                              
}, failure: { (error) in
        // 查询设备名称失败
})
```



## 固件升级


### 获取设备升级信息

**接口说明**

```objective-c
- (void)getFirmwareUpgradeInfo:(nullable void (^)(NSArray <TuyaSmartFirmwareUpgradeModel *> *upgradeModelList))success failure:(nullable TYFailureError)failure;
```

**参数说明**

| 参数    | 说明                             |
| ------- | -------------------------------- |
| success | 成功回调，设备的固件升级信息列表 |
| failure | 失败回调                         |

**`TuyaSmartFirmwareUpgradeModel` 数据模型**

| 字段          | 类型      | 说明                                            |
| ------------- | --------- | ----------------------------------------------- |
| desc          | NSString  | 升级文案                                        |
| typeDesc      | NSString  | 设备类型文案                                    |
| upgradeStatus | NSInteger | 0:无新版本 1:有新版本 2:在升级中 5:等待设备唤醒 |
| version       | NSString  | 新版本使用的固件版本                            |
| upgradeType   | NSInteger | 0:App 提醒升级 2:App 强制升级 3:检测升级         |
| url           | NSString  | 蓝牙设备的升级固件包下载 URL                    |
| fileSize      | NSString  | 固件包的 size, byte                             |
| md5           | NSString  | 固件的 md5                                       |
| upgradingDesc | NSString  | 固件升级中的提示文案                            |

**示例代码**

Objc:

```objc
- (void)getFirmwareUpgradeInfo {
	// self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];

	[self.device getFirmwareUpgradeInfo:^(NSArray<TuyaSmartFirmwareUpgradeModel *> *upgradeModelList) {
		NSLog(@"getFirmwareUpgradeInfo success");
	} failure:^(NSError *error) {
		NSLog(@"getFirmwareUpgradeInfo failure: %@", error);
	}];
}
```

Swift:

```swift
func getFirmwareUpgradeInfo() {
    device?.getFirmwareUpgradeInfo({ (upgradeModelList) in
        print("getFirmwareUpgradeInfo success")
    }, failure: { (error) in
        if let e = error {
            print("getFirmwareUpgradeInfo failure: \(e)")
        }
    })
}
```


### 设备 OTA 升级

**接口说明**

对于有固件升级的设备，可以通过发送升级固件数据包对设备进行升级。其中升级固件包需要先请求云端接口进行获取固件信息

```objective-c
/**
 * 发送 OTA 包，升级固件。升级前请务必保证设备已通过蓝牙连接
 */
- (void)sendOTAPack:(NSString *)uuid
                pid:(NSString *)pid
            otaData:(NSData *)otaData
            success:(TYSuccessHandler)success
            failure:(TYFailureHandler)failure;
```

**参数说明**

| 参数    | 说明           |
| ------- | -------------- |
| uuid    | 设备 uuid      |
| pid     | 产品 Id        |
| otaData | 升级固件的数据 |
| success | 成功回调       |
| failure | 失败回调       |

**示例代码**

Objc:

```objective-c
- (void)getFirmwareUpgradeInfo {
    // self.device = [TuyaSmartDevice deviceWithDeviceId:@"your_device_id"];

    [self.device getFirmwareUpgradeInfo:^(NSArray<TuyaSmartFirmwareUpgradeModel *> *upgradeModelList) {
        NSLog(@"getFirmwareUpgradeInfo success");
    } failure:^(NSError *error) {
        NSLog(@"getFirmwareUpgradeInfo failure: %@", error);
    }];
}

// 如果有升级，其中 TuyaSmartFirmwareUpgradeModel.url 是固件升级包的下载地址
// 根据 url 下载固件后，将数据转成 data，传给 sdk 进行固件升级
// deviceModel -- 需要升级的设备 model
// data -- 下载的固件包
[[TuyaSmartBLEManager sharedInstance] sendOTAPack:deviceModel.uuid pid:deviceModel.pid otaData:data success:^{
       NSLog(@"OTA 成功");
    } failure:^{
       NSLog(@"OTA 失败");
}];

```

Swift:

```swift
func getFirmwareUpgradeInfo() {
    device?.getFirmwareUpgradeInfo({ (upgradeModelList) in
        print("getFirmwareUpgradeInfo success");
    }, failure: { (error) in
        if let e = error {
            print("getFirmwareUpgradeInfo failure: \(e)");
        }
    })
}

// 如果有升级，其中 TuyaSmartFirmwareUpgradeModel.url 是固件升级包的下载地址
// 根据 url 下载固件后，将数据转成 data，传给 sdk 进行固件升级
// deviceModel -- 需要升级的设备 model
// data -- 下载的固件包
TuyaSmartBLEManager.sharedInstance().sendOTAPack(deviceModel.uuid, pid: deviceModel.pid, otaData: data, success: {
    print("OTA 成功");
}) {
    print("OTA 失败");
}
```



## 错误码

| 错误码 | 说明                     |
| ------ | ------------------------ |
| 1      | 设备接收的数据包格式错误 |
| 2      | 设备找不到路由器         |
| 3      | Wi-Fi 密码错误           |
| 4      | 设备连不上路由器         |
| 5      | 设备 DHCP 失败           |
| 6      | 设备连云失败             |
| 100    | 用户取消配网             |
| 101    | 蓝牙连接错误             |
| 102    | 发现蓝牙服务错误         |
| 103    | 打开蓝牙通讯通道失败     |
| 104    | 蓝牙获取设备信息失败     |
| 105    | 蓝牙配对失败             |
| 106    | 配网超时                 |
| 107    | Wi-Fi 信息发送失败       |
| 108    | Token 失效               |
| 109    | 获取蓝牙加密密钥失败     |
| 110    | 设备不存在               |
| 111    | 设备云端注册失败         |
| 112    | 设备云端激活失败         |
| 113    | 云端设备已被绑定         |
| 114    | 主动断开                 |
| 115    | 云端获取设备信息失败     |
| 116    | 设备此时正被其他方式配网 |
| 117    | OTA 升级失败             |
| 118    | OTA 升级超时             |
| 119    | Wi-Fi 配网传参校验失败   |

