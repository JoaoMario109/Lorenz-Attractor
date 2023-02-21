# Version settings
cmake_minimum_required(VERSION 3.7...3.20)

if (${CMAKE_VERSION} VERSION_LESS 3.12)
	cmake_policy(VERSION ${CMAKE_MAJOR_VERSION}.${CMAKE_MINOR_VERSION})
endif()

# Project settings
project(
	lorenz-attractor VERSION 1.0
	DESCRIPTION "Lorenz Attractor simulation in C++ with OpenGL."
	LANGUAGES C CXX
)

# Doxygen setup

# If doxygen is installed so generates the documentation
find_package(Doxygen)
if (DOXYGEN_FOUND)
	# Docs input and output
	set(DOXYGEN_IN "${CMAKE_CURRENT_SOURCE_DIR}/docs/doxygen/Doxyfile.in")
	set(DOXYGEN_OUT "${CMAKE_CURRENT_BINARY_DIR}/Doxyfile")

	# Configure the output documentation file
	configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)
	message("Doxygen build started")

	# Generates the doc
	add_custom_target(
		documentation ALL
		COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
		WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
		COMMENT "Generating documentation with Doxygen"
		VERBATIM
	)
else (DOXYGEN_FOUND)
	message("Skipping documentation generation, to generate documentation doxygen needs to be instaled")
endif (DOXYGEN_FOUND)

# Source Dir variable
set(SRC_DIR "${CMAKE_CURRENT_SOURCE_DIR}/source")

# Vendors Dir variable
set(VENDORS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/vendors")

# Translations units of the project
set(
	TRANSLATION_FILES
	"${SRC_DIR}/main.cpp"
)

# Header files of the project
set(
	HEADERS_FILES
	""
)

# Executable settings
add_executable(${PROJECT_NAME} ${TRANSLATION_FILES} ${HEADERS_FILES})
target_include_directories(${PROJECT_NAME} PRIVATE "${SRC_DIR}")
set_property(TARGET ${PROJECT_NAME} PROPERTY CXX_STANDARD 20)

# GLFW without examples included
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "Build the GLFW example programs")
# GLFW lib
set(GLFW_DIR "${VENDORS_DIR}/glfw")
add_subdirectory("${GLFW_DIR}")
target_link_libraries(${PROJECT_NAME} PRIVATE "glfw")
target_include_directories(${PROJECT_NAME} PRIVATE "${GLFW_DIR}/include")
## GLFW Predefined defines
target_compile_definitions(${PROJECT_NAME} PRIVATE "GLFW_INCLUDE_NONE")


# GLM
set(GLM_DIR "${VENDORS_DIR}/glm")
add_subdirectory("${GLM_DIR}")
target_link_libraries(${PROJECT_NAME} PRIVATE "glm")
target_include_directories(${PROJECT_NAME} PRIVATE "${GLM_DIR}")

# GLAD
set(GLAD_DIR "${VENDORS_DIR}/glad")
add_subdirectory("${GLAD_DIR}")
target_link_libraries(${PROJECT_NAME} PRIVATE "glad")
target_include_directories(${PROJECT_NAME} PRIVATE "${GLAD_DIR}/include")

# Custom ImGui build for modern OpenGL using GLFW default backend
set(IMGUI_DIR "${VENDORS_DIR}/imgui")
add_library("imgui" STATIC)
target_sources(
	"imgui"
	PRIVATE
	"${IMGUI_DIR}/imgui_demo.cpp"
	"${IMGUI_DIR}/imgui_draw.cpp"
	"${IMGUI_DIR}/imgui_tables.cpp"
	"${IMGUI_DIR}/imgui_widgets.cpp"
	"${IMGUI_DIR}/imgui.cpp"
	PRIVATE
	"${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp"
	"${IMGUI_DIR}/backends/imgui_impl_glfw.cpp"
)
target_include_directories(
	"imgui"
	PRIVATE "${IMGUI_DIR}"
	PRIVATE "${IMGUI_DIR}/backends"
	PRIVATE "${GLFW_DIR}/include"
)

target_link_libraries(${PROJECT_NAME} PRIVATE "imgui")
target_include_directories(
	${PROJECT_NAME}
	PRIVATE "${IMGUI_DIR}"
	PRIVATE "${IMGUI_DIR}/backends"
)

# Post build settings

# Copy assets directory to output build directory
file(COPY ./assets DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
