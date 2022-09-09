# 4. 使用外部共享庫和頭文件
在第三節終我們已經將自定義函式庫編譯並安裝到系統目錄下，接著在本章捷中我們將新建一個專案來使用外部共享庫和標頭檔。

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

### 解决：make后头文件找不到的问题

PS：include <hello/hello.h>  这样include是可以，这么做的话，就没啥好讲的了

关键字：INCLUDE_DIRECTORIES    这条指令可以用来向工程添加多个特定的头文件搜索路径，路径之间用空格分割

在CMakeLists.txt中加入头文件搜索路径

INCLUDE_DIRECTORIES(/usr/include/hello)

感谢：

网友：zcc720的提醒

### 解决：找到引用的函数问题

报错信息：undefined reference to `HelloFunc()'

关键字：LINK_DIRECTORIES     添加非标准的共享库搜索路径

指定第三方库所在路径，LINK_DIRECTORIES(/home/myproject/libs)

关键字：TARGET_LINK_LIBRARIES    添加需要链接的共享库

TARGET_LINK_LIBRARIES的时候，只需要给出动态链接库的名字就行了。

在CMakeLists.txt中插入链接共享库，主要要插在executable的后面

查看main的链接情况

```cpp
[root@MiWiFi-R4CM-srv bin]# ldd main 
	linux-vdso.so.1 =>  (0x00007ffedfda4000)
	libhello.so => /lib64/libhello.so (0x00007f41c0d8f000)
	libstdc++.so.6 => /lib64/libstdc++.so.6 (0x00007f41c0874000)
	libm.so.6 => /lib64/libm.so.6 (0x00007f41c0572000)
	libgcc_s.so.1 => /lib64/libgcc_s.so.1 (0x00007f41c035c000)
	libc.so.6 => /lib64/libc.so.6 (0x00007f41bff8e000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f41c0b7c000)
```

链接静态库

`TARGET_LINK_LIBRARIES(hello libhello.a)`

### 特殊的环境变量 CMAKE_INCLUDE_PATH 和 CMAKE_LIBRARY_PATH

注意：这两个是环境变量而不是 cmake 变量，可以在linux的bash中进行设置

我们上面例子中使用了绝对路径INCLUDE_DIRECTORIES(/usr/include/hello)来指明include路径的位置

我们还可以使用另外一种方式，使用环境变量export CMAKE_INCLUDE_PATH=/usr/include/hello

补充：生产debug版本的方法：
cmake .. -DCMAKE_BUILD_TYPE=debug

# 本⼈所有视频和笔记都是免费分享给⼤家的，制作视频和笔记要花费⼤量的时间成本
我也有⽼婆和孩⼦要养，恳求各位观众⽼爷们有经济实⼒的稍微打赏⼀下⼩弟，但不强求，再⼀次感谢。
您的打赏，会让我今后有更⼤的动⼒，做出更优质的视频，感谢⼤家的⽀持

![Untitled](CMake%204f0e9a3fcf9949adab4540191610cf3e/Untitled.png)


## Reference
- [從零開始詳細介紹CMake](https://www.bilibili.com/video/BV1vR4y1u77h/?p=4&spm_id_from=pageDriver&vd_source=5a6b197f885be2e93405a9e839601280)
- [learning-cmake](https://github.com/Akagi201/learning-cmake)