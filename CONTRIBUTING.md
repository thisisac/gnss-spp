# Contributing / 参与贡献

## 中文

欢迎围绕 GNSS 数据解码、坐标转换、SPP 模型、测速、测试和文档提交改进。

1. 先搜索已有 issue，避免重复工作。
2. 对新功能或 bug 修复创建独立分支，例如 `feature/improve-decoder` 或 `fix/time-conversion`。
3. 保持修改聚焦，不要在同一个 PR 中混入无关格式化。
4. 更新与行为变化相关的 README 或 `docs/` 文档。
5. 提交 PR 时说明输入格式、测试数据、运行环境和结果。
6. 不要提交原始 GNSS 数据、密钥、私有地址或未经授权的第三方材料。

由于项目当前没有自动化构建和数值回归测试，贡献者应在 Windows/MSVC 环境中记录手工验证步骤。

## English

Contributions are welcome for GNSS decoding, coordinate transforms, SPP models, velocity estimation, testing, and documentation.

1. Search existing issues before starting duplicate work.
2. Create a focused branch such as `feature/improve-decoder` or `fix/time-conversion`.
3. Keep each pull request focused and avoid unrelated formatting changes.
4. Update the README or `docs/` when behavior or interfaces change.
5. Describe input format, test data, environment, and observed results in the pull request.
6. Never commit raw GNSS data, credentials, private endpoints, or unlicensed third-party material.

Because the project currently has no automated build or numerical regression suite, contributors should record manual verification steps performed on Windows/MSVC.
