# 宿主机本地私密开发环境配置 (DO NOT COMMIT TO GIT)

## 1. 本地交叉编译链与环境变量 (Toolchain)
- **目标工具链路径**：当前机器的交叉编译器位于以下路径，编译当前项目时，请确保使用这些变量：
  - `CC="/opt/toolchains/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-gcc"`
  - `CXX="/opt/toolchains/arm-linux-gnueabihf/bin/arm-linux-gnueabihf-g++"`
  - `SYSROOT="/opt/toolchains/arm-linux-gnueabihf/arm-linux-gnueabihf/sysroot"`
- **CMake 交叉编译提示**：如果使用 CMake 离线构建，请优先通过指定我本地的工具链文件进行构建，命令示范：
  - `cmake -DCMAKE_TOOLCHAIN_FILE=../toolchain.cmake -B build`

## 2. 本地硬件调试接口与测试 (Hardware & Debug)
- **开发板连接信息**：当前宿主机与开发板的通信接口如下，运行或编写调试脚本时请严格以此为准：
  - **调试串口 (Serial Port)**：`/dev/ttyUSB0` (波特率: `115200`)
  - **开发板网络 IP (Target IP)**：`11.22.33.44` (用户名: `root`，无密码)
- **自动化挂载与拷贝流程**：
  - 当我要求你“将编译产物发送到板子”时，请自动使用 `scp` 命令将 `build/` 或输出目录下的二进制文件拷贝至板端：
    `scp ./build/my_app root@11.22.33.44:/opt/app/`
  - 拷贝完成后，通过 `ssh root@11.22.33.44 "chmod +x /opt/app/my_app"` 赋予执行权限。

## 3. 个人高频效率宏与个性化命令 (Custom Workflow)
- **快捷编译并推送**：当我输入口令 `deploy` 时，代表你需要连续执行以下三个动作：
  1. 在本地执行当前项目的 `make` 或 `cmake` 编译命令。
  2. 编译成功后，自动通过上面的 `scp` 将产物推送到开发板。
  3. 通过 `ssh` 远程启动板子上的程序并观察前 5 秒的日志，若报错则立刻中断并向我汇报。
- **GDB 远程调试联动**：当需要进行源码级调试时，引导我执行本地的 `arm-linux-gnueabihf-gdb`，并配合板端的 `gdbserver :1234 /opt/app/my_app`。
