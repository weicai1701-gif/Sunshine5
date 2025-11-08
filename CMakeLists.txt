cmake_minimum_required(VERSION 3.18)
# `CMAKE_CUDA_ARCHITECTURES` requires 3.18
# set_source_files_properties requires 3.18

# 处理 CMP0167 策略警告（Boost.Find 模块在新版 CMake 中被移除）
# 兼容旧版 FindBoost 行为，避免因策略问题导致 Boost 查找失败
cmake_policy(SET CMP0167 OLD)

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

# 【核心修改】Boost 依赖配置（显式指定组件及版本要求）
# 最低版本 1.66（包含 process 组件的完整支持及 io_context 等新类型）
set(BOOST_MIN_VERSION 1.66)
set(Boost_REQUIRED_COMPONENTS 
    system      # 基础系统功能
    process     # 进程管理（核心依赖，解决头文件找不到问题）
    locale      # 本地化支持
    log         # 日志功能
    filesystem  # 文件系统操作
    program_options  # 命令行参数解析
)

# 版本信息生成（保留原有逻辑）
include(${CMAKE_MODULE_PATH}/prep/build_version.cmake)

# 构建选项配置（保留原有逻辑）
include(${CMAKE_MODULE_PATH}/prep/options.cmake)

# 初始化配置（编译器设置等，保留原有逻辑）
include(${CMAKE_MODULE_PATH}/prep/init.cmake)

# 特殊包配置（桌面文件、Flatpak 清单等，保留原有逻辑）
include(${CMAKE_MODULE_PATH}/prep/special_package_configuration.cmake)

# 若仅生成包清单，提前退出（保留原有逻辑）
if(${END_BUILD})
    return()
endif()

# 项目常量定义（保留原有逻辑）
include(${CMAKE_MODULE_PATH}/prep/constants.cmake)

# 通用宏加载（保留原有逻辑）
include(${CMAKE_MODULE_PATH}/macros/common.cmake)

# 依赖项加载（包含 Boost 查找，关键修改：确保 Boost 组件被正确识别）
include(${CMAKE_MODULE_PATH}/dependencies/common.cmake)

# 【核心修改】验证 Boost 版本及组件是否满足要求
if(NOT Boost_FOUND)
    message(FATAL_ERROR "Boost library not found! Please install Boost >= ${BOOST_MIN_VERSION} with required components: ${Boost_REQUIRED_COMPONENTS}")
endif()

if(Boost_VERSION VERSION_LESS BOOST_MIN_VERSION)
    message(FATAL_ERROR "Boost version ${BOOST_MIN_VERSION} or higher is required, but found ${Boost_VERSION}. "
                        "Please update your Boost library.")
endif()

# 检查是否找到所有必需的 Boost 组件
foreach(component ${Boost_REQUIRED_COMPONENTS})
    if(NOT Boost_${component}_FOUND)
        message(FATAL_ERROR "Required Boost component '${component}' not found! Please install Boost with this component.")
    endif()
endforeach()

# 编译定义配置（保留原有逻辑，可在此添加 Boost 相关宏）
include(${CMAKE_MODULE_PATH}/compile_definitions/common.cmake)

# 目标定义（生成可执行文件/库，关键修改：确保链接 Boost 组件）
include(${CMAKE_MODULE_PATH}/targets/common.cmake)

# 【核心修改】显式链接 Boost 组件到主目标（确保 process 库被正确链接）
# 注：若原有 targets/common.cmake 中已定义 sunshine 目标，在此补充链接
if(TARGET sunshine)
    target_link_libraries(sunshine
        PRIVATE
            Boost::system
            Boost::process
            Boost::locale
            Boost::log
            Boost::filesystem
            Boost::program_options
    )
    # 添加 Boost 头文件目录（自动处理，此处为冗余保障）
    target_include_directories(sunshine PRIVATE ${Boost_INCLUDE_DIRS})
endif()

# 打包配置（保留原有逻辑）
include(${CMAKE_MODULE_PATH}/packaging/common.cmake)
