# 两种喂数据的方法

*Updated 2025-09-19 00:42 GMT+8*  
 *Compiled by Hongfei Yan (2025 Summer)*    
https://github.com/GMyhf/2025fall-cs101/



在 **mac** 或 **linux** 系统上，如果测试数据所在目录下包含形如 `xx.in`、`xx.out` 的文件，而待测试的代码文件是 `e1.py`，假设 `testing_code.py` 位于上两级目录中，脚本解释器为 `python3`，则可以在终端执行：

```bash
python3 ../../testing_code.py e1.py
```

若程序运行正确（如 `ac1.py`），则不会有输出；如果有错误，会直接打印提示信息。

<img src="https://raw.githubusercontent.com/GMyhf/img/main/img/202509190043562.png" alt="5932455ec86aaf5923e3d5080b76085d" style="zoom:50%;" />



另一种方式是使用评测脚本 **offlinejudge.zsh**。
 在 macOS 下执行：

```bash
zsh ../../offlinejudge.zsh e1.py ./
```

在 Linux 下执行：

```bash
bash ../../offlinejudge.zsh e1.py ./
```

运行后，左侧显示 `e1.py` 的程序输出，右侧显示正确答案，方便对照。

<img src="https://raw.githubusercontent.com/GMyhf/img/main/img/202509190044067.jpg" alt="d96c73937410a3f906c294cb92801fb4" style="zoom:50%;" />



综上，以上是通过“喂数据”来测试 Python 程序的两种方法。
 每个题目的测试数据通常以 zip 文件提供，解压后可得到类似 `0.in`、`0.out` 的数据文件对。
 相关脚本 [`testing_code.py`](https://github.com/GMyhf/2025fall-cs101/tree/main/code)、[`offlinejudge.zsh`](https://github.com/GMyhf/2025fall-cs101/tree/main/code) 也可从仓库中下载。
