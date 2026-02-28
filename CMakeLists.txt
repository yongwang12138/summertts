cmake_minimum_required(VERSION 3.10)

project(summertts LANGUAGES C CXX VERSION 1.0.0)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 按平台设置输出目录
if (WIN32)
    # Windows: DLL EXE 导入库
    set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_win64)  # .exe, .dll
    set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_win64)  # .dll
    set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_win64)  # .lib
    message(STATUS "Platform: Windows 64")
else()
    # Linux: 可执行文件 库文件
    if(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64|ARM64|arm64")
        set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_arm64)
        set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_arm64)
        set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_arm64)
        message(STATUS "Platform: Linux ARM64")
    else()
        set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_x64)
        set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_x64)
        set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/bin_x64)
        message(STATUS "Platform: Linux x64")
    endif()

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
if(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64|ARM64|arm64")
    target_compile_options(${PROJECT_NAME}_test PRIVATE 
        -O3 
        -march=armv8.2-a+fp16+rcpc+dotprod
        -mtune=generic-armv8-a # 通用 ARMv8 优化
        -ftree-vectorize
        -funroll-loops # 循环展开
    )
    message(STATUS "启用RK3588 ARM 编译优化")
else()
    target_compile_options(${PROJECT_NAME}_test PRIVATE -O3)
    message(STATUS "使用默认编译优化")
endif()

# 链接动态库 和 OpenMP 库
target_link_libraries(${PROJECT_NAME}_test PRIVATE
        summertts # 链接到动态库
)