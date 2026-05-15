# Fast Complex L1-Ball Projection (复数 $\ell_1$ 范数球快速投影算法)

本项目为本科毕业设计《复数一范数球约束投影的快速算法》的官方开源代码库。

本项目提出并实现了一种基于极坐标解耦与动态下界更新的 $\mathcal{O}(N)$ 线性时间投影算法，并利用 AVX2 指令集进行了底层硬件级并行加速。

## 📦 目录结构 (Repository Structure)

本项目主要包含两个独立的核心模块：

- `Numerical_Experiments/`: 包含论文第五章中用于验证算法渐进复杂度和加速比的两组 C++ 数值实验源码。
- `SPGL1_Integration/`: 包含用于无缝替换原生 SPGL1 求解器核心算子的 C-MEX 源码及 MATLAB Wrapper 接口。

## 🛠️ 环境依赖 (Prerequisites)

- **C++ 编译器**: 支持 C++11 及以上标准，且**必须支持 AVX2 指令集**（推荐 GCC 14.1.0 或 MSVC）。
- **MATLAB**: R2018a 或更高版本（需配置好 C/C++ 编译环境，即 `mex -setup C++`）。
- **第三方库 (仅模块 2 需要)**: 原版 [SPGL1 优化求解器](https://github.com/mpf/spgl1)。

---

## 🚀 使用指南 (Usage Instructions)

### Part 1: C++ 数值实验运行 (Numerical Experiments)

该模块包含了多场景数据分布测试（百万级）以及时空开销综合评估（千万级）的代码。代码无其他外部库依赖，可直接编译运行。

1. 进入实验目录：
   ```bash
   cd Numerical_Experiments
   ```

2. 使用开启最高优化 (`-O3`) 和 AVX2 指令集 (`-mavx2` 或 `/arch:AVX2`) 的参数进行编译。例如，在 GCC 下：
   ```bash
   g++ -O3 -mavx2 main.cpp -o experiment_run
   ```

3. 执行编译出的可执行文件：
   ```bash
   ./experiment_run
   ```
   *程序将自动在终端输出各分布场景下的耗时、加速比及峰值内存占用。*

### Part 2: SPGL1 求解器算子替换 (SPGL1 Integration)

为了在实际的基追踪去噪（BPDN）任务中发挥本算法的性能，需要将本仓库的 C-MEX 算子嵌入到 SPGL1 工具箱中。

1. **获取原版 SPGL1**: 请先从 SPGL1 官方仓库下载原版求解器源码。
2. **替换接口文件**:
   将本仓库 `SPGL1_Integration/` 目录下的 `oneProjector.m` 文件复制到原版 SPGL1 源码的 `spgl1/private/` 目录下，**覆盖**原有文件（此操作修复了多态重载下的参数错位异常）。
3. **导入底层 C-MEX 源码**:
   将本仓库中的 `oneProjector_MEX.c`（或 `.cpp`）放入 `spgl1/private/` 目录中。
4. **编译 MEX 内核**:
   打开 MATLAB，将工作区（Current Folder）切换至 `spgl1/private/` 目录，在命令行窗口执行以下编译指令（请根据您的编译器启用 AVX2 加速选项）：
   
   *对于 Windows MSVC 编译器:*
   ```matlab
   mex -O COMPFLAGS="$COMPFLAGS /O2 /fp:fast /arch:AVX2" oneProjector_MEX.c
   ```
   
   *对于 Linux GCC 编译器:*
   ```matlab
   mex -O CFLAGS="$CFLAGS -O3 -mavx2 -ffast-math" oneProjector_MEX.c
   ```

5. **运行验证**:
   编译成功后会生成对应的 `.mexw64` 或 `.mexa64` 文件。此时，您可以正常调用 SPGL1 的所有上层函数（如 `spg_bpdn`），求解器将在底层自动路由至极速版的复数投影算子。

## 📝 引用 (Citation)

如果您在研究中使用了本仓库的代码，请通过以下格式进行引用：

```bibtex
@misc{fast_complex_l1_projection,
  author = {Zhiyan Wang},
  title = {Fast Complex L1-Ball Projection Algorithm},
  year = {2026},
  publisher = {GitHub},
  journal = {GitHub repository},
  howpublished = {\url{[https://github.com/YOUR_USERNAME/Fast-Complex-L1-Projection](https://github.com/YOUR_USERNAME/Fast-Complex-L1-Projection)}}
}
```
*(注：请将链接中的 `YOUR_USERNAME` 替换为您的实际 GitHub 用户名)*
