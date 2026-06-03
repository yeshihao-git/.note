---
tags:
  - python
  - 工具
---
# pip

**what**：
python 的默认包管理器
1. pip 优先识别和安装 二进制分发包（ .whl ）

**what**：二进制分发包 
（ .whl ）已经 构建、编译 好的包，用户无需重新编译，只需 安装
1. 由 setuptools（打包工具）、wheel（专门的 .whl 打包工具）生成
2. 统一托管到 python官方包索引（PyPI，Python Package Index）供开发者下载

## pip install：OSError: [Errno 28] No space left on device

删除 linux 中 ~/ 中的文件（相当于 windows C盘的内容）

## pip安装dgl

1. [新版本](https://www.dgl.ai/pages/start.html) [历史版本](https://conda.anaconda.org/dglteam/linux-64) 下载对应的.tar.bz2包
2. 解压到目标路径：可以通过以下方式查看
```bash
python
import site
print(site.getsitepackages())
```


## pip安装faiss

```bash
pip install faiss-gpu
```

## pip安装yaml

```bash
pip install pyyaml
```

## pip安装timm

```bash
pip install timm          # 失败
conda install timm        # 失败
pip install timm          # 加镜像源 失败
pip install timm==0.4.12  # 成功 速度很快
```

## pip安装torch（带cuda）

1. 查看官网：选择安装方式：wheel(推荐)、conda
	[新版本](https://pytorch.org/get-started/locally/) [老版本](https://pytorch.org/get-started/previous-versions/)
2. 验证是否安装成功
```bash
python -c 'import torch;print(torch.__version__);print(torch.version.cuda)'
```

## pip安装torch-scatter

1. 进入 https://data.pyg.org/whl/ ：这个网站里都是 *pytorch依赖的预编译二进制包*
2. 找到对应版本的 .whl文件
3. pip install 即可
```bash
pip uninstall torch-scatter
pip install torch-scatter --upgrade -f https://data.pyg.org/whl/torch-1.12.1+cu113.html
```

## pip安装sparsesvd

1. 前置安装
```bash
pip install numpy
pip install scipy
pip install cython
```
2. 下载 https://pypi.org/project/sparsesvd/#files 中的压缩包
3. 解压后进入
4. 安装
```bash
python setup.py test
python setup.py install
pip install sparsesvd
```
