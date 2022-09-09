# 3.1 靜態庫和動態庫的構建
- 建立一個靜態庫和動態庫，提供 HelloFunc 函數供其他程序編程使用，HelloFunc 向終端輸出 Hello World 字符串。 
- 安裝標頭檔與共享庫。

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


## 安裝過程
進入 build 資料夾，並輸入以下指令編譯安裝：

```
cmake ..
make
make install
```

> 以上內容參考 [ch3-1](./ch3-1/) 程式碼

# 3.2 同時構建靜態和動態庫
必須要在 `lib` 資料夾下對 cmake 文件進行修改，但該如何改呢？

```cpp
// 如果用這種方式，只會構建一個動態庫，不會構建出靜態庫，雖然靜態庫的後綴是.a
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
ADD_LIBRARY(hello STATIC ${LIBHELLO_SRC})

// 修改靜態庫的名字，這樣是可以的，但是我們往往希望他們的名字是相同的，只是後綴不同而已
ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})
```

以上的指令都無法解決同時安裝建靜態和動態庫，解決辦法是使用 `SET_TARGET_PROPERTIES`。

### SET_TARGET_PROPERTIES 指令
這個指令可以用來設置輸出的名稱，對於動態庫，還可以用來指定動態庫版本和 API 版本，同時構建靜態和動態庫。

```cpp
SET(LIBHELLO_SRC hello.cpp)

ADD_LIBRARY(hello_static STATIC ${LIBHELLO_SRC})
// 對 hello_static 重名為 hello
SET_TARGET_PROPERTIES(hello_static PROPERTIES  OUTPUT_NAME "hello")
// cmake 在構建一個新的 target 時，會嘗試清理掉其他使用這個名字的庫，因為，在構建 libhello.so 時， 就會清理掉 libhello.a
SET_TARGET_PROPERTIES(hello_static PROPERTIES CLEAN_DIRECT_OUTPUT 1)

ADD_LIBRARY(hello SHARED ${LIBHELLO_SRC})
SET_TARGET_PROPERTIES(hello PROPERTIES  OUTPUT_NAME "hello")
SET_TARGET_PROPERTIES(hello PROPERTIES CLEAN_DIRECT_OUTPUT 1)
```

### 動態庫的版本號
一般動態庫都有一個版本號的關聯。

```
libhello.so.1.2
libhello.so ->libhello.so.1
libhello.so.1->libhello.so.1.2
```

因此最後在 CMakeLists.txt 插入如下：

```
SET_TARGET_PROPERTIES(hello PROPERTIES VERSION 1.2 SOVERSION 1)
```

> VERSION 指代動態庫版本，SOVERSION 指代 API 版本。

### 安裝共享庫和標頭檔
本例中我們將 hello 的共享庫安裝到 `<prefix>/lib` 目錄，以及將 `hello.h` 安裝到`<prefix>/include/hello` 目錄。

```c
// 文件放到該目錄下
INSTALL(FILES hello.h DESTINATION include/hello)

// 二進制，靜態庫，動態庫安裝都用 TARGETS
// ARCHIVE 指靜態庫，LIBRARY 指動態庫，RUNTIME 指可執行目標二進制
INSTALL(TARGETS hello hello_static LIBRARY DESTINATION lib ARCHIVE DESTINATION lib)
```

![](/screenshot/img3-1.png)

## 安裝過程
進入 build 資料夾，並輸入以下指令編譯安裝：

```
cmake -D CMAKE_INSTALL_PREFIX=/usr/local ..
make
make install
```

> 安裝的時候，指定一下路徑，放到系統下。
- Installing: /usr/local/include/hello/hello.h
- Installing: /usr/local/lib/libhello.dylib
- Installing: /usr/local/lib/libhello.a

> 以上內容參考 [ch3-1](./ch3-1/) 程式碼