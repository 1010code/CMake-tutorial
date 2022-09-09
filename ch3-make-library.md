# 靜態庫和動態庫的構建
- 建立一个静态库和动态库，提供 HelloFunc 函数供其他程序编程使用，HelloFunc 向终端输出 Hello World 字符串。 
- 安装头文件与共享库。

## 靜態庫和動態庫的區別
- 靜態庫的副檔名一般為 `.a` 或 `.lib`
  - 靜態庫在編譯時會直接整合到目標程序中，編譯成功的可執行文件可獨立運行。
- 動態庫的副檔名依據作業系統分為 `.so` 或 `.dll` 或 `dylib`。
  - 動態庫在編譯時不會放到連接的目標程序中，即可執行文件無法單獨運行。

## 構建實例
以下為該專案的架構，需要寫一個自定義的函式庫。因此實作內容將會放到 `lib` 資料夾下。最後透過 cmake 編譯安裝到系統目錄下。使得在其他專案能夠快速地引入該函式庫並呼叫內容。

```
.
├── build
├── CMakeLists.txt
└── lib
    ├── CMakeLists.txt
    ├── hello.cpp
    └── hello.h
```

首先建立一個標頭檔，hello.h 定義所需的函式名稱。

```cpp
#ifndef HELLO_H
#define Hello_H

void HelloFunc();

#endif
```

接著建立一個 c++ 檔 hello.cpp 實作函式功能。

```cpp
#include "hello.h"
#include <iostream>
void HelloFunc(){
    std::cout << "Hello World" << std::endl;
}
```

在最外層專案新增 cmake 内容。

```cpp
PROJECT(HELLO)
ADD_SUBDIRECTORY(lib bin)
```

在 lib 中CMakeLists.txt中的内容

```cpp
SET(LIBHELLO_SRC hello.cpp)
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
```

### ADD_LIBRARY 指令
- hello：為函式庫名稱，生成的名字前面會加上 lib 前綴。因此最終產生的文件是 `libhello.so` (Linux)。
- SHARED： 動態庫   
- STATIC：靜態庫
- ${LIBHELLO_SRC}： 源文件(以變數的方式呈現)




> 以上內容參考 [ch2-1](./ch2-1/) 程式碼

### 同时构建静态和动态库

```cpp
// 如果用这种方式，只会构建一个动态库，不会构建出静态库，虽然静态库的后缀是.a
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
ADD_LIBRARY(hello STATIC ${LIBHELLO_SRC})

// 修改静态库的名字，这样是可以的，但是我们往往希望他们的名字是相同的，只是后缀不同而已
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})
```

### SET_TARGET_PROPERTIES

这条指令可以用来设置输出的名称，对于动态库，还可以用来指定动态库版本和 API 版本

同时构建静态和动态库

```cpp
SET(LIBHELLO_SRC hello.cpp)

ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})

//对hello_static的重名为hello
SET_TARGET_PROPERTIES(hello_static PROPERTIES  OUTPUT_NAME "hello")
//cmake 在构建一个新的target 时，会尝试清理掉其他使用这个名字的库，因为，在构建 libhello.so 时， 就会清理掉 libhello.a
SET_TARGET_PROPERTIES(hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})

SET_TARGET_PROPERTIES(hello PROPERTIES  OUTPUT_NAME "hello")
SET_TARGET_PROPERTIES(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)

```

### 动态库的版本号

一般动态库都有一个版本号的关联

```cpp
libhello.so.1.2
libhello.so ->libhello.so.1
libhello.so.1->libhello.so.1.2
```

CMakeLists.txt 插入如下

`SET_TARGET_PROPERTIES(hello PROPERTIES VERSION 1.2 SOVERSION 1)`

VERSION 指代动态库版本，SOVERSION 指代 API 版本。

### 安装共享库和头文件

本例中我们将 hello 的共享库安装到<prefix>/lib目录，

将 hello.h 安装到<prefix>/include/hello 目录

```cpp
//文件放到该目录下
INSTALL(FILES hello.h DESTINATION include/hello)

//二进制，静态库，动态库安装都用TARGETS
//ARCHIVE 特指静态库，LIBRARY 特指动态库，RUNTIME 特指可执行目标二进制。
INSTALL(TARGETS hello hello_static LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
```

![](/screenshot/img3-1.png)

注意：

安装的时候，指定一下路径，放到系统下

`cmake -D CMAKE_INSTALL_PREFIX=/usr ..`