# 南海鲨电控组培训作业 · 张俊宏

> RoboMaster2027 南海鲨电控组招新培训作业
> 提交人：张俊宏（26 级 · 信息与通信工程学院 · 学号 20263008258）

---

## 一、文件清单

| 路径 | 作用 |
|---|---|
| `STM32.ioc` | **CubeMX 工程配置文件**。引脚、时钟树、外设的图形化配置都在这里，重新生成代码时以此为准 |
| `CMakeLists.txt` | CMake 构建脚本。指定工具链、源码路径、编译选项与链接脚本 |
| `CMakeLists_template.txt` | CubeMX 生成 CMakeLists 用的模板（CubeMX 每次重新生成代码时会读它） |
| `STM32F103C8TX_FLASH.ld` | **链接脚本**。规定代码、数据放在 Flash/RAM 的哪些地址（已移除 READONLY 段以便调试） |
| `Core/Src/` | 主要源码目录。`main.c` 是主程序，`gpio.c`/`usart.c` 等是各外设初始化，`stm32f1xx_it.c` 放中断服务函数 |
| `Core/Inc/` | 对应的头文件目录 |
| `Core/Startup/` | 启动汇编文件。芯片复位后最先执行，负责初始化栈、准备 C 运行环境，然后调用 `main()` |
| `Drivers/STM32F1xx_HAL_Driver/` | ST 官方 **HAL 库**源码（我们主要调用它提供的函数） |
| `Drivers/CMSIS/` | ARM Cortex-M 内核相关的定义（寄存器地址、内核函数） |
| `.gitignore` | 排除构建产物、备份文件和本地 IDE 状态的规则 |
| `.cproject` / `.project` / `.settings/` | CubeMX 生成的 Eclipse 工程元数据（备用，CLion 用不到） |
| `.mxproject` | CubeMX 上次生成代码的记录 |

**如何打开**：用 CLion 打开本目录即可（工具链为 arm-none-eabi-gcc + CMake + Ninja）。

**如何编译**：
```bash
cmake --build cmake-build-debug
# 产物：cmake-build-debug/STM32.elf / .hex / .bin
```

**如何烧录**（两条路线任选）：
```bash
# 路线 A：OpenOCD（配合 CLion 调试）
openocd -f <板级配置>.cfg -c "program cmake-build-debug/STM32.elf verify reset exit"

# 路线 B：FlyMcu 串口下载（需 USB-TTL 模块）
# 先在 CMakeLists 里加 objcopy 生成 .hex，再用 FlyMcu 烧 .hex
```

---

## 二、任务完成度

> ⚠️ **待补充**：需对照《验收标准》逐条填写。
> 目前尚未拿到《验收标准》文件，请向学长索取后按标准条目逐条填写。

| # | 实验 | 章节 | 状态 |
|---|---|---|---|
| 1 | 用按键点亮你的第一颗 LED | Ch2 GPIO | ⬜ 未开始 |
| 2 | LED 灯并行（EXTI 中断 + 非阻塞消抖） | Ch3 中断 | ⬜ 未开始 |
| 3 | 电脑与单片机的双向串口通信 | Ch4 UART | ⬜ 未开始 |
| 4 | OLED 显示画面 | Ch6 OLED | ⬜ 未开始 |
| 5 | 定时器中断控制 LED（1Hz，全程无延时） | Ch7 定时器 | ⬜ 未开始 |
| 6 | PWM 控制 SG90 舵机（3 秒 0°→180°→0°） | Ch7 PWM | ⬜ 未开始 |
| 7 | 输入捕获测量 PWM 占空比 | Ch7 输入捕获 | ⬜ 未开始 |
| 8 | ADC 测量模拟信号 | Ch8 ADC | ⬜ 未开始 |
| 9 | RTC 实时时钟（掉电不重置） | Ch9 RTC | ⬜ 未开始 |

**环境准备进度**：

| 环节 | 状态 |
|---|---|
| STM32CubeMX 6.18.1 | ✅ 完成 |
| CLion 2026.2.2 + CMake/Ninja | ✅ 完成 |
| arm-none-eabi-gcc 15.2.1 | ✅ 完成 |
| OpenOCD 0.12.0 + 板级配置 | ✅ 完成 |
| 编译产出 `.elf` / `.hex` / `.bin` | ✅ 验证通过 |
| ST-Link 驱动 + USB 稳定性 | ✅ 完成 |
| Git 仓库 + `.gitignore` | ✅ 完成 |
| **烧录跑通（点亮板载 LED）** | 🔄 **进行中**（SWD 连接待解决） |

---

## 三、项目简介

### 系统整体功能

本工程是南海鲨电控组培训作业的**基础工程模板**，基于 STM32F103C8T6，使用 ST 官方 HAL 库开发。
目前完成了开发环境搭建、时钟配置与编译链路验证，作为后续 9 个实验的统一代码基础。

### 硬件连接方式

| 部分 | 说明 |
|---|---|
| 开发板 | STM32F103C8T6（Blue Pill 类，带板载 Micro-USB 口） |
| 时钟源 | 外部 8MHz 晶振（HSE），经 PLL 倍频至 **72MHz** 系统时钟 |
| 下载调试 | ST-Link V2，通过 **SWD** 四线连接：`SWDIO`→PA13、`SWCLK`→PA14、`GND`→GND、`3V3`→3V3 |
| 调试接口 | OpenOCD 0.12.0（板级配置 `stm32f1.cfg`） |

### 数据流链路

```
源码 (.c)
   │  arm-none-eabi-gcc -c
   ▼
目标文件 (.o)
   │  链接器 + STM32F103C8TX_FLASH.ld
   ▼
固件 (.elf，含地址与调试信息)
   │  GDB → OpenOCD → ST-Link
   ▼
芯片 Flash
   │  复位
   ▼
启动文件 → 初始化 → main()
```

命令行工具链分工：
- **CMake / Ninja** —— 安排编译顺序、管理依赖（自己不编译源码）
- **arm-none-eabi-gcc** —— 交叉编译器，把 `.c` 编译成 ARM 目标文件
- **链接器** —— 合并 `.o` 与库，按链接脚本确定地址，产出 `.elf`
- **OpenOCD** —— GDB Server，把 GDB 请求转成 ST-Link 能执行的操作
- **ST-Link** —— 实体硬件探针，经 SWD 访问芯片

---

## 四、遇到的问题与解决方案

### 1. 工程放在中文路径下，构建失败

- **现象**：工程原在 `D:\文档\Downloads\STM32`，编译报路径相关错误
- **原因**：工具链（arm-none-eabi-gcc / CMake / Ninja）对含中文和空格的路径支持不佳
- **解决**：整体迁移到纯英文路径 `D:\STM32`，问题消失。
  **教训**：开发环境配置文档也明确要求「安装路径要求全部英文，不能出现空格和括号」

### 2. 下载了错误的 ARM 工具链（aarch64 版本）

- **现象**：安装 `aarch64-none-elf` 后编译报错
- **原因**：`aarch64` 是 64 位 ARM 内核（Cortex-A 系列），而 STM32F103 是 **32 位 Cortex-M3**，需要 `arm-none-eabi`
- **解决**：改下载 `arm-none-eabi-gcc 15.2.1`（mingw-w64-i686 版本），并手动把 bin 目录加进用户 PATH

### 3. ST-Link 设备反复掉线

- **现象**：设备管理器能看到 ST-Link，但插上一分钟左右就自己消失
- **根因**：**USB 选择性暂停**。Windows 认为设备空闲就断电挂起省电
- **解决**：
  ```powershell
  powercfg /query SCHEME_CURRENT 2a737441-1930-4402-8d77-b2bebba308a3 48e6b7a6-50f5-4782-a5d4-53bb8f07e226
  # 值为 0x1 表示已启用 → 关闭：
  powercfg /setacvalueindex SCHEME_CURRENT 2a737441-1930-4402-8d77-b2bebba308a3 48e6b7a6-50f5-4782-a5d4-53bb8f07e226 0
  powercfg /setdcvalueindex SCHEME_CURRENT 2a737441-1930-4402-8d77-b2bebba308a3 48e6b7a6-50f5-4782-a5d4-53bb8f07e226 0
  powercfg /setactive SCHEME_CURRENT
  ```
  另需在「设备管理器 → USB 根集线器 → 电源管理」取消勾选「允许计算机关闭此设备以节约电源」

### 4. ST-Link 驱动安装失败

- **现象**：执行讲义附带的 `stlink_winusb_install.bat` 返回 `0x80020000`
- **原因**：该脚本内部用的是老旧的 `dpinst_amd64.exe`，在 Win10/11 上已失效
- **解决**：改用 Windows 自带命令直接安装（驱动就在本地 `D:\OpenOCD\drivers\ST-Link\`）：
  ```powershell
  pnputil /add-driver "D:\OpenOCD\drivers\ST-Link\stlink_dbg_winusb.inf" /install
  # 成功：oem18.inf，提供商 STMicroelectronics，服务 WinUSB
  ```

### 5. ST-Link 经扩展坞报 Code 43（设备描述符请求失败）

- **现象**：设备管理器显示「未知 USB 设备(设备描述符请求失败)」，硬件 ID 变成 `VID_0000&PID_0002`
- **原因**：ST-Link V2 是 USB 1.1 全速设备，接在扩展坞上要经过三级 USB Hub，链路读不出描述符
- **解决**：从扩展坞拔下，**直插机身 USB 口**（优先 USB 2.0 口），中间不经任何 Hub/延长线

### 6. SWD 连不上芯片（`STLINK_JTAG_GET_IDCODE_ERROR`）

- **现象**：
  ```
  Info : STLINK V2J37S7 (API v2) VID:PID 0483:3748   ← ST-Link 已打开
  Info : Target voltage: 3.240000                     ← 板子有电
  Error: STLINK_JTAG_GET_IDCODE_ERROR                 ← 读不到芯片 ID
  ```
- **排查过程**：
  - 降速到 100kHz / 10kHz / 5kHz 全部失败 → **排除速率与时序问题**
  - `reset_config none` / `srst_only` 均失败 → 排除复位线配置
  - 连续 5 次连接 **0/5 成功** → 属「**系统性接线错误**」而非接触不良
    （判定法：偶发成功是接触不良；稳定失败是接错线）
- **当前定位**：`Target voltage` 正常说明 3V3 与 GND 是通的，问题压缩到 **SWDIO / SWCLK 两根线**
- **下一步**：核对接线定义（不同厂家板子的 SWD 排针丝印顺序不同：
  有的是 `3V3 DIO CLK GND`，有的是 `GND CLK DIO 3V3`，必须看板子印的字），
  并确认 BOOT0 跳线帽处于正确位置

---

## 五、备注

- **参考资料**：
  - 《南海鲨电控组 STM32 学习指南 V1.0》（队内讲义，9 个实验出处）
  - KeysKing 的 STM32 教程（B 站 `space.bilibili.com/6100925`）
  - B 站江协科技（接线讲解）
- **提交方式**：GitHub 公开仓库 + 作业收集表登记
