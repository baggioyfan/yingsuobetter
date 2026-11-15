# 签名密钥说明

本仓库**不包含**签名密钥 `GoGoGo.jks`。

- **本地构建**：自行放置 `keystore/GoGoGo.jks`（路径被 `.gitignore` 忽略，不会误提交）。
  密钥库口令 / 别名 / 口令可通过 `local.properties` 提供：
  ```
  KEYSTORE_PASSWORD=***
  KEY_ALIAS=***
  KEY_PASSWORD=***
  ```
  缺省回退到项目历史默认值，仅供个人构建使用。
- **CI 构建**：在仓库 Secrets 中配置 `SIGNING_KEYSTORE_BASE64`（jks 的 base64），
  `build.yml` 会在构建前解码到 `keystore/GoGoGo.jks` 后立即使用。
- **为什么**：签名密钥一旦公开，任何人都能签出"同签名"的 APK 覆盖安装到用户设备上。
  把密钥移出公开仓库后，升级链（同签名覆盖）保持不变，但外部无法再伪造同签名包。

> 声明：本项目为非官方 fork。发布物请以 Release 页的 SHA-256 校验值为准。
