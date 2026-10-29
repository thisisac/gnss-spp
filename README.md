# GNSS SPP / 双频 GNSS 单点定位

一个基于 C++ 的双频 GPS/BDS 单点定位（Single Point Positioning, SPP）与单点测速程序。项目来自《卫星导航算法与程序设计》课程设计，包含源码、实验结果和 MATLAB 绘图脚本。个人课程报告未随仓库发布。

This repository contains a C++ dual-frequency GPS/BDS Single Point Positioning (SPP) and standalone velocity-estimation program. It includes the source code, experiment outputs, and MATLAB plotting scripts; the personal course report is intentionally excluded.

## 1. 项目能力 / Capabilities

### 中文

- 解析 NovAtel OEM7 二进制消息，识别同步头、消息长度、Message ID 和 CRC；
- 解码 GPS 与 BDS 广播星历、双频伪距、载波相位、多普勒和接收机位置消息；
- 完成 GPS 时间、儒略日、地固坐标系（ECEF）、大地坐标系（BLH）和 ENU 坐标转换；
- 根据广播星历计算信号发射时刻的卫星位置、速度、钟差和钟漂；
- 使用 Geometry-Free（GF）和 Melbourne-Wübbena（MW）组合探测异常观测值；
- 对电离层、对流层和地球自转等误差进行改正；
- 使用加权最小二乘迭代求解接收机 ECEF 坐标、GPS/BDS 接收机钟差；
- 使用多普勒观测值计算三维速度和接收机钟漂；
- 输出位置、速度、PDOP、精度估计、可见卫星数和 ENU 统计量；
- 使用 `SPPdraw.m` 对结果进行误差和观测条件可视化。

### English

- Decode NovAtel OEM7 binary messages, including synchronization, message length, Message ID, and CRC;
- Decode GPS/BDS broadcast ephemerides and dual-frequency pseudorange, carrier-phase, Doppler, and receiver-position records;
- Convert between GPS time, Julian date, ECEF, geodetic BLH, and local ENU frames;
- Compute satellite position, velocity, clock bias, and clock drift at signal-transmission time;
- Detect abnormal observations with Geometry-Free (GF) and Melbourne-Wübbena (MW) combinations;
- Apply ionospheric, tropospheric, and Earth-rotation corrections;
- Estimate receiver ECEF coordinates and GPS/BDS receiver-clock biases with iterative weighted least squares;
- Estimate three-dimensional velocity and receiver clock drift from Doppler observations;
- Export position, velocity, PDOP, uncertainty, satellite-count, and ENU statistics;
- Visualize positioning errors and observation conditions with `SPPdraw.m`.

## 2. 程序架构 / Architecture

```text
OEM7 binary file or serial/socket stream
                |
                v
      DataAcquisition.cpp
  synchronization / CRC / message dispatch
                |
                +--> observation epoch (EPOCHOBSDATA)
                +--> GPS ephemerides (SATEPHREC[])
                +--> BDS ephemerides (SATEPHREC[])
                |
                v
       DetectOutlier()
       GF/MW continuity checks
                |
                v
  ComputeSatPVTAtSignalTrans()
  satellite PVT and error corrections
                |
                v
             SPP()
  weighted least-squares position and clock
                |
                +--> SPV() Doppler velocity
                |
                v
      POSRES + RESfile.txt
                |
                v
          SPPdraw.m / MATLAB
```

The implementation is organized around the following modules:

| 模块 / Module | 主要职责 / Responsibility |
|---|---|
| `rtklib.h` | Shared constants, GNSS data structures, configuration constants, and function declarations. |
| `DataAcquisition.cpp` | OEM7 byte-stream buffering, message synchronization, CRC validation, and message decoding. |
| `TimeConvert.cpp` | Calendar time, Julian date, GPS week, and seconds-of-week conversions. |
| `CoordConvert.cpp` | ECEF/BLH/ENU coordinate conversions and satellite azimuth/elevation calculations. |
| `SatEphCalculate.cpp` | Ephemeris propagation, signal-transmission-time PVT, outlier detection, SPP, and Doppler velocity. |
| `Martix.cpp` / `Matrix.h` | Matrix operations used by least-squares estimation and covariance calculations. |
| `sockets.h` | Windows socket helpers for real-time input experiments. |
| `main.cpp` | File-mode execution loop, result-file formatting, epoch reset, and optional socket-mode code. |

详细模块说明见 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) / See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for the detailed design.

## 3. 数据流与关键数据结构 / Data Flow and Data Structures

### 中文

每个历元由 `EPOCHOBSDATA` 表示。它包含观测时间、卫星观测数组、卫星 PVT 中间结果、解码得到的参考位置和解码状态。`SATOBSDATA` 保存单颗卫星的 PR、载波相位、多普勒、频点、卫星系统和锁定状态；`SATEPHREC` 保存广播星历；`POSRES` 保存最终位置、速度、钟差、PDOP、协方差和卫星统计量。

解码器把不完整的数据尾部保留到下一次读取，因而可以处理消息跨越读取缓冲区的情况。粗差探测依赖连续历元状态；遇到时间断裂或观测跳变时清理历史记录。SPP 先计算卫星发射时刻 PVT，再构造伪距观测方程迭代更新接收机坐标和钟差，收敛后由多普勒方程求速度。

### English

Each epoch is represented by `EPOCHOBSDATA`. It contains observation time, satellite observations, intermediate satellite-PVT records, decoded reference position, and decoder state. `SATOBSDATA` stores pseudorange, carrier phase, Doppler, frequency, constellation, and lock flags for one satellite. `SATEPHREC` stores broadcast ephemeris, while `POSRES` stores the final position, velocity, clock terms, PDOP, covariance, and satellite statistics.

The decoder preserves an incomplete trailing message for the next read, allowing messages to cross input-buffer boundaries. Outlier detection maintains continuity state across epochs and resets it after a time gap or an observation jump. SPP first computes satellite PVT at transmission time, then iteratively solves pseudorange equations for receiver position and clock bias. Doppler equations are solved for velocity after the position solution converges.

## 4. 安装 / Installation

本项目没有预编译安装包。安装方式是克隆仓库、准备 Visual Studio/MSVC 和 Windows SDK，然后按下面的构建说明建立本地程序。

There is no prebuilt installer. Clone the repository, install Visual Studio/MSVC and the Windows SDK, then follow the build instructions below.

## 5. 构建 / Build

源码使用 Windows API（`windows.h`、Winsock）和 MSVC 安全 C 运行库接口，建议使用 Visual Studio 2019 或更高版本、C++17 标准，并链接 `Ws2_32.lib`。

The source targets Windows/MSVC and uses `windows.h`, Winsock, and MSVC secure C-runtime functions. Use Visual Studio 2019 or later with C++17 and link `Ws2_32.lib`.

当前材料版 `src/main.cpp` 保留了课程代码中的本地输入路径。运行前请改为自己的 OEM7 文件路径，并确保输出目录存在：

The current material version of `src/main.cpp` preserves the original course-project input path. Replace it with an authorized local OEM7 file before running:

```cpp
D:\\viual stidio\\code\\RTKNav\\data\\oem719-202504011900-1.bin
```

## 6. 运行与结果 / Usage and Outputs

程序默认读取文件模式数据并将结果写入 `RESfile.txt`。`results/RESfile.txt` 和 `results/RESsocket.txt` 是已有实验输出，不是运行所需的输入。

The program runs in file mode by default and writes `RESfile.txt`. The files under `results/` are archived experiment outputs, not required input data.

建议的运行顺序 / Recommended workflow:

1. 将授权使用的 OEM7 二进制文件放在本地 `data/` 目录；
2. 修改 `src/main.cpp` 的输入路径；
3. 在 Visual Studio 中构建并运行；
4. 检查 `RESfile.txt` 的历元级定位和测速结果；
5. 使用 `results/SPPdraw.m` 或 MATLAB 脚本进行可视化。

## 7. 文档与可复现性 / Documentation and Reproducibility

详见 [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) 和 [`docs/REPRODUCIBILITY.md`](docs/REPRODUCIBILITY.md)。个人课程报告因包含个人信息，未随仓库发布。

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) and [`docs/REPRODUCIBILITY.md`](docs/REPRODUCIBILITY.md). The personal course report is intentionally excluded because it contains personal information.

原始 GNSS 输入数据没有随项目发布，因此无法保证已有实验结果在任意数据集上逐字节复现。精度与特定数据、参考坐标和配置有关。

The original GNSS binary input is not distributed, so archived experiment values cannot be reproduced byte-for-byte from this repository alone. Accuracy depends on the specific data, reference coordinates, and configuration.

## 8. 项目状态 / Project Status

这是一个课程研究与实验项目，适合学习 GNSS 数据解码、广播星历传播、SPP 建模和结果分析。它不是经过跨平台持续集成验证的生产级导航库。

This is a course and research project for studying GNSS decoding, broadcast-ephemeris propagation, SPP modeling, and result analysis. It is not a production-grade, cross-platform navigation library validated by continuous integration.

## 9. 许可证 / License

MIT License，详见 [`LICENSE`](LICENSE)。发布前请将版权主体替换为你的姓名或组织，并确认报告、图片、数据和第三方代码拥有公开许可。

MIT License. See [`LICENSE`](LICENSE). Replace the copyright holder before publication and confirm that reports, figures, data, and third-party code are redistributable.
