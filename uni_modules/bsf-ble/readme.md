# bsf-ble

此项目是基于BLE蓝牙原生协议专为`uniapp`/`uniappx`的App项目定制的`UTS`插件，支持Android、IOS，可实现蓝牙搜索，蓝牙连接，消息订阅，数据发送，数据接收，数据解析等功能。

[文档](https://github.com/byteee-fund/bsf-ble-doc) | [DEMO](https://github.com/byteee-fund/bsf-ble-demo)

## 兼容性

- Android
- IOS
- Uniapp / Uniappx


## 使用说明

### 引入插件

```js
import * as BLEManager from "@uni_modules/bsf-ble";

```

## API接口

### 基础方法

#### checkPermission()
检查是否已获取蓝牙权限
```js
BLEManager.checkPermission(): boolean
```

#### requestPermission(options)
请求蓝牙权限
```js
BLEManager.requestPermission({
  onPermit: () => void,  // 权限授予回调
  onRefuse: () => void   // 权限拒绝回调
})
```

#### isBleEnable()
检查蓝牙是否启用
```js
BLEManager.isBleEnable(): boolean
```

#### initBle(options)
初始化蓝牙模块
```js
BLEManager.initBle({
  onBleStateChange: (state: number) => void,  // 蓝牙状态变化回调
  onCharacteristicChanged: (res: any) => void,  // 特征值变化回调
  onStateChange: (address: string, state: number) => void,  // 设备连接状态变化回调
  onServicesDiscovered: () => void  // 服务发现完成回调
})
```

蓝牙状态码说明：
- 0: unknown (未知状态)
- 1: resetting (正在重置)
- 2: unsupported (不支持蓝牙)
- 3: unauthorized (未授权)
- 4: poweredOff (蓝牙关闭)
- 5: poweredOn (蓝牙开启)

### 设备操作

#### getConnectedDevices(serviceUUIDs)
获取已连接的设备列表
```js
BLEManager.getConnectedDevices(serviceUUIDs: string[]): Device[]
```

#### startScan(options)
开始扫描蓝牙设备
```js
BLEManager.startScan({
  onDeviceScaned: (device: Device) => void  // 发现设备回调
})
```

#### stopScan()
停止扫描蓝牙设备
```js
BLEManager.stopScan()
```

#### connect(options)
连接蓝牙设备
```js
BLEManager.connect({
  address: string  // 设备地址
})
```

#### disconnect(address)
断开蓝牙设备连接
```js
BLEManager.disconnect(address: string)
```

### 数据传输

#### openNotification(options)
开启特征值变化通知
```js
BLEManager.openNotification({
  serviceUUID: string,  // 服务 UUID
  characteristicUUID: string,  // 特征值 UUID
  callback: (res: any) => void  // 数据接收回调
})
```

#### closeNotification(options)
关闭特征值变化通知
```js
BLEManager.closeNotification({
  serviceUUID: string,  // 服务 UUID
  characteristicUUID: string,  // 特征值 UUID
  callback: (res: any) => void  // 关闭结果回调
})
```

#### writeData(options)
向设备写入数据
```js
BLEManager.writeData({
  data: number[],  // 要写入的数据
  serviceUUID: string,  // 服务 UUID
  characteristicUUID: string  // 特征值 UUID
})
```

## 类型定义

### Device 接口
```ts
interface Device {
  address: string;  // 设备地址
  isConnected: boolean;  // 连接状态
  // 其他设备信息...
}
```



