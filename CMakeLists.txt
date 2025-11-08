cmake_minimum_required(VERSION 3.18)
# `CMAKE_CUDA_ARCHITECTURES` requires 3.18
# set_source_files_properties requires 3.18

# 项目基本信息
project(Sunshine VERSION 0.23.1
        DESCRIPTION "Self-hosted game stream host for Moonlight"
        HOMEPAGE_URL "https://app.lizardbyte.dev/Sunshine")

set(PROJECT_LICENSE "GPL-3.0")

set(PROJECT_LONG_DESCRIPTION "Offering low latency, cloud gaming server capabilities with support for AMD, Intel, \
and Nvidia GPUs for hardware encoding. Software encoding is also available. You can connect to Sunshine from any \
Moonlight client on a variety of devices. A web UI is provided to allow configuration, and client pairing, from \
your favorite web browser. Pair from the local server or any mobile device.")

# 默认构建类型设置
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to 'Release' as none was specified.")
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Choose the type of build." FORCE)
endif()

# CMake 模块路径配置
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")

# 【关键修改】Boost 依赖配置（适配代码中使用的 io_context 和 process 新类型）
set(BOOST_MIN_VERSION 1.66)  # 最低支持版本（包含 io_context 和 process_environment）
set(Boost_REQUIRED_COMPONENTS system process)  # 显式指定所需组件

# 版本信息生成
include(${CMAKE_MODULE_PATH}/prep/build_version.cmake)

# 构建选项配置
include(${CMAKE_MODULE_PATH}/prep/options.cmake)

# 初始化配置（编译器设置等）
include(${CMAKE_MODULE_PATH}/prep/init.cmake)

# 特殊包配置（桌面文件、Flatpak 清单等）
include(${CMAKE_MODULE_PATH}/prep/special_package_configuration.cmake)

# 若仅生成包清单，提前退出
if(${END_BUILD})
    return()
endif()

# 项目常量定义
include(${CMAKE_MODULE_PATH}/prep/constants.cmake)

# 通用宏加载
include(${CMAKE_MODULE_PATH}/macros/common.cmake)

# 依赖项加载（包含 Boost 查找）
include(${CMAKE_MODULE_PATH}/dependencies/common.cmake)

# 【关键修改】验证 Boost 版本是否满足要求
if(Boost_VERSION VERSION_LESS BOOST_MIN_VERSION)
    message(FATAL_ERROR "Boost version ${BOOST_MIN_VERSION} or higher is required, but found ${Boost_VERSION}. "
                        "Please update your Boost library.")
endif()

# 编译定义配置
include(${CMAKE_MODULE_PATH}/compile_definitions/common.cmake)

# 目标定义（生成可执行文件/库）
include(${CMAKE_MODULE_PATH}/targets/common.cmake)

# 打包配置
include(${CMAKE_MODULE_PATH}/packaging/common.cmake)
