
IF(DYNAMIC_OPENSCENEGRAPH)
    ADD_DEFINITIONS(-DOSGQT_LIBRARY)
ELSE()
    ADD_DEFINITIONS(-DOSG_LIBRARY_STATIC)
ENDIF()

SET(LIB_NAME osgQt)
SET(HEADER_PATH ${OpenSceneGraph_SOURCE_DIR}/include/${LIB_NAME})

SET(SOURCES_H
    ${HEADER_PATH}/QGraphicsViewAdapter
)

IF ( Qt5Widgets_FOUND )
    QT5_WRAP_CPP( SOURCES_H_MOC ${SOURCES_H} OPTIONS
          "-fosgQt/QGraphicsViewAdapter" )
    set(OSGQT_QT_VERSION 5)
ELSE()
    QT4_WRAP_CPP( SOURCES_H_MOC ${SOURCES_H} OPTIONS "-f" )
    set(OSGQT_QT_VERSION 4)
ENDIF()

SET(OSGQT_VERSION_HEADER "${PROJECT_BINARY_DIR}/include/osgQt/Version")
CONFIGURE_FILE("${CMAKE_CURRENT_SOURCE_DIR}/Version.in" ${OSGQT_VERSION_HEADER})

SET(TARGET_H
    ${HEADER_PATH}/Export
    ${HEADER_PATH}/GraphicsWindowQt
    ${HEADER_PATH}/QFontImplementation
    ${HEADER_PATH}/QGraphicsViewAdapter
    ${HEADER_PATH}/QWidgetImage
    ${HEADER_PATH}/QWebViewImage
    ${OSGQT_VERSION_HEADER}
)


# FIXME: For OS X, need flag for Framework or dylib
SET(TARGET_SRC
    GraphicsWindowQt.cpp
    QFontImplementation.cpp
    QGraphicsViewAdapter.cpp
    QWidgetImage.cpp
    ${SOURCES_H_MOC}
    ${OPENSCENEGRAPH_VERSIONINFO_RC}
)

IF( QT4_FOUND)
    SET(TARGET_SRC ${TARGET_SRC} ReaderQFont.cpp)
ENDIF()

SET(TARGET_LIBRARIES
    osgText
    osgWidget
    osgDB
    osgUtil
    osg
    OpenThreads
)

IF(CMAKE_COMPILER_IS_GNUCXX)
    # Remove -pedantic flag as it barfs on Qt headers
   STRING(REGEX REPLACE "-pedantic" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
ENDIF()

# FIXME: This should work but something with the LINK_WITH_VARIABLES macro is not working
#SET(TARGET_LIBRARIES_VARS
#    QT_QTCORE_LIBRARY 
#    QT_QTGUI_LIBRARY
#    QT_QTOPENGL_LIBRARY
#)
SET(TARGET_LIBRARIES  ${OPENSCENEGRAPH_LIBRARIES})

IF( QT4_FOUND )
    INCLUDE_DIRECTORIES( ${QT_INCLUDE_DIR} ${QT_QTCORE_INCLUDE_DIR} ${QT_QTGUI_INCLUDE_DIR}  ${QT_QTOPENGL_INCLUDE_DIR} )
    SET(TARGET_LIBRARIES
        ${TARGET_LIBRARIES}
        ${QT_QTCORE_LIBRARY} 
        ${QT_QTGUI_LIBRARY}
        ${QT_QTOPENGL_LIBRARY}
    )
ENDIF( QT4_FOUND )

SETUP_LIBRARY(${LIB_NAME})

IF ( Qt5Widgets_FOUND )
    qt5_use_modules( ${LIB_NAME} Widgets OpenGL )
ENDIF ( Qt5Widgets_FOUND )
