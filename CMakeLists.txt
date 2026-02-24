cmake_minimum_required(VERSION 3.10)

project(summertts LANGUAGES C CXX VERSION 1.0.0)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 按平台设置输出目录
if (WIN32)
    # Windows: DLL EXE 导入库 放 bin_win
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_win)  # .exe, .dll
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_win)  # .dll
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_win)  # .lib
else()
    # Linux/macOS: 可执行文件 库文件 放 bin_linux
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_linux)
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_linux)
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_linux)
endif()

# 查找 OpenMP
find_package(OpenMP REQUIRED)

# 添加库子目录
add_subdirectory(lib)

# --------------------------
# 构建测试程序
# --------------------------
add_executable(${PROJECT_NAME}_test main.cpp)
target_include_directories(${PROJECT_NAME}_test PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/include
        ${CMAKE_CURRENT_SOURCE_DIR}/lib/src/header
)

# 添加编译选项
target_compile_options(${PROJECT_NAME}_test PRIVATE -O3)

# 链接动态库库和 OpenMP 库
target_link_libraries(${PROJECT_NAME}_test PRIVATE
        summertts # 链接到动态库
)