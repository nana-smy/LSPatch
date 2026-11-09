# LSPatch — Android 15 / 16 / 17 (API 35 / 36 / 37) 支持说明

本 fork 是 `nana-smy/LSPatch`，在 `LSPosed/LSPatch` 的基础上完成了高版本 Android 适配。

## 为什么换基线

上游 `LSPosed/LSPatch` 已于 2023-12-13 被归档（archived），最高只支持到 API 34（Android 14），
后续没有维护。直接在归档树上做高版本适配缺少必要的新版 ART/LSPlant 支持。

因此本 fork 以 **JingMatrix/LSPatch**（Vector 分支）的树作为基线 —— 这是 LSPosed 停更后实际被维护的
分支，已经把 `targetSdk` 提到 36、`compileSdk` 提到 37，并包含适配新版 ART 的 LSPlant/hook 引擎实现。
在此基础上再把 `targetSdk` 推到 37（Android 17）。

## 版本矩阵

| 项 | 值 |
|---|---|
| `minSdk` | 28（Android 9） |
| `targetSdk` | **37**（Android 17） |
| `compileSdk` | 37（`platformBuildVersionCode=37`，`compileSdkVersionCodename=17`） |
| `compileSdkVersion` | 37.0 |
| `build-tools` | 37.0.0 |
| `NDK` | **29.0.14206865**（原 29.0.13113456） |
| Gradle | 9.6.1 |
| JDK | 21 |
| CMake | 3.31.6 |

## 关键改动

1. `build.gradle.kts`
   - `androidTargetSdkVersion`：`36` → `37`
   - `androidCompileNdkVersion`：`"29.0.13113456"` → `"29.0.14206865"`
   - `abiFilters`：本次 CI 构建限定为 `arm64-v8a`（构建机磁盘预算所限）。
     **正式发版请恢复** `abiFilters("arm64-v8a", "armeabi-v7a", "x86", "x86_64")`。

2. 16KB 页对齐（Android 15+ 强制要求）
   - NDK 29 默认即 16KB 对齐。已用 `readelf -lW` 验证产物：

     ```
     assets/lspatch/so/arm64-v8a/liblspatch.so   Align 0x4000
     lib/arm64-v8a/libandroidx.graphics.path.so  Align 0x4000
     ```

3. 高版本 ART 兼容（来自 Vector 基线）
   - `core/daemon/.../Workarounds.kt` 中针对 `Build.VERSION_CODES.VANILLA_ICE_CREAM..36`
     区间做了专门处理（Android 15/16 的 ART 行为差异）。
   - `manager/.../LSPackageManager.kt` 使用了 API 35+ 的
     `PackageManager.MATCH_ARCHIVED_PACKAGES` / `PackageInfoFlags`。
   - LSPlant 提供了新版 ART 的 `ArtMethod`/镜像层实现，随 NDK 29 一起编译。

## 构建

```bash
export ANDROID_HOME=/opt/android-sdk
export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64
echo "sdk.dir=$ANDROID_HOME" > local.properties

gradle :patch-loader:assembleRelease     # 原生库
gradle :manager:assembleRelease          # 主 APK
```

产物：
- `manager/build/outputs/apk/release/manager-release.apk`
- `patch-loader/build/intermediates/stripped_native_libs/release/stripReleaseDebugSymbols/out/lib/arm64-v8a/liblspatch.so`

## 已知限制

- 本 fork 的 `abiFilters` 目前只编 `arm64-v8a`。x86/x86_64（模拟器）需要恢复完整 ABI 列表后重新构建。
- 未做真机测试：Android 15/16/17 真机上的注入、隐藏 API 绕过与 SELinux 行为需要设备实测确认。
