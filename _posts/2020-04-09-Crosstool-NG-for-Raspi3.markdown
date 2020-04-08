---
layout: single
title:  "Crosstool-NG for Raspberry Pi 3 with Ubuntu 18.04 初新者的筆記"
date:   2020-04-09 00:41:00 +0800
categories: Embedded
tags: linux embedded crosstool-ng raspberry-pi
---
一直很想好好摸索Embedded Linux的領域，
只是一直給自己找藉口遲遲沒有好好的學習(玩這個的挫折感實在太重了啦QQ)

總之剛好在設計新的移動型機器人，
目標是希望可以用一個單晶片作為電腦與馬達的溝通橋樑
(看是要試試看ROS 2.0或是做一個Modbus TCP/RTU gateway)

這篇主要是針對Raspberry Pi做一個交叉編譯的Toolchain<small>*的一些小小心得*</small>
(光是搞這個就讓我很頭痛orzzz)

# Enviroment
|    |   Target board (Raspi 3)   | Host PC             |
|:--:|:--------------------------:|---------------------|
| OS | Ubuntu 18.04 Server 32bits | Ubuntu 18.04 64bits |

## Requirements
先在安裝一些必須的套件
```bash
$ sudo apt-get install bison cvs gperf flex texinfo automake \
                       libtool libtool-bin libtool-doc help2man ncurses*
```
其中`gperf`的版本建議是`3.0.4`，就我而言apt直接安裝的版本會遇到問題

## Install Crosstool-NG
在此我是安裝`1.24.0`的版本
這部份照[crosstool-ng](https://crosstool-ng.github.io/docs/install/)的教學文件安裝即可
其中可以分享的是：
設定安裝路徑時的`-prefix=/some/place`可以考慮直接放在`/opt`底下，像是`/opt/crosstool-ng`
好處是不必設定`PATH`環境變數，而且不知道為什麼這樣做他的Completion功能好像比較正常XD

### Basic funcdation of Crosstool-NG
**查看指令**
`$ ct-ng help`

**查看有哪些參數範本**
```bash
$ ct-ng list-samples
...
[G...]   armv6-rpi-linux-gnueabi
[G...]   armv7-rpi2-linux-gnueabihf
[G...]   armv8-rpi3-linux-gnueabihf
...
```

**查看範本環境**
`$ ct-ng show-<範本名稱>`
`$ ct-ng show-armv8-rpi3-linux-gnueabihf # 看armv8-rpi3-linux-gnueabihf環境`

**載入範本**
`$ ct-ng <範本名稱>`
`$ ct-ng armv8-rpi3-linux-gnueabihf`
原先以為載入這個範本就可以使用，但是需要再調整一些參數才行。

**設定Toolchain參數**
`$ ct-ng menuconfig`
這是最痛苦的部份...
有很多設定根本不知道是什麼用途，不小心就連編譯都編不過。
後面分享一些大概知道的參數

**編譯Toolchain**
`$ ct-ng build`
這部份要特別注意不能有環境變數`${LD_LIBRARY_PATH}`
有的話可以`$ unset LD_LIBRARY_PATH`

# Look over Raspberry Pi
我大概反覆編toolchain編好幾次，每編一次就是幾十分鐘過去。
編完發現不能用真的很痛苦
後來才知道
我太懶了喇幹
根本沒有好好看過目標板的資訊跟設定的內容

然後好好看一下我Raspberry Pi的資訊...
```bash
## On target board
pi: $ ld -v              # 看binutils版本
pi: $ cat /proc/cpuinfo  # 看CPU資訊
pi: $ lscpu              # 看CPU資訊
pi: $ uname -r           # 看kernel版本
pi: $ ldd --version      # 看glibc版本
pi: $ gcc -v             # 看gcc版本
```
根據以上資訊好好去`menuconfig`設定Toolchain參數orzzz

# Configure Toolchain
載入rpi3的範本後，需要調整一下。
進`menuconfig`
<pre><span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                  </font></span><span style="background-color:#3465A4"><font color="#EEEEEC"><b>    </b></font></span><span style="background-color:#3465A4"><font color="#FCE94F"><b>P</b></font></span><span style="background-color:#3465A4"><font color="#EEEEEC"><b>aths and misc options  ---&gt;</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">                                                         </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>T</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">arget options  ---&gt;                                                                 </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>T</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">oolchain options  ---&gt;                                                              </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>O</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">perating System  ---&gt;                                                               </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>B</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">inary utilities  ---&gt;                                                               </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>C</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">-library  ---&gt;                                                                      </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>C</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> compiler  ---&gt;                                                                     </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>D</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">ebug facilities  ---&gt;                                                               </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>C</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">ompanion libraries  ---&gt;                                                            </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │</font></span><span style="background-color:#2E3436"><font color="#555753"><b>  </b></font></span>
<span style="background-color:#3465A4"><font color="#06989A">  </font></span><span style="background-color:#D3D7CF"><font color="#EEEEEC"><b>│</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436"> │                      </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>C</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">ompanion tools  ---&gt; </font></span></pre>

### 在`Paths and misc options`裡面

首先可以考慮在家目錄下建立一個存放製作的Toolchain的資料夾，結構如下：
```bash
my-toolchain
├── ct-ng-1.24.0_build
├── src
└── x-tools
```

**設定編譯時下載東西的位置到`my-toolchain/src`**
<pre><span style="background-color:#D3D7CF"><font color="#2E3436">(${HOME}/my-toolchain/src) </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>L</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">ocal tarballs directory</font></span></pre>
內容是
`${HOME}/my-toolchain/src`

**設定編譯檔的存放位置到`my-toolchain/ct-ng-1.24.0_build`**
<pre><span style="background-color:#D3D7CF"><font color="#2E3436">(${HOME}/ct-ng-1.24.0_build) </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>W</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">orking directory</font></span></pre>
內容是
`${HOME}/my-toolchain/ct-ng-1.24.0_build`

**設定Toolchain的存放位置到`my-toolchain/x-tools`**
<pre><span style="background-color:#D3D7CF"><font color="#2E3436">(${HOME}/my-toolchain/x-tools) </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>P</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">refix directory</font></span></pre>
內容是
`${HOME}/my-toolchain/x-tools`
之後就可以在裡面找到交叉編譯的工具

**設定編譯時使用的線程數量(根據HOST的CPU設定)**
<pre><span style="background-color:#D3D7CF"><font color="#2E3436">(8) N</font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>u</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">mber of parallel jobs</font></span></pre>

### 在Target options裡面

<pre><span style="background-color:#D3D7CF"><font color="#2E3436">(-O) </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>T</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">arget CFLAGS</font></span></pre>
內容改成`-O`。這個功能我也不太清楚是什麼，等我了解後再補充說明

在此部份，其他的設定基本上都是OK的
但可以依照Target board去設定 是否使用MMU、位元數等等

### 在Toolchain options裡面
<pre><span style="background-color:#D3D7CF"><font color="#2E3436">(rpi3) </font></span><span style="background-color:#D3D7CF"><font color="#729FCF"><b>T</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">uple&apos;s vendor string</font></span></pre>
可以設定工具中間的名稱，方便辨識。
但這邊要注意的是不能加上`-`
因為他有限制整個工具的名稱的格式，如果加上`-`會導致名稱格式的數量錯誤

### 在Operating System裡面
<pre><span style="background-color:#D3D7CF"><font color="#729FCF"><b>V</b></font></span><span style="background-color:#D3D7CF"><font color="#2E3436">ersion of linux (5.3)  ---&gt;</font></span></pre>
選擇Target board的kernel版本
這邊我搞了很久，因為預設沒有我的Target board的版本(5.3.0)
這部份要自己修改menu將我們要的版本資訊加進去

首先，回到下載的`/crosstool-ng-1.24.0`資料夾裡面
```bash
crosstool-ng-1.24.0
├── packages
├──── linux # 要在這加入預下載的Linux kernel的checksum，後面說明
├── config
├──── versions
└────── linux.in # 編輯這個檔案，將我們要的版本加入Menu裡面
```

首先在`crosstool-ng-1.24.0/packages/linux`裡面
創一個版本的資料夾，裏面需要有兩個檔案`chksum` and `version.desc`，例如
```bash
packages
├── 5.3.0
├──── chksum
└──── version.desc
```
`version.desc`內容是空的
但是在`chksum`裏面就麻煩了
需要針對kernel的檔案去生md5, sha1, sha256, sha512的checksum編碼，確保下載的檔案沒有被動過手腳
所以...(打到這裡好累XDDD)
在`/packages/linux`裡面的`package.desc`可以得知
他會到 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git 去下載所需的kernel
然後我們可以從 https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git 下載我們要的kernel檔`linux-5.3.tar.gz`
下載之後可以使用`Gtkhash`這類的工具(`sudo apt-get install gtkhash`)
生出這個kernel檔的checksum編碼
我的`chksum`的內容如下：
```bash
md5 linux-5.3.tar.xz c99feaade8047339528fb066ec5f8a49                           
sha1 linux-5.3.tar.xz 988855d1fee4eb12495a5b3602e2f3471623ec3e                  
sha256 linux-5.3.tar.xz 78f3c397513cf4ff0f96aa7d09a921d003e08fa97c09e0bb71d88211b40567b2
sha512 linux-5.3.tar.xz 6b5edef47c319a3fa7f6c20a3e0903a5acd89ec75e32dc5f99adcb60c9fe118ea312722d9c3d27e2e3900afa2455afb86e83a8b6bb131009bc79ddbe6fb0595d
md5 linux-5.3.tar.gz 06140dced08642dbd9cdbdecc9fdfbce                           
sha1 linux-5.3.tar.gz 68fd4bbd3ec86aeb80a568061a9c8709633c112e                  
sha256 linux-5.3.tar.gz 69beef77f43b31a81f7b13750a189ab088589c64b79ce0d6d62c3d922ee59c0a
sha512 linux-5.3.tar.gz 64355947f18fb0bc55e8e0804a44768a7773a3a204f6145514a362033a572a8d8252dc29143ea5b75b68207dc7fc22cce7285b25c8dbaee7f3416e766df2e0ba
```
*P.S. 我也搞不懂.tar.xz有沒有用，我隨便生出來的哈哈*

然後編輯`crosstool-ng-1.24.0/config/versions/linux.in`
根據其他版本去新增即可~~~~~

完成之後，`make clean` and `make uninstall`清除安裝的`ct-ng`
然後重新`make` and `make install`安裝
之後開`menuconfig`應該就可以看到新增的版本了

### 在Binary utilities裡面
<pre><span style="background-color:#D3D7CF"><font color="#2E3436">version of binutils (2.30)  ---&gt;</font></span></pre>
選擇Target board的binutils版本

### 在C-Library裡面
<pre><span style="background-color:#D3D7CF"><font color="#2E3436">Version of glibc (2.27)  ---&gt;</font></span></pre>
選擇Target board的glibc版本

### 在C compiler裡面
<pre><span style="background-color:#D3D7CF"><font color="#2E3436">Version of gcc (8.3.0)  ---&gt;</font></span></pre>
選擇Target board的gcc版本

這部份我的Target board的gcc版本是7.5.0，但是我設定8.3.0是沒問題的(不曉得這樣O不OK就是了...)


以上。
是我這個小菜鳥成功的設定，提供參考。如果哪裡敘述有誤請不吝指教~

## 測試
`$ ct-ng build`
沒意外編成功後
到`${HOME}/my-toolchain/x-tools/bin`可以看到工具
寫一個簡單的.c檔，用工具編譯他
```bash
$ ./armv8-rpi3-linux-gnueabihf-gcc hello.c -o hello
```
然後在把編譯完的檔案丟到目標板執行看看
```bash
## On Host PC
$ scp hello ubuntu@10.0.0.1:/home/ubuntu
```
```bash
## On Target board
ubuntu@ubuntu:~$ ./hello-pi3 
hello, world
```
如果在Target board上執行出現No such file or directory
大概就是Toolchain設定有錯誤不能使用
這時候再好好檢查一下參數是否都有設定正確

## References:
[1] [crosstool-NG](https://crosstool-ng.github.io/)
[2] [Crosstool-NG 紀錄](https://www.eebreakdown.com/2015/05/crosstool-ng_13.html?showComment=1586369551960#c8221321082422471883)
[3] [crosstool-ng详解](https://www.crifan.com/files/doc/docbook/crosstool_ng/release/htmls/)
[4] [GCC中-march、-mtune、-mcpu三个参数的设置](https://gaomf.cn/2016/06/15/GCC%E4%B8%AD-march%E3%80%81-mtune%E3%80%81-mcpu%E4%B8%89%E4%B8%AA%E5%8F%82%E6%95%B0%E7%9A%84%E8%AE%BE%E7%BD%AE/)
