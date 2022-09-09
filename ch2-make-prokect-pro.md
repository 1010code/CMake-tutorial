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

### 安裝過程
進入 build 資料夾，並輸入以下指令編譯安裝：

```
cmake ..
make
make install
```

> 以上內容參考 [ch2-1](./ch2-1/) 程式碼

# 安裝
- 一種是從代碼編譯後直接 make install 安裝
- 一種是打包時的指定目錄安裝
    - 簡單的可以這樣指定目錄：make install DESTDIR=/tmp/test
    - 稍微複雜一點可以這樣指定目錄：./configure –prefix=/usr

## 如何安裝 Hello World
- 使用CMAKE一個新的指令：INSTALL
- INSTALL的安裝可以包括：二進制、動態庫、靜態庫以及文件、目錄、腳本等
- 使用CMAKE一個新的變量：CMAKE_INSTALL_PREFIX

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

### 安裝文件 COPYRIGHT 和 README
- FILES：文件
- DESTINATION：
  - 寫絕對路徑
  - 可以寫相對路徑，相對路徑實際路徑是：${CMAKE_INSTALL_PREFIX}/<DESTINATION 定義的路徑>

```
INSTALL(FILES COPYRIGHT README DESTINATION share/doc/cmake/)
```

> CMAKE_INSTALL_PREFIX  預設是在 /usr/local/

cmake -D CMAKE_INSTALL_PREFIX=/usr 是在 cmake 的時候指定 CMAKE_INSTALL_PREFIX 變數的路徑。

### 安裝腳本 runhello.sh

PROGRAMS：非目标文件的可执行程序安装(比如腳本之類)

```
INSTALL(PROGRAMS runhello.sh DESTINATION bin)
```

> 實際安裝到的路徑是 /usr/local/bin

### 安装 doc 中的 hello.txt
- 是通過在 doc 目錄建立 CMakeLists.txt ，通過 install 下的 file

```    
INSTALL(DIRECTORY doc/ DESTINATION share/doc/cmake)
``` 

DIRECTORY 後面連接的是所在 Source 目錄的相對路徑。注意：abc 和 abc/ 有很大的區別，目錄名不以/結尾：這個目錄將被安裝為目標路徑下的，目錄名以/結尾：將這個目錄中的內容安裝到目標路徑。

### 安裝過程
進入 build 資料夾，並輸入以下指令編譯安裝：

```
cmake ..
make
make install
```

> 以上內容參考 [ch2-2](./ch2-2/) 程式碼