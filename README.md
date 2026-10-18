# Screenshot（反禁用截屏录屏）

强制允许截屏和录屏的 LSPosed / Vector 模块，通过 hook 移除 Android 的 `FLAG_SECURE` 截图保护。基于 libxposed API 102，纯 Kotlin。

## 功能特性

- 移除 `Window` 的 `FLAG_SECURE` 标志，让应用无法禁止截屏 / 录屏；
- 强制 `SurfaceView.setSecure(false)`；
- Hook `WindowManagerGlobal` 的 `addView` / `updateViewLayout`，清除 `LayoutParams` 的 `FLAG_SECURE`；
- Hook system_server 的 `WindowState.isSecureLocked`，从系统层面强制窗口判定为非安全；
- 低版本的 `WindowManagerService.isSecureLocked` 一并 hook，高版本静默跳过。

## 工作原理

| Hook | 目标 | 作用 |
| --- | --- | --- |
| `Window.setFlags(int, int)` | 应用进程 | 清除 `FLAG_SECURE` |
| `Window.addFlags(int)` | 应用进程 | 清除 `FLAG_SECURE` |
| `SurfaceView.setSecure(boolean)` | 应用进程 | 强制 `false` |
| `WindowManagerGlobal.addView(...)` | 应用进程 | 清除 `LayoutParams.flags` 的 `FLAG_SECURE` |
| `WindowManagerGlobal.updateViewLayout(...)` | 应用进程 | 清除 `LayoutParams.flags` 的 `FLAG_SECURE` |
| `WindowState.isSecureLocked()` | system_server | 返回 `false` |

作用域：`system`（system_server 进程）。

## 使用

1. 安装支持 libxposed API 102 的框架（Vector，或新版 LSPosed）；
2. 安装本模块，在框架中启用并勾选作用域 `system`；
3. 重启目标应用后，即可对原本禁止截屏的应用正常截屏 / 录屏。

## 构建

环境要求：JDK 21、Android SDK 37（`build-tools 37.0.0`）、Gradle（使用仓库内 wrapper）。

Debug 包（无需签名）：

```bash
./gradlew assembleDebug
```

Release 包（需要 `app/signing.properties`，`release.keystore` 与 `signing.properties` 均被 `.gitignore` 忽略，不提交）：

```properties
STORE_FILE=release.keystore
STORE_PASSWORD=<你的密码>
KEY_ALIAS=<别名>
KEY_PASSWORD=<密钥密码>
```

```bash
./gradlew assembleRelease
```

## 免责声明

本项目仅用于解除你自己设备上的截屏限制，请遵守当地法律法规与目标应用的服务条款，勿用于窃取他人隐私或侵权内容。由此产生的一切后果由使用者自行承担。
