set(plugin CnoidOpenRTMPlugin)

set(plugin_sources
  OpenRTMPlugin.cpp
  RTSCommonUtil.cpp
  ControllerRTCItem.cpp
  BodyIoRTCItem.cpp
  RTCWrapper.cpp
  RTCItem.cpp
  SimulationExecutionContext.cpp
  SimulationPeriodicExecutionContext.cpp
  RTSNameServerView.cpp
  RTSPropertiesView.cpp
  RTMImageView.cpp
  RTSConfigurationView.cpp
  BodyStateSubscriberRTCItem.cpp
  LoggerUtil.cpp
  RTSTypeUtil.cpp
  PortConnectionDialog.cpp
  ProfileHandler.cpp
  pugixml/pugixml.cpp
  #deprecated/BodyRTCItem.cpp
  #deprecated/VirtualRobotRTC.cpp
  #deprecated/VirtualRobotPortHandler.cpp
  #deprecated/ChoreonoidExecutionContext.cpp
  #deprecated/ChoreonoidPeriodicExecutionContext.cpp
  #deprecated/BridgeConf.cpp
  #deprecated/PointCloudSubscriberRTCItem.cpp
  )

set(public_headers
  OpenRTMUtil.h
  RTCItem.h
  BodyIoRTCItem.h
  ControllerRTCItem.h
  OpenRTMPlugin.h
  ProfileHandler.h
  RTSystem.h
  RTSystemItem.h
  RTSCommonUtil.h
  RTCWrapper.h
  RTSDiagramView.h
  deprecated/BodyRTCItem.h
  )

set(plugin_headers
  ${public_headers} 
  exportdecl.h
  )

choreonoid_make_headers_public(${public_headers})

set(qt_wrap_cpp_files
  RTSNameServerView.h
  RTSConfigurationView.h)

option(ENABLE_NEW_RT_SYSTEM_ITEM_IMPLEMENTATION "Enable the new implementation of RTSystemItem" OFF)
#option(ENABLE_BACKGROUND_STATE_DETECTION "Enable background state detection" OFF)

if(ENABLE_NEW_RT_SYSTEM_ITEM_IMPLEMENTATION)
  add_definitions(-DENABLE_NEW_RT_SYSTEM_ITEM_IMPLEMENTATION)
  
  #if(ENABLE_BACKGROUND_STATE_DETECTION)
    #add_definitions(-DENABLE_BACKGROUND_STATE_DETECTION)
  #endif()

  set(plugin_sources ${plugin_sources}
    experimental/RTSystemItemEx.cpp
    experimental/RTSDiagramViewEx.cpp
    )
  set(qt_wrap_cpp_files ${qt_wrap_cpp_files} experimental/RTSDiagramViewEx.h)
else()
  set(plugin_sources ${plugin_sources}
    RTSystemItem.cpp
    RTSDiagramView.cpp
    )
  set(qt_wrap_cpp_files ${qt_wrap_cpp_files} RTSDiagramView.h)
endif()  

qt5_add_resources(RC_SRCS OpenRTMPlugin.qrc)

qt5_wrap_cpp(qtsources ${qt_wrap_cpp_files})

set(plugin_sources ${plugin_sources} ${qtsources})

unset(idl_cpp_files)
unset(idl_h_files)
if(USE_BUILTIN_CAMERA_IMAGE_IDL)
  set(idl_files ${idl_files} CameraImage)
  choreonoid_compile_corba_idl_to_cpp(idl_cpp_files idl_h_files deprecated/corba ${idl_files} LOCAL)
else()
  set(OPENRTM_LIBRARIES ${OPENRTM_LIBRARIES} ${OPENRTM_CAMERA_LIBRARY})
endif()
include_directories(${CMAKE_CURRENT_BINARY_DIR})

make_gettext_mofiles(${plugin} mofiles)

if(MSVC)
  add_compile_options(${OPENRTM_CFLAGS})
endif()

choreonoid_add_plugin(${plugin}
  ${plugin_sources} ${idl_cpp_files} ${idl_h_files} ${mofiles} ${RC_SRCS} HEADERS ${plugin_headers})

if(IS_MASTER_PROJECT)
  target_compile_definitions(${plugin} PUBLIC ${CHOREONOID_QT_COMPILE_DEFINITIONS})
  target_link_libraries(${plugin}
    PUBLIC CnoidBodyIoRTC
    ${OPENRTM_LIBRARIES} ${CHOREONOID_CORBA_PLUGIN_LIBRARIES} ${CHOREONOID_BODY_PLUGIN_LIBRARIES})
else()
  target_link_libraries(${plugin}
    CnoidBodyPlugin CnoidCorbaPlugin CnoidBodyIoRTC ${OPENRTM_LIBRARIES})
endif()

if(CHOREONOID_ENABLE_PYTHON)
  add_subdirectory(pybind11)
endif()

file(MAKE_DIRECTORY ${PROJECT_BINARY_DIR}/${CHOREONOID_PLUGIN_SUBDIR}/rtc)
install(DIRECTORY ${PROJECT_BINARY_DIR}/${CHOREONOID_PLUGIN_SUBDIR}/rtc
  DESTINATION ${CHOREONOID_PLUGIN_SUBDIR}/rtc FILES_MATCHING PATTERN "*" EXCLUDE)


add_subdirectory(pugixml)