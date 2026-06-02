---
tags:
  - cpp
  - boost
---
# 编译boost

```bash
wget https://archives.boost.io/release/1.87.0/source/boost_1_87_0.tar.gz  #官网下载boost库
tar -zxvf boost_1_87_0.tar.gz && cd boost_1_87_0                          #解压
./bootstrap.sh
./b2
sudo ./b2 install
ls /​usr/local/include/boost
```