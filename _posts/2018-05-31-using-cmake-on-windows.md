---
layout: single
title: 'Using CMake on Windows'
date: 2018-05-31 20:13
comments: true
categories: Development-Environment
tags: windows cmake
---
習慣在Linux下開發之後，臨時要在Windows下快速編譯一個專案覺得好麻煩
(實在不想用肥得要命的Visual Studio)

這時候想到微軟近期的最大發明vscode以及windows 10少數的優點power shell

**或許在Windows下開發能跟Linux差不多**

總之，先下載一些東西吧

# Installization

1. [CMake](https://cmake.org/)
2. [MinGW](http://www.mingw.org/)
3. [VSCode](https://code.visualstudio.com/)

## VSCode
其實就是一個輕量好用的IDE，有許多Plugins
網路上都有許多分享，就不在此多做介紹了

## CMake 安裝
到官方網站下載最新的.msi安裝檔下載即可執行安裝，
其中的安裝步驟會出現：
![cmake_install](http://user-image.logdown.io/user/42991/blog/39591/post/7797952/wbw1dKBWQDyUptQVu0Zh_image01-1.png)
為了方便起見選擇將cmake路徑加到PATH環境變數當中

## MinGW
習慣在Linux開發的人都知道GCC以及G++是多麼方便好用的，因此就有MinGW這個東西產生
>MinGW（Minimalist GNU for Windows），又稱mingw32，是將GCC編譯器和GNU Binutils移植到Win32平台下的產物，包括一系列表頭檔（Win32API）、庫和可執行檔。

下載回來執行後會跳出一個Installization Manager的安裝視窗，
![mingw-installization-manager](https://perso.uclouvain.be/allan.barrea/opencv/_images/mingw_install_1.png)
選擇'mingw32-base'以及'mingw32-gcc-g++'
接下來，
我的電腦->右鍵 內容->進階系統設定->進階->環境變數
找到變數Path並編輯，將MinGW執行檔的路徑新增在變數中
![mingw-path](http://user-image.logdown.io/user/42991/blog/39591/post/7797952/BSveyuoRbaBqKGg6LGhq_mingw-path.PNG)

# Build & Run

以下用一個簡單的程式碼範例示範在Windows下如何用cmake編譯程式碼
```cpp main.cpp
#include<iostream>

int main (int argc, char* argv[]) {
  std::cout<<"Hello CMake"<<std::endl;
  return 0;
}
```

## 工作資料夾
為了不讓編譯出來的檔案造成資料夾的髒亂，習慣建立一個'build'資料夾放編譯的機械碼
Work Dir/
|
|-CMakeLists.txt
|-main.cpp
|-build/

## 編輯CMakeLists.txt
```cmake CMakeLists.txt
cmake_minimum_required (VERSION 2.8)
project(Begin)
add_executable(main main.cpp)
```

## 開啟Powershell
移動到build資料夾下
```bash
cd C:\Path\to\Work Dir\build\
```
用cmake轉譯成makefile檔，並編譯
```bash
cmake -G "MinGW Makefiles" -D CMAKE_CXX_COMPILER=mingw32-g++.exe -D CMAKE_MAKE_PROGRAM=mingw32-make.exe ..
mingw32-make.exe
```
沒有問題的話應該能在`build`資料夾下看到生成的執行檔囉
