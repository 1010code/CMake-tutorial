# 4. 使用外部共享庫和頭文件
在第三節中我們已經將自定義函式庫編譯並安裝到系統目錄下，接著在本章捷中我們將新建一個專案來使用外部共享庫和標頭檔。

```
.
├── build
├── CMakeLists.txt
└── src
    ├── CMakeLists.txt
    └── main.cpp
```

main.cpp

```cpp
#include <hello.h>

int main(){
	HelloFunc();
}
```

在最外層 `CMakeLists.txt` 建立：
```
PROJECT(HELLO)
ADD_SUBDIRECTORY(lib bin)
```

另外在 src 目錄下 `CMakeLists.txt` 建立：

```c
# Linux 方法
INCLUDE_DIRECTORIES(/usr/include/hello)
ADD_EXECUTABLE(hello main.cpp)

TARGET_LINK_LIBRARIES(hello libhello.so)
```

```c
# macOS 方法一
INCLUDE_DIRECTORIES(/usr/local/include/hello)
ADD_EXECUTABLE(hello main.cpp)

FIND_LIBRARY(HELLO_LIBRARY hello HINTS /usr/local/lib)
TARGET_LINK_LIBRARIES(hello PUBLIC ${HELLO_LIBRARY})
```

```c
# macOS 方法二
INCLUDE_DIRECTORIES(/usr/local/include/hello)
LINK_DIRECTORIES(/usr/local/lib)
ADD_EXECUTABLE(hello main.cpp)
TARGET_LINK_LIBRARIES(hello libhello.dylib)
```

### 解決 make 標頭檔找不到的問題
INCLUDE_DIRECTORIES 這個指令可以用來向專案添加多個特定的頭文件搜索路徑，路徑之間用空格分割。在 res 資料夾下的 `CMakeLists.txt` 中加標頭檔的路徑。

```c
INCLUDE_DIRECTORIES(/usr/local/include/hello)
```

> `include <hello/hello.h>`  這樣 include 也可以。

### 解決：找到引用的函數問題

若出現此錯誤訊息： `undefined reference to HelloFunc()`，則表示在編譯過程中沒有正確被連結到動態庫。可以使用 `LINK_DIRECTORIES` 指令添加非標準的共享庫搜索路徑。
 
```c
LINK_DIRECTORIES(/usr/local/lib)
```

另外再透過 `TARGET_LINK_LIBRARIES` 指令添加需要鏈接的共享庫(只需要給出動態鏈接庫的名字就行)。mac: dylib; Linux: so。

```c
TARGET_LINK_LIBRARIES(hello libhello.dylib)
```

> 注意：在 CMakeLists.txt 中插入鏈接共享庫，主要要插在 executable 的後面。

Linux 用 `ldd` 查看編譯過後的鏈接狀況：

```sh
ldd hello
```

macOS 用戶：
```sh
otool -L hello
```


鏈接靜態庫的寫法：
```c
TARGET_LINK_LIBRARIES(hello libhello.a)
```

### 特殊的环境变量 CMAKE_INCLUDE_PATH 和 CMAKE_LIBRARY_PATH

注意：这两个是环境变量而不是 cmake 变量，可以在linux的bash中进行设置

我们上面例子中使用了绝对路径INCLUDE_DIRECTORIES(/usr/include/hello)来指明include路径的位置

我们还可以使用另外一种方式，使用环境变量export CMAKE_INCLUDE_PATH=/usr/include/hello

补充：生产debug版本的方法：
cmake .. -DCMAKE_BUILD_TYPE=debug


## Reference
- [從零開始詳細介紹CMake](https://www.bilibili.com/video/BV1vR4y1u77h/?p=4&spm_id_from=pageDriver&vd_source=5a6b197f885be2e93405a9e839601280)
- [learning-cmake](https://github.com/Akagi201/learning-cmake)