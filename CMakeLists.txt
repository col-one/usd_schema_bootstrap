cmake_minimum_required(VERSION 3.5)
set(USDPLUGIN_NAME myNewSchema)

project(${USDPLUGIN_NAME})

set(CMAKE_INSTALL_PREFIX "./install" CACHE PATH "..." FORCE)

find_package(PythonLibs 3.7 REQUIRED)
add_definitions(-D_GLIBCXX_USE_CXX11_ABI=1) # =0 if you build usd with _GLIBCXX_USE_CXX11_ABI=0

set(CMAKE_MODULE_PATH
	${USD_BUILD_DIR}
	${CMAKE_MODULE_PATH}
)
add_subdirectory(src)
