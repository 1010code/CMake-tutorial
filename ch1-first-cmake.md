## 前言
cmake 是一個高級編譯配置工具，簡單來說 cmake 就是生成 Makefile 的工具。使用的時機是當專案變得非常複雜有非常多的相依程式，可以透過 cmake 幫助快速輸出一個可執行文件或者共享庫(dll，so等等)。官方網站是 [www.cmake.org](http://www.cmake.org/)，可以透過官方網站獲得更多關於 cmake 的訊息。

## CMake安装
可以到官方網站 [http://www.cmake.org/HTML/Download.htm](http://www.cmake.org/HTML/Download.html) 下載相對應系統版本的安裝檔。

## 第一個 CMake 文件
### 1. 建立 `main.cpp`
首先開啟 VScode 新增一個 `main.cpp`。

```cpp
#include <iostream>

int main(){
std::cout <<  "hello world" << std::endl;
}
```

### 2. 建立 `CMakeLists.txt`
接著新增 `CMakeLists.txt`，並設定編譯剛剛所新增的程式。

- PROJECT 指令
    - 可以用來指定專案的名字
    - <projectname>_BINARY_DIR，本例中是 HELLO_BINARY_DIR
    - <projectname>_SOURCE_DIR，本例中是 HELLO_SOURCE_DIR
- SET 指令
    - 創立和指定變數
    - SRC_LIST 變數就包含了 main.cpp
    - 多檔時可以用空白或分號區隔： SET(SRC_LIST main.cpp t1.cpp t2.cpp)
- MESSAGE 指令
    - 向終端輸出用戶自定義的訊息，共有三種模式：
        - SEND_ERROR：產生錯誤訊息，生成過程被跳過。
        - STATUS：一般文字訊息输出前缀為 —。
        - FATAL_ERROR：立即终止所有 cmake 過程。
- ADD_EXECUTABL 指令
    - 生成可執行文件。

`ADD_EXECUTABLE(hello ${SRC_LIST})` 是生成的可執行文件名是 hello，源文件是讀取SRC_LIST 變數中的內容

```c
PROJECT (HELLO)
SET(SRC_LIST main.cpp)
MESSAGE(STATUS "This is BINARY dir " ${HELLO_BINARY_DIR})
MESSAGE(STATUS "This is SOURCE dir "${HELLO_SOURCE_DIR})
ADD_EXECUTABLE(hello ${SRC_LIST})
```

如果改變了專案名稱，上述兩個變數名也要改變。因此也可以使用預設變數 PROJECT_BINARY_DIR  和 PROJECT_SOURCE_DIR，這兩個變數和 HELLO_BINARY_DIR，HELLO_SOURCE_DIR 是一致的。兩個變數都指向當前的工作目錄，後面外部編譯會再提到使用。

上述例子也可以簡寫成：
```c
PROJECT(HELLO)
ADD_EXECUTABLE(hello main.cpp)
```

> 專案名稱 HELLO 和生成的可執行文件 hello 是沒有任何關係的。

### 3. 生成 makefile
使用 cmake，生成 makefile 文件。目錄下就生成了以下文件： CMakeFiles, CMakeCache.txt, cmake_install.cmake 等文件，其中關鍵為 Makefile 是透過 `CMakeLists.txt` 自動生成的。

```sh
cmake .
```

### 4. make 編譯
使用 make 指令編譯剛剛所產生的 makefile 文件。

```sh
make
```

### 5. 執行
最终生成了 Hello 的可執行程序。

```sh
./hello
```

## 語法的基本原則
- 變數使用 `${}` 方式取值，但是在 IF 控制語句中可直接使用變數名。
- 指令(參數 1 參數 2...)，參數之間使用空格或分號分開。
    - 以上面的 ADD_EXECUTABLE 指令為例，如果存在另外一個 func.cpp 文件就要寫成：
    - `ADD_EXECUTABLE(hello main.cpp func.cpp)` 或 `ADD_EXECUTABLE(hello main.cpp;func.cpp)`
- 指令與大小寫無關的，但建議全部指令以大寫表示。

## 語法注意事項
- `SET(SRC_LIST main.cpp)` 可以寫成 `SET(SRC_LIST "main.cpp")`，如果文件名稱中含有空格，就必須加上雙引號。
- `ADD_EXECUTABLE(hello main)` 副檔名可省略，他會自動去找 `.c` 和 `.cpp`，但建議還是寫清楚附上完整名稱。

# 內部構建和外部構建
上述例子就是內部構建，缺點就是產生的臨時文件特別多，不易管理專案。而外部構建，就會把生成的臨時文件放在 build 目錄下，不會對源文件有任何影響，因此建議使用外部構建方式。

## 外部構建方式舉例
延續剛剛例子，這次使用外部構建方式來編譯文件。首先在當前專案目錄下建立一個 build 資料夾，目的是讓生成文件存放至該處。接著 `cd build` 進入該資料夾後執行 `cmake ..` 產生 makefile 文件。最後在該目錄資料夾下輸入 `make` 指令編譯文件並產生 Hello 的可執行程序。

1. 建立 build 資料夾
2. 終端機進入 build 執行 `cmake ..`
3. 在 build 目錄下執行 `make`

在編譯的過程中可以注意外部構建的兩個變數輸出訊息：
- HELLO_SOURCE_DIR 是專案源路徑 xxx
- HELLO_BINARY_DIR 編譯路徑 xxx/bulid

又或者可以使用以下指令就不用手動新增 build 資料夾並進入。

```sh
# 配置項目
cmake -S . -B build 
# 建構項目
cmake --build build
```
