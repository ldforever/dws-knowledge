# 大华 SDK 接口参考

> 来源：`LogisticsBaseCSharp.xml`（C# 封装 XML 注释）与 `LogisticsBaseCSharp.dll` 反编译。原生接口位于 `LogisticsBase64.dll`（C++）。

## 1. 生命周期 API

| API | 说明 |
|---|---|
| `vslbCreateBaseApp()` | 创建 BaseApp 句柄 |
| `vslbInitBaseApp(handle, cfgPath, bool)` | 初始化，读取配置文件 |
| `vslbRun(handle, VslbRunResultCB)` | 启动，注册结果回调 |
| `vslbStop(handle)` | 停止 |
| `vslbFiniBaseApp(handle)` | 反初始化 |
| `vslbDestroyBaseApp(handle)` | 销毁句柄 |

## 2. 回调注册 API

| API | 回调 | 触发 |
|---|---|---|
| `vslbAttachCameraDisconnectCB` | `VslbCameraDisconnectCB(cameraKey)` | 相机断线 |
| `vslbAttachRealImageCB` | `VslbRealImageCB(cameraKey, cameraIp, image)` | 实时图 |
| `vslbAttachAllCameraCodeinfoCB` | `VslbAllCameraCodeInfoCB` | 包裹结束，所有相机扫码信息 |
| `vslbAttachIpcCombineInfoCB` | `VslbIpcCombineInfoCB` | 全景+条码拼接图 |
| `vslbAttachRealErrorInfoCB` | `VslbRealErrorInfoCB(timeStamp, errorType, errorContent)` | 绑定异常 |

## 3. 控制 API

| API | 说明 |
|---|---|
| `vslbSetStopSignal(handle, signal, val)` | 通知/停止信号 |
| `vslbSetSerialPortCommand(handle, command)` | 串口称指令：1 开机 / 2 停机 / 3 报警停机 / 4 报警恢复重启 |
| `vslbSetCameraIoOutput(handle, command)` | 相机 IO：0 停止称 / 1 启动称 |
| `vslbWndSendMsg(handle, filePath, barcodeInfo, sendMode)` | 句柄定向输出（0 不带回车，非 0 带回车） |
| `vslbGetWorkCameraKey(handle, index, tags)` | 获取工作相机 Key |

## 4. 结果结构体 VslbRunResult

```text
VslbRunResult
├── image           原始图片
├── codes           条码列表
├── resultAreas     条码区域
├── steadyWeight    稳定重量
├── waybillImage    面单图像
├── volumeInfo      体积信息（length/width/height/volume, mm/mm³）
└── outputResult    输出模块返回值（0 成功 / -1 失败）
```

## 5. 上层封装（LogisticsWrapper）

C# 封装把原生回调转成 .NET 事件：

| 事件参数类 | 属性 |
|---|---|
| `LogisticsCodeEventArgs` | CameraID（相机 IP 或图片路径）、OutputResult、CodeList |
| `RealImageArgs` | CameraID、Image |
| `CameraStatusArgs` | 相机连接状态 |
| `AllCameraCodeInfoArgs` | 各相机扫码信息数组 |
| `IpcCombineInfoArgs` | 全景图/抠图/拼接图/命名信息 |
| `RealErrorInfo` | 时间戳、异常类型、异常描述 |

## 6. 初始化模式

```csharp
var handle = LogisticsAPI.vslbCreateBaseApp();
LogisticsAPI.vslbInitBaseApp(handle, "Cfg/LogisticsBase.cfg", true);
LogisticsAPI.vslbAttachCameraDisconnectCB(handle, OnCameraDisconnect);
LogisticsAPI.vslbAttachRealImageCB(handle, OnRealImage);
LogisticsAPI.vslbAttachAllCameraCodeinfoCB(handle, OnAllCameraCode);
LogisticsAPI.vslbAttachIpcCombineInfoCB(handle, OnIpcCombine);
LogisticsAPI.vslbAttachRealErrorInfoCB(handle, OnError);
LogisticsAPI.vslbRun(handle, OnRunResult);
// ...
LogisticsAPI.vslbStop(handle);
LogisticsAPI.vslbFiniBaseApp(handle);
LogisticsAPI.vslbDestroyBaseApp(handle);
```

## 7. 对 Node.js 重建的启示

这套 API 的设计可以直接映射为 Node.js 的事件总线/服务接口：

| 原生概念 | Node.js 对应 |
|---|---|
| `vslbRun` + 回调 | 采集服务启动 + 事件发射（`EventEmitter`） |
| `OnPacketResultReached` | `codeRead.packetResult` 事件 |
| `VslbRunResult` | 包裹结果对象（TypeScript interface） |
| `vslbSetStopSignal` | 控制接口（REST/gRPC/IPC） |
| `vslbWndSendMsg` | 输出通道写入 |

> [!note] V1 适用性
> V1 采用**智能相机六面扫**，相机端自带解码并直接上报条码，因此**不调用 vslb* 原生 SDK**。上述 API 仅作为"事件/结果模型"的设计参考（见 [[04-六面扫智能相机对接方案]] 第 9 节的概念映射）。

---

相关文档：[[01-系统技术架构]] | [[02-核心模块技术细节]] | [[01-技术选型方案]]
