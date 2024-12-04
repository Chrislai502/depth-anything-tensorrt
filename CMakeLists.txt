# ---------------------------------------------------------------------------
#                Set minimum required version of CMake
# ---------------------------------------------------------------------------
# The `cmake_minimum_required` command ensures compatibility with specific
# features or commands introduced in the specified CMake version.
# Version 3.12 is chosen here, as it provides a good balance of modern
# functionality and compatibility.
cmake_minimum_required(VERSION 3.12)

# ---------------------------------------------------------------------------
#                Define the project name
# ---------------------------------------------------------------------------
# The `project` command sets the project name. In this case, the project is
# named "depth-anything-tensorrt", which suggests it involves depth
# estimation or manipulation and leverages NVIDIA's TensorRT.
project(depth-anything-tensorrt)

# ---------------------------------------------------------------------------
#                Set the C++ standard
# ---------------------------------------------------------------------------
# C++17 is chosen for this project, ensuring access to modern C++ features
# such as structured bindings, filesystem library, and more.
set(CMAKE_CXX_STANDARD 17)

# ---------------------------------------------------------------------------
#                Add source files
# ---------------------------------------------------------------------------
# The `SOURCES` variable defines the list of `.cpp` files (source files)
# that will be compiled to create the final executable. These files likely
# contain the main logic for the application.
set(SOURCES
	main.cpp              # Main entry point of the application.
	utils.cpp             # Utility functions used across the application.
	depth_anything.cpp    # Implementation of the core functionality.
    # Additional source files can be added here as the project grows.
)

# ---------------------------------------------------------------------------
#                Add header files
# ---------------------------------------------------------------------------
# The `HEADERS` variable lists `.h` or `.hpp` files (header files), which
# declare the interfaces, classes, or functions used in the source files.
set(HEADERS
	utils.h               # Header for utility functions.
	depth_anything.h      # Header for the core application functionality.
    # Additional header files can be added here as the project grows.
)

# ---------------------------------------------------------------------------
#                Find and include CUDA
# ---------------------------------------------------------------------------
# The `find_package(CUDA REQUIRED)` command locates the NVIDIA CUDA toolkit,
# which provides tools and libraries for GPU-accelerated computations.
# The `include_directories` command ensures that the CUDA headers are made
# available to the compiler.
find_package(CUDA REQUIRED)
include_directories(${CUDA_INCLUDE_DIRS})

# ---------------------------------------------------------------------------
#                Find and include OpenCV
# ---------------------------------------------------------------------------
# OpenCV is a computer vision library widely used for tasks like image
# processing, object detection, and depth estimation. The `find_package`
# command locates the OpenCV installation, and `include_directories` ensures
# that OpenCV headers are accessible to the compiler.

# Note: The `OpenCV_DIR` variable is explicitly set to the OpenCV build directory.
# On Windows, this is necessary if OpenCV is not installed system-wide.
# set(OpenCV_DIR "D:\\Utils\\opencv\\opencv49_cpu\\build")
find_package(OpenCV REQUIRED)

# If OpenCV is not found, error out
if (NOT OpenCV_FOUND)
    message(FATAL_ERROR "OpenCV not found. Please ensure OpenCV is installed and available in the system path.")
endif()

include_directories(${OpenCV_INCLUDE_DIRS})


# ---------------------------------------------------------------------------
#                Set TensorRT path
# ---------------------------------------------------------------------------
# Locate TensorRT headers and libraries
find_path(TENSORRT_INCLUDE_DIR NvInfer.h PATHS $ENV{TENSORRT_HOME}/include /usr/include /usr/local/include)
find_library(NVINFER_LIBRARY nvinfer PATHS $ENV{TENSORRT_HOME}/lib /usr/lib /usr/local/lib)
find_library(NVINFER_PLUGIN_LIBRARY nvinfer_plugin PATHS $ENV{TENSORRT_HOME}/lib /usr/lib /usr/local/lib)
find_library(NVONNXPARSER_LIBRARY nvonnxparser PATHS $ENV{TENSORRT_HOME}/lib /usr/lib /usr/local/lib)

# If TensorRT is not found, throw an error
if (NOT TENSORRT_INCLUDE_DIR OR NOT NVINFER_LIBRARY OR NOT NVINFER_PLUGIN_LIBRARY OR NOT NVONNXPARSER_LIBRARY)
    message(FATAL_ERROR "TensorRT not found. Please ensure TensorRT is installed and paths are configured correctly.")
endif()

# Include TensorRT headers and set libraries
include_directories(${TENSORRT_INCLUDE_DIR})
set(TENSORRT_LIBS ${NVINFER_LIBRARY} ${NVINFER_PLUGIN_LIBRARY} ${NVONNXPARSER_LIBRARY})


# ---------------------------------------------------------------------------
#                Add executable target
# ---------------------------------------------------------------------------
# Create the executable from the source and header files defined earlier.
# The `add_executable` command specifies the target's name (`depth-anything-tensorrt`)
# and the list of source/header files to compile and link.
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

# ---------------------------------------------------------------------------
#                Link libraries to the executable
# ---------------------------------------------------------------------------
# The `target_link_libraries` command links the necessary libraries to the
# executable target. These include:
#   - `OpenCV_LIBS`: Libraries required by OpenCV for image processing.
#   - `CUDA_LIBRARIES`: Libraries required by CUDA for GPU computations.
#   - `TENSORRT_LIBS`: TensorRT libraries for deep learning inference.
target_link_libraries(${PROJECT_NAME}
    ${OpenCV_LIBS}       # OpenCV libraries.
    ${CUDA_LIBRARIES}    # CUDA libraries.
    ${TENSORRT_LIBS}     # TensorRT libraries, selected based on version/platform.
)
