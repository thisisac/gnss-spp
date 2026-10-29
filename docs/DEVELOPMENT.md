# Development / 开发说明

## 中文

开发环境建议为 Windows 10/11、Visual Studio 2019 或更高版本、C++17 和 MATLAB（仅用于绘图）。将 `src/` 文件加入 Visual Studio 项目，链接 `Ws2_32.lib`。调试时先使用文件模式，确认 OEM7 同步、CRC、观测数量和星历数量，再检查 SPP 迭代和结果文件。

当前没有自动化单元测试；手工验证应记录输入文件、参考坐标、编译器版本、配置和输出摘要。

## English

The recommended environment is Windows 10/11, Visual Studio 2019 or later, C++17, and MATLAB for plotting. Add the files under `src/` to a Visual Studio project and link `Ws2_32.lib`. Debug file mode first: verify OEM7 synchronization, CRC, observation counts, and ephemeris counts before inspecting SPP iterations and output files.

There is no automated unit-test suite yet. Record the input file, reference coordinates, compiler version, configuration, and output summary for manual verification.
