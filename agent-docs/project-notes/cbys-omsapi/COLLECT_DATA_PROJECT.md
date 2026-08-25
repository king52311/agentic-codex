# collect-data 采集项目说明

`collect-data` 是根据 `product/LiteDJHarborDemo` 反编译结构重新整理的新采集服务。

## 旧逻辑映射

- RocketMQ 消费：`MQConsumeMsgListenerProcessor` -> `transport/RocketMqConsumerRunner`
- 华为平台消息解析：`ResHandles.resHandleByData` -> `adapter/huawei/HuaweiHarborCollector`
- 电信赛恩报文解析：`StorageEcuHandles.StorageEcu` -> `parser/SaienMeterPayloadParser`
- 原始消息入库：`DBDevice/AddResponse.xml` -> `mapper/CollectDataMapper.insertRawDeviceMessage`
- 最新/历史表具入库：`AddRequest.xml` -> `repository/CollectDataRepository`

## 新结构

- `adapter`：平台适配。新增平台实现 `PlatformCollector` 即可。
- `parser`：设备协议解析。平台适配层可组合不同解析器。
- `domain`：统一领域对象，如 `RawDeviceMessage`、`MeterReading`。
- `repository`：公共数据库写入逻辑，集中处理 `nb_device`、`ecu`、`ecu_history`。
- `transport`：输入通道，目前包含 HTTP 和 RocketMQ。

## 扩展方式

接入其他平台时：

1. 新增 `PlatformCollector` 实现。
2. 在 `extract` 中把平台原始消息转换成 `RawDeviceMessage`。
3. 在 `parseReading` 中调用对应设备协议解析器，输出 `MeterReading`。
4. 公共入库逻辑无需改动。

## 当前状态

- 已创建独立 Spring Boot 项目 `collect-data`。
- 默认端口 `7078`。
- RocketMQ 默认关闭，配置开启后消费 `DianxinNa / DianxinNaTag`。
- `mvn -q -DskipTests compile` 已通过。
