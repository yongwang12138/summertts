cmake_minimum_required(VERSION 3.10)

project(summertts LANGUAGES C CXX VERSION 1.0.0)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 编译动态库必须开启
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# ============ 第三方库管理 ============
# 启用 FetchContent
include(FetchContent)
# 设置第三方库下载目录
set(THIRD_PARTY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/third_party)
# 创建第三方库目录
file(MAKE_DIRECTORY ${THIRD_PARTY_DIR})

# ---------- eigen ----------
set(EIGEN_INCLUDE_DIR ${THIRD_PARTY_DIR}/eigen)

if(EXISTS ${EIGEN_INCLUDE_DIR})
    message(STATUS "Found existing Eigen at ${EIGEN_INCLUDE_DIR}")
else ()
    message(STATUS "Downloading Eigen to ${EIGEN_INCLUDE_DIR} ...")
    FetchContent_Populate(
        eigen
        GIT_REPOSITORY https://gitlab.com/libeigen/eigen.git
        GIT_TAG 3.4.0
        SOURCE_DIR ${EIGEN_INCLUDE_DIR}
        QUIET # 减少下载时的输出信息
    )
endif()

# --------------------------
# 收集库源码
# --------------------------
set(TTS_LIB_SRC
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/compat.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/flags.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/fst.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/fst-types.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/mapped-file.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/properties.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/symbol-table.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/symbol-table-ops.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/util.cc
        ${PROJECT_SOURCE_DIR}/src/tn/openfst/weight.cc
        ${PROJECT_SOURCE_DIR}/src/tn/processor.cc
        ${PROJECT_SOURCE_DIR}/src/tn/token_parser.cc
        ${PROJECT_SOURCE_DIR}/src/tn/utf8_string.cc
        ${PROJECT_SOURCE_DIR}/src/engipa/EnglishText2Id.cpp
        ${PROJECT_SOURCE_DIR}/src/engipa/InitIPASymbols.cpp
        ${PROJECT_SOURCE_DIR}/src/engipa/alphabet.cpp
        ${PROJECT_SOURCE_DIR}/src/engipa/ipa.cpp
        ${PROJECT_SOURCE_DIR}/src/hz2py/hanzi2phoneid.cpp
        ${PROJECT_SOURCE_DIR}/src/hz2py/Hanz2Piny.cpp
        ${PROJECT_SOURCE_DIR}/src/hz2py/pinyinmap.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_conv1d.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_softmax.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_layer_norm.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_relu.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_gelu.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_tanh.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_flip.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_cumsum.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_softplus.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_clamp_min.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_sigmoid.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_conv1d_transposed.cpp
        ${PROJECT_SOURCE_DIR}/src/nn_op/nn_leaky_relu.cpp
        ${PROJECT_SOURCE_DIR}/src/platform/tts_file_io.cpp
        ${PROJECT_SOURCE_DIR}/src/platform/tts_logger.cpp
        ${PROJECT_SOURCE_DIR}/src/utils/utils.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/iStft.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/hann.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/attention_encoder.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/multi_head_attention.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/ffn.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/ConvFlow.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/DDSConv.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/ElementwiseAffine.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/random_gen.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/ResidualCouplingLayer.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/ResBlock1.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/WN.cpp
        ${PROJECT_SOURCE_DIR}/src/modules/pqmf.cpp
        ${PROJECT_SOURCE_DIR}/src/models/TextEncoder.cpp
        ${PROJECT_SOURCE_DIR}/src/models/StochasticDurationPredictor.cpp
        ${PROJECT_SOURCE_DIR}/src/models/FixDurationPredictor.cpp
        ${PROJECT_SOURCE_DIR}/src/models/DurationPredictor_base.cpp
        ${PROJECT_SOURCE_DIR}/src/models/ResidualCouplingBlock.cpp
        ${PROJECT_SOURCE_DIR}/src/models/Generator_base.cpp
        ${PROJECT_SOURCE_DIR}/src/models/Generator_hifigan.cpp
        ${PROJECT_SOURCE_DIR}/src/models/Generator_MS.cpp
        ${PROJECT_SOURCE_DIR}/src/models/Generator_Istft.cpp
        ${PROJECT_SOURCE_DIR}/src/models/Generator_MBB.cpp
        ${PROJECT_SOURCE_DIR}/src/models/SynthesizerTrn.cpp
)

# --------------------------
# 构建动态库
# --------------------------
add_library(${PROJECT_NAME} SHARED ${TTS_LIB_SRC})
target_include_directories(${PROJECT_NAME} PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/include
        ${CMAKE_CURRENT_SOURCE_DIR}/src/header
        ${CMAKE_CURRENT_SOURCE_DIR}/src/tn/header
        ${THIRD_PARTY_DIR}/engine
        ${EIGEN_INCLUDE_DIR}
)

# 添加编译选项
target_compile_options(${PROJECT_NAME} PRIVATE -O3)

# 链接 OpenMP 库
target_link_libraries(${PROJECT_NAME} PRIVATE OpenMP::OpenMP_CXX)

# Windows 平台特定设置
if (WIN32 AND MINGW)
    # 对于 MinGW，可能需要额外链接 pthread
    target_link_libraries(${PROJECT_NAME} PRIVATE
            -lpthread
            -lgomp
    )
endif ()