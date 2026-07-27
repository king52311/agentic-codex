# Maven 多模块本地联调方法

## 适用场景

适用于一个服务通过 Maven 依赖引用另一个本地模块的项目，例如：

- A 模块独立发布为 jar。
- B 服务通过 Maven 坐标依赖 A。
- 开发阶段希望 B 优先使用本地源码或本地仓库里的 A。

## 推荐方式一：本地 Reactor 聚合

当多个模块分散在不同目录，但需要一起构建时，可以增加一个仅供本地联调使用的聚合 `pom.xml`。

聚合工程应包含：

- 目标服务模块。
- 被依赖的本地源码模块。
- 被依赖模块需要的公共工具模块。

构建时使用：

```bash
mvn -pl <target-module> -am clean package -DskipTests
```

要点：

- `-pl` 指定最终要构建的目标模块。
- `-am` 自动构建目标模块依赖的本地模块。
- 同一 Reactor 内会优先使用本地源码产物，减少反复发私服。

## 推荐方式二：本地 install

如果不想每次都从聚合工程构建，可以先把被依赖模块安装到本地 Maven 仓库：

```bash
mvn -pl <dependency-module-a>,<dependency-module-b> -am clean install -DskipTests
```

然后再构建或启动目标服务：

```bash
mvn -pl <service-module> -am clean package -DskipTests
```

这种方式适合：

- 依赖模块改动不频繁。
- 目标服务需要多次重启。
- 不希望污染共享 snapshot 仓库。

## Snapshot 依赖刷新

目标服务依赖 snapshot 包时，正式构建或发版前建议强制刷新依赖：

```bash
mvn -pl <service-module> -am -U clean package -DskipTests
```

`-U` 用于强制更新 snapshot，避免继续使用本地旧缓存。

## 常见问题

### 本地改了依赖模块，服务仍然像旧逻辑

优先检查：

- 是否重新构建依赖模块。
- IDE 是否仍引用旧 external library。
- 本地 Maven 仓库是否保留旧 snapshot。
- 构建目标服务时是否需要加 `-U`。

### 目标服务构建时报缺少类

不要只怀疑目标依赖没有发布，先按报错类定位所属模块，再把缺失模块补进 Reactor 或执行本地 `install`。

### 本地联调是否等于线上生效

不等于。本地 Reactor 或本地 `install` 只影响本机。测试环境和生产环境仍要按正式发布流程重新构建并部署最终运行服务。
