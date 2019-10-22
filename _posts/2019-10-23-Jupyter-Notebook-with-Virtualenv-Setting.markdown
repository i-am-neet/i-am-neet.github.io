---
layout: post
title:  "Jupyter Notebook with Virtualenv Setting"
date:   2019-10-23 02:10:10 +0800
categories: jekyll update
---
因為最近在學習DRL使用Virtualenv建立一個學習的虛擬環境。
學習的過程中發現，用Jupyter Notebook來建立自己的DRL學習筆記似乎是一個很棒的決定。

正當我裝完Jupyter要開始我的學習之旅發現
乾 怎麼光是`import torch`就跑不起來

請教Google之後才知道要針對所建立的Virtualenv建立一個kernel

# Installization
安裝ipykernel，稍候使用此工具為ipython建立Virtualenv的kernel
```bash
$ pip install --user ipykernel
```

# Install kernel
對虛擬環境`myenv`建立kernel
```bash
$ python -m ipykernel install --user --name=myenv
Installed kernelspec myenv in /home/user/.local/share/jupyter/kernels/myenv
```
可以在終端顯示的路徑中看到kernel.json的檔案

這邊遇到一個小問題分享一下：
因為我把所有環境都放在一個資料夾下，類似
```bash
virtualenv/
└── myenv
└── myenv02
```
我很懶得就想在`--name`後面直接接路徑
```bash
$ python -m ipykernel install --user --name=/home/user/virtualenv/myenv
```
結果是不行的！

要乖乖的到myenv上一層操作

# Verify and Uninstall
確認jupyter有哪些kernel
```bash
$ jupyter kernelspec list
```

刪除kernel
```bash
$ jupyter kernelspec uninstall myenv
```
