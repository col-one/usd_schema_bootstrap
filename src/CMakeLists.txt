include(pxrConfig)

file(GLOB SRCFILES CONFIGURE_DEPENDS
     "*.h"
     "*.cpp"
)

add_library(${USDPLUGIN_NAME}
    SHARED
        ${SRCFILES}
)


set(USD_INCLUDE_DIR "${USD_BUILD_DIR}/include")

target_include_directories(${USDPLUGIN_NAME}
	                    PRIVATE ${PXR_INCLUDE_DIRS}
						        ${PXR_INCLUDE_DIRS}/boost-1_70
								${Python_INCLUDE_DIRS}
								${Python_LIBRARY_DIRS} ${USD_BUILD_DIR}/lib)


set(USD_LIB_DIR "${USD_BUILD_DIR}/lib")

target_link_directories(${USDPLUGIN_NAME} PRIVATE ${USD_LIB_DIR} ${Python_LIBRARY_DIRS})

target_link_libraries(${USDPLUGIN_NAME}
	PUBLIC
	tf
	sdf
	usd
	${Python_LIBRARIES})


set_target_properties(${USDPLUGIN_NAME} PROPERTIES PREFIX "")


target_compile_features(${USDPLUGIN_NAME}
	PUBLIC
	    cxx_auto_type
)

# install
# variables that will replace in plugInfo.json
set(PLUG_INFO_LIBRARY_PATH "../${CMAKE_SHARED_LIBRARY_PREFIX}${USDPLUGIN_NAME}${CMAKE_SHARED_LIBRARY_SUFFIX}")
set(PLUG_INFO_RESOURCE_PATH "resources")
set(PLUG_INFO_ROOT "..")

# simply copy plugInfo.json file to cmake_dir/bin dir
# @ONLY prevents the variables swap in the file with ${} syntax
configure_file(
	plugInfo.json
	${CMAKE_BINARY_DIR}/plugInfo.json
	@ONLY)

install(TARGETS ${USDPLUGIN_NAME}
	DESTINATION plugin
)

# copy the file during install
install(
	FILES ${CMAKE_BINARY_DIR}/plugInfo.json
	${CMAKE_CURRENT_SOURCE_DIR}/generatedSchema.usda # the auto generated schema, next to the plugInfo.json
	DESTINATION plugin/${USDPLUGIN_NAME}/resources
)

install(FILES
	${CMAKE_CURRENT_SOURCE_DIR}/plugInfo.root.json
	DESTINATION plugin/ RENAME plugInfo.json)



# python binding setup
set(USDPLUGIN_PYTHON_NAME _${USDPLUGIN_NAME})
set(MODULE_NAME ${USDPLUGIN_NAME})

file(GLOB WRAPFILES
     "wrap*.cpp"
)

add_library(${USDPLUGIN_PYTHON_NAME}
	SHARED
	    module.cpp
		moduleDeps.cpp
        ${WRAPFILES}
)


target_compile_definitions(${USDPLUGIN_PYTHON_NAME}
	PRIVATE
	    MFB_PACKAGE_NAME=${USDPLUGIN_NAME}
		MFB_ALT_PACKAGE_NAME=${USDPLUGIN_NAME}
		MFB_PACKAGE_MODULE=schemaFoo.${MODULE_NAME}
)
if(WIN32)
set_target_properties(${USDPLUGIN_PYTHON_NAME} PROPERTIES SUFFIX ".pyd")
else(WIN32)
set_target_properties(${USDPLUGIN_PYTHON_NAME} PROPERTIES PREFIX "")
endif(WIN32)


target_include_directories(${USDPLUGIN_PYTHON_NAME}
	                       PRIVATE ${PXR_INCLUDE_DIRS}
						           ${PXR_INCLUDE_DIRS}/boost-1_70
								   ${Python_INCLUDE_DIRS})
target_link_directories(${USDPLUGIN_PYTHON_NAME}
	                   PRIVATE ${Python_LIBRARY_DIRS})
target_link_libraries(${USDPLUGIN_PYTHON_NAME}
	                 ${USDPLUGIN_NAME}
					 ${Python_LIBRARIES}
					 )

 set(PYTHON_PACKAGE_PATH "python/${USDPLUGIN_NAME}")
 set(INSTALL_PYTHONPACKAGE_DIR ${PYTHON_PACKAGE_PATH})
 set(INSTALL_WRAPPER_DIR ${PYTHON_PACKAGE_PATH}/${MODULE_NAME})



 install(
	 TARGETS ${USDPLUGIN_PYTHON_NAME}
	 DESTINATION ${INSTALL_PYTHONPACKAGE_DIR}
 )

# configure __init__ file for python package, repalce @USDPLUGIN_PYTHON_NAME@
configure_file(
	__init__.py
	${CMAKE_BINARY_DIR}/__init__.py
	@ONLY)

 install(
	 FILES ${CMAKE_BINARY_DIR}/__init__.py
	 DESTINATION ${INSTALL_PYTHONPACKAGE_DIR}
	 RENAME __init__.py
 )



