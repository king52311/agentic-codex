# API 定时任务梳理

当前 `MsApiApplication` 中 `@EnableScheduling` 被注释，Spring `@Scheduled` 定时任务不会自动执行。

## Spring 定时任务

| 类 | 方法 | 时间 | 说明 |
| --- | --- | --- | --- |
| `Taskmain` | `getLHData` | 每天 13:00 | 获取浪花数据 |
| `Taskmain` | `getHXData` | 每天 12:30 | 获取环翔数据 |
| `Taskmain` | `getSMData` | 每天 12:45 | 获取水门数据 |
| `Taskmain` | `getBHData` | 每天 20:00 | 获取炳华数据 |
| `Taskmain` | `getTaskOpenLh` | 每 10 分钟 | 浪花批量开阀 |
| `Taskmain` | `getTaskAppValvePayCheck` | 每小时 | APP 缴费赛恩阀控表开阀 |
| `Taskmain` | `getWTData` | 每天 13:15 | 获取万泰水表数据 |
| `Taskmain` | `getValvePayWT` | 每小时 | 万泰缴费开阀 |
| `AutoPricesController` | `autoPrices` | 每天 01:00 | 自动扣费 |

## 非 Spring 定时任务

| 类 | 机制 | 时间 | 说明 |
| --- | --- | --- | --- |
| `PinCodeManager` | `java.util.Timer` | 每 2 分钟 | 清理过期验证码，不依赖 `@EnableScheduling` |

## 结论

- 采集、开阀、自动扣费这些 Spring 定时任务代码存在。
- 当前默认不生效，除非恢复 `@EnableScheduling`。
- 验证码清理任务会随类加载后独立运行。
