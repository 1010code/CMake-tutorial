# 讓專案看起來更像一個工程

- 為工程添加一個子目錄 src，用來放置工程源代碼
- 添加一個子目錄 doc，用來放置這個工程的文檔 hello.txt
- 在工程目錄添加文本文件 COPYRIGHT, README
- 在工程目錄添加一個 runhello.sh 腳本，用來調用 hello 二進制
- 將構建後的目標文件放入構建目錄的 bin 子目錄
- 將 doc 目錄 的內容以及 COPYRIGHT/README 安裝到 /usr/share/doc/cmake/

## 將目標文件放入構建目錄的 bin 子目錄
每個目錄下必須要有一個 `CMakeLists.txt` 文件。

```
.
├── build
├── CMakeLists.txt
└── src
    ├── CMakeLists.txt
    └── main.cpp
```

在最外層的 `CMakeLists.txt` 需要給予專案名稱以及添加子目錄，同時還可以指定二進制和目標二進制的生成路徑。以下範例將 src 子目錄加入工程並指定編譯輸出(包含編譯中間結果)路徑為 bin 目錄。如果不進行 bin 目錄的指定，那麼編譯結果(包括中間結果)都將存放在 `build/src` 目錄。

```c
PROJECT(HELLO)
ADD_SUBDIRECTORY(src bin)
```

### ADD_SUBDIRECTORY 指令
- 這個指令用於向當前工程添加存放源文件的子目錄，並可以指定中間二進制和目標二進制存放的位置。
- EXCLUDE_FROM_ALL 函數是將寫的目錄從編譯中排除。
- [Cmake命令之add_subdirectory介绍](https://www.jianshu.com/p/07acea4e86a3)

> ADD_SUBDIRECTORY(source_dir [binary_dir] [EXCLUDE_FROM_ALL])

在 src 下的 `CMakeLists.txt` 則要詳細說明要被執行編譯的文件。

```c
ADD_EXECUTABLE(hello main.cpp)
```

### 更改二進制的保存路徑
SET 指令重新定義 EXECUTABLE_OUTPUT_PATH 和 LIBRARY_OUTPUT_PATH 變量 來指定最終的目標二進制的位置。哪裡要改變目標存放路徑，就在哪裡加入上述的定義，所以應該在 src 下的 `CMakeLists.txt` 下撰寫以下內容：

```c
SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
```

# 安裝

- 一种是从代码编译后直接 make install 安装
- 一种是打包时的指定 目录安装。
    - 简单的可以这样指定目录：make install DESTDIR=/tmp/test
    - 稍微复杂一点可以这样指定目录：./configure –prefix=/usr

## 如何安装HelloWord

使用CMAKE一个新的指令：INSTALL

INSTALL的安装可以包括：二进制、动态库、静态库以及文件、目录、脚本等

使用CMAKE一个新的变量：CMAKE_INSTALL_PREFIX

```c
.
├── build
├── CMakeLists.txt
├── COPYRIGHT
├── doc
│   └── hello.txt
├── README
├── runhello.sh
└── src
    ├── CMakeLists.txt
    └── main.cpp
```

### 安装文件COPYRIGHT和README

INSTALL(FILES COPYRIGHT README DESTINATION share/doc/cmake/)

FILES：文件

DESTINATION：

1、写绝对路径

2、可以写相对路径，相对路径实际路径是：${CMAKE_INSTALL_PREFIX}/<DESTINATION 定义的路径>

CMAKE_INSTALL_PREFIX  默认是在 /usr/local/

cmake -DCMAKE_INSTALL_PREFIX=/usr    在cmake的时候指定CMAKE_INSTALL_PREFIX变量的路径

### 安装脚本runhello.sh

PROGRAMS：非目标文件的可执行程序安装(比如脚本之类)

INSTALL(PROGRAMS runhello.sh DESTINATION bin)

说明：实际安装到的是 /usr/local/bin

### 安装 doc 中的 hello.txt

- 一、是通过在 doc 目录建立CMakeLists.txt ，通过install下的file
- 二、是直接在工程目录通过
    
     INSTALL(DIRECTORY doc/ DESTINATION share/doc/cmake)
    

DIRECTORY 后面连接的是所在 Source 目录的相对路径

注意：abc 和 abc/有很大的区别

目录名不以/结尾：这个目录将被安装为目标路径下的

目录名以/结尾：将这个目录中的内容安装到目标路径

### 安装过程

cmake ..

make

make install