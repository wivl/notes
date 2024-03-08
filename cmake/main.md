# CMake 教程

## 静态链接库

### 定义静态库

```cmake
add_library(lib lib.cpp)
```

### 使用子文件夹作为库

```cmake
add_subdirectory(./lib)

include_directories(./lib)
link_directories(C:/path/to/lib)

add_executable(target main.cpp)

target_link_libraries(target lib.lib)
```