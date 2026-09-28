# Sync binaries of ziglang

从 Zig 官网 <https://ziglang.org/download/index.json> 同步 Zig 二进制程序(各平台归档包及源码包),并通过 GitHub Actions 自动发布到本仓库的 Releases.

## 工作原理

同步流程由 `.github/workflows/dosync.yaml` 定义,核心步骤如下:

1. **解析版本并生成下载清单**
   - 拉取官网全量版本清单 `index.json`(对象按版本号组织),未指定版本时取最新稳定版(过滤 `-dev`/`rc` 等非纯数字版本后按 `sort -Vr` 取最大),指定版本时按版本号精确命中.
   - 若解析到预发布版本(`-dev`/`rc`/`beta`/`alpha`)则直接报错退出,避免误发非稳定版.
   - 下载清单取自该版本下每个条目自带的 `tarball` 与 `shasum`(SHA256)字段,格式为 `<sha256>  <文件名>`,下载 URL 由 `https://ziglang.org/download/<版本>/<文件名>` 直接组装,无需写死平台列表,天然适配各版本架构.
2. **并行下载全部二进制包**:基于 `xargs -P` 并发下载(默认 8 路并发,失败自动重试).
3. **校验 SHA256**:用清单中的校验值通过 `sha256sum -c` 逐文件校验,保证文件完整性.
4. **发布到 Releases**:以版本号为 tag(如 `v0.15.2`),上传所有下载文件.

## 触发方式

- **定时触发**:已屏蔽(不再自动执行).
- **手动触发**(`workflow_dispatch`):可在 Actions 页面手动运行,支持以下输入参数:

| 参数 | 说明 | 必填 | 示例 |
| --- | --- | --- | --- |
| `binvern` | 指定版本号(如 `0.15.2`),不填则使用最新稳定版 | 否 | `0.15.2` |

## 产物

每次同步会在 Releases 中生成一个以版本号命名的发行(如 `Zig v0.15.2 Binaries`),包含对应平台的全部文件(`zig-linux-x86_64-0.15.2.tar.xz`、`zig-macos-aarch64-0.15.2.tar.xz` 等),可直接下载使用.

## 目录结构

```
ziglang/
├── .github/workflows/
│   └── dosync.yaml   # 同步工作流定义
├── LICENSE
└── README.md
```
