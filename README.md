llama-server for Android (ARM64, Armv9-A Optimized)

基于[llama.cpp](https://github.com/ggml-org/llama.cpp)构建的 Android 原生可执行文件，专为 Armv9-A 架构 处理器深度优化，纯 CPU 推理，支持 OpenMP 静态链接，开箱即用。

---

✨ 特性

· ✅ 纯 CPU 推理 – 不依赖 GPU、NPU，兼容所有 Android 设备
· ✅ Armv9-A 指令集优化 – 启用 dotprod、i8mm 指令，关闭 SVE（避免兼容性问题）
· ✅ 静态链接 – OpenMP 运行时和 C++ STL 完全静态链接，无需安装额外库
· ✅ 自动构建 – 每 3 天检测上游 bXXXX 格式的新标签，自动编译并发布到 Releases
· ✅ 精简体积 – 仅保留 llama-server 二进制，适合直接部署到终端设备

---

🚀 适用设备

针对以下 Armv9-A 处理器进行了实机调优（基于 Cortex-X2/A710/A510 核心组合），兼容所有 arm64-v8a 设备：

处理器系列 示例型号
骁龙 7+ Gen2 Redmi Note 12 Turbo, 真我 GT Neo5 SE
骁龙 8+ Gen1 小米12S系列, 一加Ace Pro
骁龙 8 Gen1 小米12 Pro, 荣耀 Magic4
骁龙 8 Gen2 小米13, 一加11, 三星 S23
骁龙 8 Gen3 小米14, 一加12, 荣耀 Magic6
天玑 9000 / 9200 / 9300 / 9400 全系列（均为 Armv9-A 架构）

💡 任何支持 Android 15+ 且 CPU 指令集包含 armv9-a、dotprod、i8mm 的设备均可运行。

---

📥 下载

从 [Releases](https://github.com/045200/llama-cpp-Android/releases)页面下载最新发布的 llama-server 文件。

文件名示例：
llama-server-android-arm64-openmp-repack-static-7plus-gen2-armv9

---

🛠️ 使用方法

1. 推送到设备

```bash
adb push llama-server /data/local/tmp/
adb shell chmod +x /data/local/tmp/llama-server
```

2. 准备模型文件

将 GGUF 格式的模型（如 llama-2-7b.Q4_K_M.gguf）推送到设备：

```bash
adb push model.gguf /data/local/tmp/
```

3. 运行服务器

```bash
adb shell
cd /data/local/tmp
./llama-server -m model.gguf -t 4 --host 0.0.0.0 --port 8080
```

参数说明：

· -t：线程数（建议设为 CPU 大核数量，如 4~8）
· --host：监听地址，0.0.0.0 允许外部访问
· --port：服务端口

4. 测试推理

在电脑上打开浏览器访问 http://设备IP:8080，或使用 curl：

```bash
curl http://设备IP:8080/completion -H "Content-Type: application/json" -d '{"prompt":"你好，请介绍一下你自己","n_predict":128}'
```

---

⚙️ 构建细节

本项目通过 GitHub Actions 自动构建，构建配置如下：

项目 说明
NDK r27d
Android API 35 (Android 15)
ABI arm64-v8a
编译标志 -march=armv9-a+nosve+dotprod+i8mm -mtune=cortex-a710
CMake 选项 GGML_OPENMP=ON GGML_CPU_AARCH64_REPACK=ON GGML_ARM_I8MM=ON GGML_ARM_DOTPROD=ON BUILD_SHARED_LIBS=OFF ANDROID_STL=c++_static
链接方式 OpenMP 静态链接（-static-openmp）
构建系统 CMake + Ninja

每次上游[llama.cpp](https://github.com/ggml-org/llama.cpp)发布新的 bXXXX 格式标签时，工作流会自动触发构建并发布到 Releases（保留最近 5 个版本）。

---

⚠️ 注意事项

1. Android 版本要求：最低 Android 15（API 35），因为使用了 Android 15 的 NDK 头文件。低版本设备可能无法运行。
2. 性能提示：
   · 建议在 root 或具有高性能模式 的设备上运行，避免系统降频。
   · 通过 -t 参数控制线程数，过多线程可能导致调度开销增加。
3. OpenMP 线程数：默认使用所有可用 CPU 核心，可通过环境变量 OMP_NUM_THREADS 覆盖。
4. 权限：如果服务器需要监听外部端口，请确保设备已开启网络权限（Android 10+ 默认允许 adb 转发端口）。

---

📝 许可证

本仓库的构建脚本遵循 MIT 许可证。
生成的二进制文件基于[llama.cpp](https://github.com/ggml-org/llama.cpp)构建，其本身遵循 MIT 许可证。

---

🤝 致谢

· [llama.cpp](https://github.com/ggml-org/llama.cpp)项目团队

· GitHub Actions 提供的免费构建资源