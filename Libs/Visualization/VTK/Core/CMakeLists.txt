project(CTKVisualizationVTKCore)

#
# 3rd party dependencies
#
include(${VTK_USE_FILE})

if(CTK_LIB_Scripting/Python/Core AND CTK_LIB_Scripting/Python/Core_PYTHONQT_USE_VTK)
  find_package(PythonQt REQUIRED)
endif()

#
# See CTK/CMake/ctkMacroBuildLib.cmake for details
#

set(KIT_export_directive "CTK_VISUALIZATION_VTK_CORE_EXPORT")


# Source files
set(KIT_SRCS
  ctkVTKConnection.cpp
  ctkVTKConnection.h
  ctkVTKErrorLogMessageHandler.cpp
  ctkVTKErrorLogMessageHandler.h
  ctkVTKObject.h
  ctkVTKObjectEventsObserver.cpp
  ctkVTKObjectEventsObserver.h
  vtkLightBoxRendererManager.cpp
  vtkLightBoxRendererManager.h
  )

# Headers that should run through moc
set(KIT_MOC_SRCS
  ctkVTKConnection.h
  ctkVTKObjectEventsObserver.h
  )

# UI files
set(KIT_UI_FORMS
  )

# Resources
set(KIT_resources
  )

if(CTK_LIB_Scripting/Python/Core AND CTK_LIB_Scripting/Python/Core_PYTHONQT_USE_VTK)
  list(APPEND KIT_SRCS
    ctkVTKPythonQtWrapperFactory.cpp
    )
endif()

set(KIT_INCLUDE_DIRECTORIES )

if(CTK_LIB_Scripting/Python/Core AND CTK_LIB_Scripting/Python/Core_PYTHONQT_USE_VTK)
  list(APPEND KIT_INCLUDE_DIRECTORIES
    ${PYTHON_INCLUDE_DIRS}
    ${PYTHONQT_INCLUDE_DIR}
    )
endif()

# Set VTK_LIBRARIES variable
if(${VTK_VERSION_MAJOR} GREATER 5)
  set(VTK_LIBRARIES
    vtkCommonCore
    vtkCommonDataModel
    vtkCommonSystem
    vtkIOImage
    vtkInteractionStyle
    vtkRenderingAnnotation
    vtkRenderingCore
    vtkRenderingFreeTypeOpenGL
    vtkRenderingOpenGL
    vtkTestingRendering
    )
  if (UNIX AND NOT APPLE)
    find_package(FontConfig QUIET)
    if (FONTCONFIG_FOUND)
      list(APPEND VTK_LIBRARIES
        vtkRenderingFreeTypeFontConfig
        )
    endif()
  endif()
else()
  set(VTK_LIBRARIES
    vtkCommon
    vtkFiltering
    vtkRendering
    vtkHybrid
    )
endif()

if(CTK_LIB_Scripting/Python/Core AND CTK_LIB_Scripting/Python/Core_PYTHONQT_USE_VTK)
  list(APPEND VTK_LIBRARIES ${PYTHON_LIBRARY} ${PYTHONQT_LIBRARIES})
  if(${VTK_VERSION_MAJOR} GREATER 5)
    list(APPEND VTK_LIBRARIES vtkWrappingPythonCore)
  elseif(${VTK_VERSION_MAJOR}.${VTK_VERSION_MINOR} VERSION_GREATER 5.6)
    list(APPEND VTK_LIBRARIES vtkPythonCore)
  else()
    list(APPEND VTK_LIBRARIES vtkCommonPythonD)
  endif()
endif()

if(Q_WS_X11)
  # If the faster 'gold' linker is used, to avoid complaints about undefined symbol
  # 'XGetWindowAttributes', 'XCreateWindow', ..., let's link against X11 libraries.
  find_package(X11)
  list(APPEND VTK_LIBRARIES ${X11_LIBRARIES})
endif()

# Target libraries - See CMake/ctkFunctionGetTargetLibraries.cmake
# The following macro will read the target libraries from the file 'target_libraries.cmake'
ctkFunctionGetTargetLibraries(KIT_target_libraries)

ctkMacroBuildLib(
  NAME ${PROJECT_NAME}
  EXPORT_DIRECTIVE ${KIT_export_directive}
  SRCS ${KIT_SRCS}
  MOC_SRCS ${KIT_MOC_SRCS}
  UI_FORMS ${KIT_UI_FORMS}
  INCLUDE_DIRECTORIES ${KIT_INCLUDE_DIRECTORIES}
  TARGET_LIBRARIES ${KIT_target_libraries}
  RESOURCES ${KIT_resources}
  LIBRARY_TYPE ${CTK_LIBRARY_MODE}
  )

if(CTK_WRAP_PYTHONQT_LIGHT)
  ctkMacroBuildLibWrapper(
    TARGET ${PROJECT_NAME}
    SRCS ${KIT_SRCS}
    WRAPPER_LIBRARY_TYPE ${CTK_LIBRARY_MODE}
    )
endif()

# Plugins
#if(CTK_BUILD_QTDESIGNER_PLUGINS)
#  add_subdirectory(Plugins)
#endif()

# Testing
if(BUILD_TESTING)
  add_subdirectory(Testing)
endif()
