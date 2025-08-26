cmake_minimum_required(VERSION 3.16)
project(autolabeling-openvino LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ---- Qt ----
find_package(Qt6 REQUIRED COMPONENTS Core Widgets Gui)

# ---- OpenCV ----
set(OpenCV_DIR "${CMAKE_CURRENT_SOURCE_DIR}/opencv/build")
set(OpenCV_INCLUDE_DIR "${OpenCV_DIR}/include")
set(OpenCV_LIB_DIR "${OpenCV_DIR}/x64/vc16/lib")

include_directories(${OpenCV_INCLUDE_DIR})
link_directories(${OpenCV_LIB_DIR})

# Tự động tìm các file .lib trong thư mục
file(GLOB OpenCV_LIBS_RELEASE "${OpenCV_LIB_DIR}/*world*.lib")
set(OpenCV_LIBS ${OpenCV_LIBS_RELEASE})

# ---- OpenVINO ----
set(OPENVINO_DIR "${CMAKE_CURRENT_SOURCE_DIR}/openvino")
set(OPENVINO_INCLUDE_DIR "${OPENVINO_DIR}/runtime/include")
set(OPENVINO_LIB_DIR "${OPENVINO_DIR}/runtime/lib/intel64/Release")

include_directories(${OPENVINO_INCLUDE_DIR})
link_directories(${OPENVINO_LIB_DIR})

# Tự động tìm các file .lib trong thư mục
file(GLOB OPENVINO_LIBS_RELEASE "${OPENVINO_LIB_DIR}/*.lib")
set(OPENVINO_LIBS ${OPENVINO_LIBS_RELEASE})

# ---- Executable ----
add_executable(autolabel
    main.cpp
    # Thêm các file source khác ở đây nếu có
)

# ---- Link libraries ----
target_link_libraries(autolabel
    PRIVATE
        Qt6::Core
        Qt6::Widgets
        Qt6::Gui
        ${OpenCV_LIBS}
        ${OPENVINO_LIBS}
)

# ---- Copy DLLs sau khi build ----
if(WIN32)
    # Copy OpenVINO DLLs
    file(GLOB OPENVINO_DLLS "${OPENVINO_DIR}/runtime/bin/intel64/Release/*.dll")
    foreach(dll ${OPENVINO_DLLS})
        add_custom_command(TARGET autolabel POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${dll}
                $<TARGET_FILE_DIR:autolabel>
        )
    endforeach()

    # Copy OpenCV DLLs
    file(GLOB OPENCV_DLLS "${OpenCV_DIR}/x64/vc16/bin/*.dll")
    foreach(dll ${OPENCV_DLLS})
        add_custom_command(TARGET autolabel POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
                ${dll}
                $<TARGET_FILE_DIR:autolabel>
        )
    endforeach()
endif()
