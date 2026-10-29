# GNSS SPP

一个基于 C++ 的双频 GPS/BDS 单点定位（Single Point Positioning, SPP）程序。
项目来源于《卫星导航算法与程序设计》课程设计，包含源码、实验结果、MATLAB 绘图脚本和课程报告。

## 功能

- 解析 NovAtel OEM7 二进制观测数据；
- 解码 GPS/BDS 广播星历、伪距、载波相位和多普勒观测值；
- 计算卫星位置、速度、钟差和钟漂；
- 进行时间系统、坐标系统、地球自转、电离层和对流层相关处理；
- 使用 GF/Melbourne-Wübbena 组合进行粗差探测；
- 使用最小二乘法完成单点定位和单点测速；
- 输出位置、速度、PDOP、钟差、卫星数和精度统计量；
- 使用 MATLAB 脚本对 ENU 误差、卫星数和其他指标进行可视化。

## 目录

```text
src/       C++ 源码
results/   示例结果文件和 MATLAB 绘图脚本
docs/      课程报告（PDF/DOCX）
```

## 构建

源码使用 Windows API（`windows.h`、Winsock）和 MSVC 安全 C 运行库接口，建议使用 Visual Studio 2019 或更高版本、C++17 标准构建。可以将 `src/` 中的 `.cpp` 文件加入一个空的“空项目”后编译，并链接 Winsock 库 `Ws2_32.lib`。

当前材料版入口文件 `src/main.cpp` 保留了原始课程代码中的输入路径：

```cpp
D:\\viual stidio\\code\\RTKNav\\data\\oem719-202504011900-1.bin
```

在运行前请将该路径改为本地 OEM7 数据文件，并确认输出目录存在。该路径来自原始课程材料，不是本项目默认数据集。

## 运行与输出

程序默认将定位结果写入 `RESfile.txt`。`results/` 中的 `RESfile.txt`、`RESsocket.txt` 是已有实验输出，不是程序运行所必需的输入。原始输入二进制观测文件未包含在本仓库中，请使用自己拥有授权的 OEM7 数据。

## 报告与复现实验

`docs/SPP-course-report.pdf` 是项目说明和精度分析报告，`results/SPPdraw.m` 用于绘制结果。报告中给出的精度结论依赖特定观测文件、参考坐标和运行环境，不能直接视为所有数据集上的性能保证。

## 数据与版权说明

仓库不附带原始 GNSS 数据。发布前请确认观测数据、报告中的图片和第三方代码均有公开许可；如需发布为正式开源项目，请将根目录 `LICENSE` 中的版权主体替换为你的姓名或组织。
