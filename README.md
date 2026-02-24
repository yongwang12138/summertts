# SummerTTS

本项目基于 [SummerTTS](https://github.com/huakunyang/SummerTTS) 进行Windows平台适配和动态库封装，为Windows开发者提供方便的语音合成解决方案。


## 🌟 项目特点

- **Windows原生支持**：完整适配Windows平台，解决原项目在Windows上的编译和运行问题
- **动态库封装**：提供标准的动态链接库(DLL)接口，方便各种编程语言调用
- **零外部依赖**：继承原项目的优点，编译后无额外依赖，开箱即用
- **中文/英文支持**：支持中文或者英文的语音合成
- **多说话人选择**：支持单说话人和多说话人模型切换


## 📦 功能概述

SummerTTS是一个完全本地运行的语音合成引擎，基于vits算法实现，使用Eigen进行神经网络计算，不依赖PyTorch、TensorFlow等深度学习框架。本项目在保持原项目核心功能的基础上，针对Windows平台进行了优化。

## 🚀 使用说明

### 快速测试用例
通过命令行执行以下指令，即可完成语音合成测试：
```bash
# 语法：[可执行程序] [输入文本文件路径] [模型文件路径] [输出音频文件路径]
summertts_test.exe ../test.txt ../models/single_speaker_fast.bin out.wav
```

# License
- 本项目适用MIT License，基于本项目的开发，使用人员或机构，请遵循 MIT License: https://mit-license.org
