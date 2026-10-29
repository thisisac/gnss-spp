# SPP Architecture / SPP 程序架构

## 1. Scope / 范围

This document describes the implementation that is actually present in `src/`. It is an engineering map for readers who want to understand, build, debug, or extend the project.

本文档以 `src/` 中实际存在的代码为准，说明程序的模块边界、数据流、估计模型、输入输出和已知限制，便于阅读、调试和后续扩展。

## 2. Runtime Pipeline / 运行流程

```text
Read bytes
  -> find OEM7 sync (AA 44 12)
  -> parse header and message length
  -> verify CRC
  -> decode observations and ephemerides
  -> retain incomplete tail for the next read
  -> detect GF/MW outliers
  -> compute satellite PVT at transmission time
  -> apply corrections and build pseudorange equations
  -> iterate weighted least squares for receiver position
  -> solve Doppler equations for velocity
  -> write POSRES fields and text output
```

对应中文流程：

```text
读取字节 -> 查找 OEM7 同步头 -> 解析消息头和长度 -> CRC 校验
-> 解码观测和星历 -> 缓存不完整尾部 -> GF/MW 粗差探测
-> 计算卫星发射时刻 PVT -> 误差改正与方程组构建
-> 最小二乘迭代求位置 -> 多普勒求速度 -> 输出 POSRES 和结果文件
```

## 3. Components / 模块职责

### `DataAcquisition.cpp`

The decoder scans for the OEM7 synchronization sequence, reads the complete header and payload, validates CRC, dispatches by message ID, and copies an incomplete trailing message to the beginning of the buffer. This is the boundary between raw bytes and typed GNSS records.

该模块负责同步头搜索、消息长度解析、CRC 校验、Message ID 分派以及跨缓冲区消息拼接，是“原始字节流”和“结构化 GNSS 数据”之间的边界。

### `TimeConvert.cpp`

This module converts calendar time, Julian date, GPS week, and seconds of week. These conversions support both ephemeris evaluation and result timestamps.

该模块支持历书时间、儒略日、GPS 周和周内秒之间的转换，为星历传播和结果输出提供统一时间基准。

### `CoordConvert.cpp`

The coordinate layer handles WGS-84 ECEF/BLH conversion, local ENU transformation, and azimuth/elevation calculations. Elevation and azimuth are used by observation screening and quality analysis.

该模块处理 WGS-84 椭球上的 ECEF/BLH 转换、ENU 局部坐标转换以及卫星高度角/方位角计算。

### `SatEphCalculate.cpp`

This is the main estimation module. It contains broadcast-ephemeris propagation, signal-transmission-time satellite PVT, outlier detection, pseudorange SPP, accuracy assessment, and Doppler velocity estimation.

这是核心算法模块，包含广播星历传播、信号发射时刻卫星 PVT、粗差探测、伪距 SPP、精度评估和多普勒测速。

### `Martix.cpp` and `Matrix.h`

These files provide matrix storage and operations used by normal-equation formation, inversion, covariance propagation, and least-squares updates. The filename `Martix.cpp` is retained from the original project.

这些文件提供法方程构建、矩阵求逆、协方差传播和最小二乘更新所需的矩阵运算。`Martix.cpp` 的拼写保留原始项目名称。

### `main.cpp`

The entry point opens the input and output files, repeatedly decodes epochs, calls `DetectOutlier`, calls `SPP`, calls `SPV` after a successful position solution, writes one result line, and clears epoch buffers before the next iteration.

主函数负责打开输入/输出文件，循环解码历元，调用 `DetectOutlier` 和 `SPP`，在定位成功后调用 `SPV`，写入结果并清空历元缓存。

## 4. Estimation Model / 解算模型

The position solution estimates receiver ECEF coordinates and constellation clock terms from corrected pseudorange observations. The implementation iterates the linearized observation equations until the coordinate update is sufficiently small or the iteration limit is reached. `POSRES` stores the solution and quality indicators.

定位解算使用改正后的伪距观测值估计接收机 ECEF 坐标和不同卫星系统的钟差。程序通过线性化观测方程迭代更新坐标，直到改正数足够小或达到迭代上限，结果写入 `POSRES`。

For a satellite `s`, the corrected pseudorange model is conceptually:

```text
P_s = geometric_range_s
      + c * receiver_clock_bias
      - c * satellite_clock_bias
      + ionosphere + troposphere + Earth_rotation_terms
```

For a receiver state `x`, the linearized least-squares system is represented as `B * dx = w`. The matrix layer forms the normal equations, solves `dx`, updates the ECEF state, and propagates the covariance used by `POSRES`.

对卫星 `s`，改正后的伪距模型可概括为：

```text
P_s = 几何距离
      + 光速 × 接收机钟差
      - 光速 × 卫星钟差
      + 电离层项 + 对流层项 + 地球自转项
```

对接收机状态 `x`，程序将线性化方程表示为 `B * dx = w`，由矩阵模块构造法方程、求解 `dx`、更新 ECEF 状态，并传播 `POSRES` 使用的协方差。

The velocity solution uses Doppler observations and the satellite velocity/line-of-sight information computed by the PVT layer. The output includes three velocity components and receiver clock drift.

测速解算使用多普勒观测值、卫星速度和视线方向信息，输出三维速度和接收机钟漂。

## 5. Error Handling / 错误处理

- Incomplete input messages are retained for the next read.
- CRC failures advance the search and do not enter the estimation layer.
- Unsupported constellations or signal types are skipped.
- Invalid or discontinuous observations are removed by the outlier layer.
- A failed SPP epoch is not written as a valid solution.

## 6. Known Limitations / 已知限制

1. The original `main.cpp` contains a hard-coded Windows input path.
2. Socket-mode code is present as commented course code and uses Windows-specific networking.
3. No build system or automated numerical regression test is included.
4. Original binary observation files are not included.
5. Exact report numbers depend on the original data and reference coordinates.

1. 原始 `main.cpp` 含有 Windows 本地硬编码输入路径。
2. Socket 模式以课程代码形式保留，并依赖 Windows 网络 API。
3. 当前没有 CMake 构建系统和自动化数值回归测试。
4. 原始二进制观测数据未发布。
5. 报告中的数值依赖原始观测文件和参考坐标。
