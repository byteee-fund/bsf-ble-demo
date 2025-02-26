<template>
  <view class="content">
    <!-- 顶部标题 -->
    <view class="header">
      <text class="app-title">bsf-ble示例工程</text>
    </view>
	  <text class="list-title">蓝牙状态： {{bleState ? "可用": "不可用"}}</text>	
	  <view class="device-list">
		<text class="list-title">已连接的设备</text>
		<!-- 当前设备信息 -->
		<view class="device-info" v-for="(device, index) in connectDevices" :key="index" >
		  <text class="label">绑定设备:</text>
		  <text class="value">{{ device?.name || '未绑定' }}</text>
		  <text class="label">连接状态:</text>
		  <text class="value">{{ device?.isConnected ? "已连接" : "未连接" }}</text>
		  
		  <!-- 按钮操作区 -->
		  <view class="action-buttons">
		    <button @click="connect(device)" class="button primary" :disabled="device?.isConnected">连接</button>
		    <button @click="notify" class="button primary" :disabled="!device?.isConnected">监听</button>
		    <button @click="writeData" class="button primary" :disabled="!device?.isConnected">写入</button>
		    <button @click="disconnectDevice" class="button danger" :disabled="!device?.isConnected">断开</button>
		  </view>
		</view>
	</view>

   
	
	
	<button @click="startScan" class="button primary">开始扫描</button>
	<button @click="stopScan" class="button secondary">停止扫描</button>

    <!-- 扫描到的设备列表 -->
    <view class="device-list">
      <text class="list-title">扫描到的设备</text>
      <view v-for="(device, index) in devices" :key="index" class="device-item">
        <text>名称: {{ device?.name || '未知' }}</text>
        <text>地址: {{ device?.address || '未知' }}</text>
        <button @click="connect(device)" class="connect-button">连接</button>
      </view>
    </view>
	
	<view class="footer">
		<text>Copyright © BSF 2025</text>
		<text class="link" @click="openUrl">https://byteee.fund</text>
	</view>
  </view>
</template>




<script>
import * as BLEManager from "../../uni_modules/bsf-ble";

export default {
  data() {
    return {
      state: "未连接",
	  serviceUUID: "0000FFE0-0000-1000-8000-00805F9B34FB",
	  notifyCharateristicUUID: "66880002-0000-1000-8000-008012563489",
	  writeCharateristicUUID: "66880001-0000-1000-8000-008012563489",
	  macAddress: "F7:2A:B8:73:C6:AC", // 安卓
	  bleState: false,
	  connectDevices: [], 
      devices: [], // 扫描到的设备列表
      device: null, // 当前绑定的设备
      isConnected: false // 是否已连接状态
    };
  },
  onLoad() {
    // this.loadCachedDevice();
    this.requestPermission();
  },
  onShow() {
	this.getConnectedDevices();
	this.bleState = BLEManager.isBleEnable();
  },
  methods: {

    // 请求蓝牙权限
    requestPermission() {
      if (!BLEManager.checkPermission()) {
		  console.log("未请求权限");
        BLEManager.requestPermission({
          onPermit: () => {
			  console.log("权限已授予")
			this.init();	
		  },
          onRefuse: () => {
			  uni.showToast({
			  	title: "权限未开启",
				duration:2000
			  });
			  console.log("权限被拒绝")
		  }
        });
      } else {
		  console.log("权限已申请");
		  this.init();
		  // let devices = BLEManager.getConnectedDevices([this.serviceUUID]);
		  // 	console.log(devices);
	  }
    },
	
	getConnectedDevices() {
		let deviceRet = BLEManager.getConnectedDevices([this.serviceUUID]);
		 this.connectDevices = deviceRet.map((device) => {
			return {
			  ...device,
			  isConnected: true
			  // isConnected: BLEManager.isConnected(device.address) // 判断连接状态
			};
		  });
		  
		  // // 自动重连
		  // if (!connected) {
		  // 	console.log("连接设备");
		  // 	BLEManager.connect({
		  // 	  address: "328F1B08-7548-4C38-A5AA-C8DB04FD5B6C",
		  // 	});
		  // }
		  
		console.log(this.connectDevices);
	},
	init() {
			// 初始化蓝牙
			BLEManager.initBle({
				onBleStateChange: (state) => {
					// unknown	0	未知状态（CBCentralManager 未初始化）
					// resetting	1	正在重置状态
					// unsupported	2	当前设备不支持蓝牙
					// unauthorized	3	应用未获得蓝牙权限
					// poweredOff	4	蓝牙已关闭
					// poweredOn	5	蓝牙已打开且准备工作
					console.log(state);
					if (state) {
									
						// 获取连接的设备
						this.getConnectedDevices();
							
					}
					
					this.bleState = state;
					
				},
				onCharacteristicChanged: (res) => {
						console.log(res);
				},
				 onStateChange: (address, state) => {
					  console.log(address, state);
						
					 let deviceExists = false;
					 
					     this.connectDevices = this.connectDevices.map((device) => {
					         if (device.address === address) {
					             deviceExists = true; // 标记设备已存在
					             return {
					                 ...device,
					                 isConnected: state === 2, // 更新连接状态
					             };
					         }
					         return device;
					     });
					 
					     // 如果 state == 2 且设备不存在于 connectDevices 中，添加新的设备
					     if (state === 2 && !deviceExists) {
					         this.connectDevices.push({
					             address: address,
					             isConnected: true, // 新设备的连接状态为 true
					         });
					     }
				 },
				 onServicesDiscovered: () => {
					 BLEManager.openNotification({
					   serviceUUID: this.serviceUUID,
					   characteristicUUID: "0000FFE3-0000-1000-8000-00805F9B34FB",
					   callback: (res) => {
								console.log(res);
								
							}
					 });
					 
					 
					 BLEManager.openNotification({
					   serviceUUID: this.serviceUUID,
					   characteristicUUID: "0000FFE2-0000-1000-8000-00805F9B34FB",
					   callback: (res) => {
								console.log(res);
								
							}
					 });
					 
					 
					 BLEManager.writeData({
					   data: [-90, 3, 66, 1, 0, 70, 106],
					   serviceUUID: this.serviceUUID,
					   characteristicUUID: "0000FFE3-0000-1000-8000-00805F9B34FB"
					 });
				 }
			});
	},
	
    // 开始扫描设备
    startScan() {
      this.devices = [];
      BLEManager.startScan({
		  // serviceUUIDs: ["1812"],
        onDeviceScaned: (device) => {
          const exists = this.devices.some((d) => d.address === device.address);
          if (!exists) this.devices.push(device);
        },
       
      });
    },
    stopScan() {
      BLEManager.stopScan();
    },
    connect(device) {
      BLEManager.connect({
        address: device.address,
      });
    },
	connectScanDevice(device) {
	  BLEManager.stopScan();
	  BLEManager.connect({
	    address: device.address,
	  });
	},
    disconnectDevice() {
		// BLEManager.writeData({
		//   data: [13],
		//   serviceUUID: this.serviceUUID,
		//   characteristicUUID: this.charateristicUUID
		// });
		// 关闭通知
		BLEManager.closeNotification({
			serviceUUID: this.serviceUUID,
			  characteristicUUID: this.notifyCharateristicUUID,
			  callback: (res)=> {
			  	console.log(res);
			  }
		})
		setTimeout(()=>{
			BLEManager.disconnect("328F1B08-7548-4C38-A5AA-C8DB04FD5B6C");
		}, 1000)
		

    },
    notify() {
      BLEManager.openNotification({
        serviceUUID: this.serviceUUID,
        characteristicUUID: this.notifyCharateristicUUID,
        callback: (res) => {
			console.log(res);
			if (res == "0d") {
				   BLEManager.disconnect("328F1B08-7548-4C38-A5AA-C8DB04FD5B6C");					 
			}
		}
      });
    },
    writeData() {
      BLEManager.writeData({
        data: [1, 0],
        serviceUUID: this.serviceUUID,
        characteristicUUID: this.writeCharateristicUUID
      });
    },
    // readData() {
    //   BLEManager.readData({
    //     serviceUUID: this.serviceUUID,
    //     characteristicUUID: this.charateristicUUID
    //   });
    // },
    // updateState(state) {
    //   this.state = state === 2 ? "已连接" : "未连接";
    // }
	openUrl() {
		// #ifdef APP-PLUS
		plus.runtime.openURL('https://byteee.fund', (err) => {
			if (err) {
				uni.showToast({
					title: '打开链接失败',
					icon: 'none'
				});
			}
		});
		// #endif
		
		// #ifdef H5
		window.open('https://byteee.fund', '_blank');
		// #endif
	},
  }
};

</script>


<style>
.content {
  display: flex;
  flex-direction: column;
  padding: 20rpx;
}

.header {
  text-align: center;
  margin-bottom: 20rpx;
}

.app-title {
  font-size: 48rpx;
  font-weight: bold;
  color: #007aff;
}

.device-info {
  margin-bottom: 20rpx;
  padding: 20rpx;
  background-color: #f0f0f0;
  border-radius: 8rpx;
}

.label {
  font-size: 32rpx;
  color: #333;
}

.value {
  font-size: 36rpx;
  color: #007aff;
  margin-bottom: 10rpx;
}

.action-buttons {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  margin-bottom: 20rpx;
}

.button {
  margin: 10rpx;
  padding: 20rpx;
  border-radius: 8rpx;
  font-size: 32rpx;
}

.primary {
  background-color: #007aff;
  color: white;
}

.secondary {
  background-color: #f0ad4e;
  color: white;
}

.danger {
  background-color: #d9534f;
  color: white;
}

.device-list {
  margin-top: 20rpx;
}

.list-title {
  font-size: 36rpx;
  margin-bottom: 10rpx;
  color: #333;
}

.device-item {
  padding: 20rpx;
  margin-bottom: 10rpx;
  background-color: #ffffff;
  border-radius: 8rpx;
  box-shadow: 0 2rpx 5rpx rgba(0, 0, 0, 0.2);
}

.connect-button {
  margin-top: 10rpx;
  background-color: #007aff;
  color: #fff;
  padding: 10rpx 20rpx;
  border-radius: 4rpx;
  text-align: center;
}

</style>
