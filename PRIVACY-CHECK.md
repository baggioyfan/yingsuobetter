# 发布前隐私与残留自检（PRIVACY-CHECK）

检查对象：`publish/GoGoGo-1.13.3`（171 个跟踪文件，24 MB）与 Release 附件（两个 APK + 源码 zip）。
检查时间：2026-11-15

## 1. 本机残留（全部为"无"）

| 检查项 | 结果 |
|---|---|
| `local.properties`（含本机 SDK 路径） | 未跟踪、目录内也不存在 |
| `.gradle/`、`build/`、`app/build/` 构建产物 | 不存在 |
| `.idea/`、`*.iml`、`.settings/` | 不存在 |
| `*.log`、`*.hprof`、崩溃转储 | 不存在 |
| `git status` | 工作区干净，无未跟踪文件 |
| 仓库体积 | 24 MB，全部为源码 + 上游 `libs/` 二进制依赖 |

## 2. 个人标识字符串（138 个文本文件全量正则扫描）

扫描模式：本机用户名、Windows 用户目录绝对路径、AppData 目录、本地工作区目录名、邮箱、手机号、IPv4、常见访问令牌前缀。

| 结果 | 说明 |
|---|---|
| 本机用户名 / 绝对路径 / AppData / 工作区路径 | **0 命中** |
| 手机号 / 令牌 | **0 命中** |
| 邮箱 | 仅命中**上游作者**的 `zcsexp@gmail.com`（出现在 `CODE_OF_CONDUCT.md`、`SECURITY.md`、`strings.xml`），属上游公开信息，按 GPL 保留原样 |
| IPv4 | 仅 GitHub Actions 版本号误报（`auto-changelog@v4.0.4` 等） |

## 3. 二进制/打包产物抽查

| 文件 | 条目数 | 结果 |
|---|---|---|
| `Go_1.13.3_arm64-v8a_release.apk` | 955 | 无本机路径/用户名 |
| `Go_1.13.3_arm64-v8a_debug.apk` | 959 | 无本机路径/用户名 |
| `GoGoGo-1.13.3-source.zip` | 171 | 无本机路径/用户名 |

## 4. 仓库内确实存在、但属于"上游原物"的文件

| 文件 | 说明 |
|---|---|
| `keystore/GoGoGo.jks` | **已于 v1.13.3 从仓库与全部历史中移除**。历史上它来自上游公开提交，本构建沿用是为了让 1.12.3 能覆盖升级；现在密钥只在构建环境（CI Secret）中存在，仓库里只剩 `keystore/README.md` 说明 |
| `app/libs/*.so`、`BaiduLBS_Android.jar` | 上游附带的百度地图 SDK 二进制，体积主要来源（12 MB + 6.6 MB） |
| `.github/` | 上游的 CI 与模板配置，原样保留 |

## 5. 提交身份

本地提交作者为 `YG <yg@users.noreply.github.com>`（占位身份，不含真实邮箱）。
如需改成你自己的 GitHub 身份，推送前执行：

```bash
git config user.name "你的名字"
git config user.email "你的GitHub邮箱"
git commit --amend --reset-author --no-edit
git tag -f -a v1.13.3 -m "影梭 1.13.3 路径点优化版"
```

## 6. 结论

- 本仓库与两个 APK、源码包中**未发现**本机路径、用户名、账号、令牌等残留
- 无构建产物、无临时文件、无 IDE 配置
- 签名密钥已不在仓库内（见 `keystore/README.md`），构建时由 CI Secret 注入

发布安全。

## 7. 已知暴露面（游客视角同样可见，属"设计取舍"而非隐私泄露）

| 项 | 现状 | 影响 | 处理建议 |
|---|---|---|---|
| 签名密钥 | **已移除**：仓库与 git 历史均不含 jks | 外部无法再签出同证书 APK | 升级链不变（Release 仍用同密钥签出）；CI 通过 `SIGNING_KEYSTORE_BASE64` 注入 |
| `app/build.gradle` 中的百度地图 AK 默认值 | 明文（即原版 APK 内使用的公开 AK） | 该 AK 已随官方 APK 公开，增量风险低；但可被他人复用配额 | 可改为仅从 `local.properties` 读取；或在百度控制台重置 AK |
| `route-loop.patch` | 补丁内含同一 AK | 同上 | 同上 |

> 以上三项都不含个人信息，属于"开源分发必然暴露的构建信息"。真正的隐私项（本机路径、用户名、邮箱、令牌）在本仓库中为零。
