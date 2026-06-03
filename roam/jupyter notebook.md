---
tags:
  - 工具
---
# jupyter notebook

**what**：
一款支持实时代码运行、富文本笔记和数据可视化的交互式开发环境；用于解决代码开发和文档分离的问题；通过整合各类语言的内核（python、cpp...）实现与不同编程语言的交互

**what**：IPython
jupyter notebook 的 默认 python 内核 , 增器的 python 交互解释器 ，支持：
1. 魔法命令（ %开头的命令 ，eg：%matplotlib inline）
2. Shell命令（ !开头的命令 ，eg：!pip install）
3. 显示系统（图片、LateX）

**how**：
```bash
!<命令>                      # 运行终端命令
!conda run -n <虚拟环境名>   # 使用conda环境
```

## jupyter配置密码和工作目录

| 路径                                                                  | 说明               |
| ------------------------------------------------------------------- | ---------------- |
| ~/.jupyter/jupyter_notebook_config.py 中的 c.NotebookApp.password     | 密码               |
| ~/.jupyter/jupyter_notebook_config.py 中的 c.NotebookApp.notebook_dir | 工作目录（网页进入时显示的目录） |

## jupyter中使用matplotlib图片不显示

> **问题复现**：
> 虽然我在 jupyter 里使用了 %matplotlib inline ，这将作用于当前 jupyter进程；后续使用 !python... 是在运行外部命令脚本，在独立子进程中进行的；因此无法通过 jupyter 中 plt.show() 的方式显示图片

解决：
```bash
# 代码中使用 plt.savefig() 保存图片
plt.savefig("train_loss.png")

# jupyter notebook中使用 IPython
from IPython.display import Image
Image(filename="train_loss.png")
```

## 本地访问服务器中的jupyter

```bash
# NOTE 通过SSH本地转发端口实现对服务器中的jupyter进行访问
ssh -L 8888:localhost:8888 hubu@111.47.28.118          # 本地转发端口：将111.47.28.118服务器的8888端口映射到本地8888端口
pip install jupyter                                    # 服务器安装 jupyter
jupyter notebook --no-browser --ip=0.0.0.0 --port=8888 # 启动 jupyter
```