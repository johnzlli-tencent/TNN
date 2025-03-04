if(DEBUG)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")
endif()


include_directories(${CMAKE_CURRENT_SOURCE_DIR}/source/tnn/device/opencl)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/third_party/opencl/include)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/third_party/stb)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/platforms/android/src)

set(COMMON_SRC ${CMAKE_CURRENT_SOURCE_DIR}/platforms/android/src/test_common.cc)

target_link_libraries(TNN dl rt)

if(TNN_X86_ENABLE)
endif()

if(TNN_OPENVINO_ENABLE)

    if(NOT DEFINED ENV{OPENVINO_ROOT_DIR})
        message(FATAL_ERROR "not defined environment variable:OPENVINO_ROOT_DIR")
    endif()

    if (TNN_OPENVINO_BUILD_SHARED)
        set(LINK_TYPE "SHARED")
        set(LIB_EXT ".so")
    else()
        set(LINK_TYPE "STATIC")
        set(LIB_EXT ".a")
    endif()

    add_library(inference_engine ${LINK_TYPE} IMPORTED)
    add_library(inference_engine_legacy ${LINK_TYPE} IMPORTED)
    add_library(inference_engine_transformations ${LINK_TYPE} IMPORTED)
    add_library(inference_engine_lp_transformations ${LINK_TYPE} IMPORTED)
    # add_library(MKLDNNPlugin ${LINK_TYPE} IMPORTED)
    add_library(ngraph ${LINK_TYPE} IMPORTED)
    # add_library(pugixml STATIC IMPORTED)
    add_library(tbb ${LINK_TYPE} IMPORTED)

    set_target_properties(inference_engine PROPERTIES IMPORTED_LOCATION $ENV{OPENVINO_ROOT_DIR}/deployment_tools/inference_engine/lib/intel64/${LIB_PFX}inference_engine${LIB_EXT})
    set_target_properties(inference_engine_legacy PROPERTIES IMPORTED_LOCATION $ENV{OPENVINO_ROOT_DIR}/deployment_tools/inference_engine/lib/intel64/${LIB_PFX}inference_engine_legacy${LIB_EXT})
    set_target_properties(inference_engine_transformations PROPERTIES IMPORTED_LOCATION $ENV{OPENVINO_ROOT_DIR}/deployment_tools/inference_engine/lib/intel64/${LIB_PFX}inference_engine_transformations${LIB_EXT})
    set_target_properties(inference_engine_lp_transformations PROPERTIES IMPORTED_LOCATION $ENV{OPENVINO_ROOT_DIR}/deployment_tools/inference_engine/lib/intel64/${LIB_PFX}inference_engine_lp_transformations${LIB_EXT})
    # set_target_properties(MKLDNNPlugin PROPERTIES IMPORTED_LOCATION $ENV{OPENVINO_ROOT_DIR}/deployment_tools/inference_engine/lib/intel64/${LIB_PFX}MKLDNNPlugin.so)
    set_target_properties(ngraph PROPERTIES IMPORTED_LOCATION $ENV{OPENVINO_ROOT_DIR}/deployment_tools/ngraph/lib/${LIB_PFX}ngraph${LIB_EXT})
    set_target_properties(tbb PROPERTIES IMPORTED_LOCATION $ENV{OPENVINO_ROOT_DIR}/deployment_tools/inference_engine/external/tbb/lib/libtbb.so.2)
    # set_target_properties(pugixml PROPERTIES IMPORTED_LOCATION $ENV{OPENVINO_ROOT_DIR}/lib/${LIB_PFX}pugixml.a)
    
    target_link_libraries(TNN inference_engine inference_engine_legacy inference_engine_transformations inference_engine_lp_transformations ngraph tbb)
endif()

if(TNN_CUDA_ENABLE)
    enable_language(CUDA)
    include_directories(${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES})
    add_library(nvinfer SHARED IMPORTED)
    add_library(nvinfer_plugin SHARED IMPORTED)
    add_library(cudnn SHARED IMPORTED)
    set_target_properties(nvinfer PROPERTIES IMPORTED_LOCATION $ENV{TENSORRT_ROOT_DIR}/lib/libnvinfer.so)
    set_target_properties(nvinfer_plugin PROPERTIES IMPORTED_LOCATION $ENV{TENSORRT_ROOT_DIR}/lib/libnvinfer_plugin.so)
    set_target_properties(cudnn PROPERTIES IMPORTED_LOCATION $ENV{CUDNN_ROOT_DIR}/lib64/libcudnn.so)
    target_link_libraries(TNN nvinfer nvinfer_plugin cudnn)
endif()

if(TNN_RK_NPU_ENABLE)
    message(STATUS "Build TNN RKNPU")
    target_link_libraries(TNN rknpu_ddk)
endif()

