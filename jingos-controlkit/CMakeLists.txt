cmake_minimum_required(VERSION 3.5)

set(KF5_VERSION "5.76.0") # handled by release scripts
set(KF5_DEP_VERSION "5.76.0") # handled by release scripts

project(jingoscontrolkit1.0 VERSION ${KF5_VERSION})

set(REQUIRED_QT_VERSION 5.12.0)

SET(CMAKE_CXX_STANDARD 11)

################# Disallow in-source build #################

if("${CMAKE_SOURCE_DIR}" STREQUAL "${CMAKE_BINARY_DIR}")
   message(FATAL_ERROR "jingoscontrolkit requires an out of source build. Please create a separate build directory and run 'cmake path_to_jingoscontrolkit [options]' there.")
endif()

option(BUILD_SHARED_LIBS "Build a shared module" ON)
option(DESKTOP_ENABLED "Build and install The Desktop style" ON)
option(BUILD_EXAMPLES "Build and install examples" OFF)
option(DISABLE_DBUS "Build without D-Bus support" OFF)
if(DEFINED STATIC_LIBRARY)
    message(FATAL_ERROR "Use the BUILD_SHARED_LIBS=OFF option to build a static library, STATIC_LIBRARY is no longer a supported option")
endif()

# Make CPack available to easy generate binary packages
include(CPack)
include(FeatureSummary)

find_package(Qt5 ${REQUIRED_QT_VERSION} REQUIRED NO_MODULE COMPONENTS Core Quick Gui Svg QuickControls2 Concurrent)
if (BUILD_TESTING)
    find_package(Qt5QuickTest ${REQUIRED_QT_VERSION} CONFIG QUIET)
endif()
add_definitions(-DQT_NO_FOREACH)
set(CMAKE_AUTOMOC ON)
set(AUTOMOC_MOC_OPTIONS -Muri=com.jingos.controlkit)
set(CMAKE_INCLUDE_CURRENT_DIR ON)

if(NOT BUILD_SHARED_LIBS)
    add_definitions(-DJINGOSCONTROLKIT_BUILD_TYPE_STATIC)
    add_definitions(-DQT_PLUGIN)
    add_definitions(-DQT_STATICPLUGIN=1)
endif()

################# set KDE specific information #################

find_package(ECM 5.76.0 NO_MODULE)
set_package_properties(ECM PROPERTIES TYPE REQUIRED DESCRIPTION "Extra CMake Modules." URL "https://commits.kde.org/extra-cmake-modules")

# where to look first for cmake modules, before ${CMAKE_ROOT}/Modules/ is checked
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${ECM_MODULE_PATH})

include(GenerateExportHeader)
include(ECMSetupVersion)
include(ECMGenerateHeaders)
include(CMakePackageConfigHelpers)
include(ECMPoQmTools)
include(ECMQMLModules)
include(KDEInstallDirs)
include(KDECMakeSettings)
include(ECMQtDeclareLoggingCategory)
include(ECMAddQch)
include(KDEFrameworkCompilerSettings NO_POLICY_SCOPE)
include(KDEPackageAppTemplates)

set(CMAKECONFIG_INSTALL_DIR "${KDE_INSTALL_CMAKEPACKAGEDIR}/JingOSControlKit1.0")

option(BUILD_QCH "Build API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)" OFF)
add_feature_info(QCH ${BUILD_QCH} "API documentation in QCH format (for e.g. Qt Assistant, Qt Creator & KDevelop)")

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/JingOSControlKit1Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/JingOSControlKit1Config.cmake"
    INSTALL_DESTINATION  ${CMAKECONFIG_INSTALL_DIR}
    PATH_VARS  KF5_INCLUDE_INSTALL_DIR CMAKE_INSTALL_PREFIX
)

install(FILES
    "${CMAKE_CURRENT_BINARY_DIR}/JingOSControlKit1Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/JingOSControlKit1ConfigVersion.cmake"
    "${CMAKE_CURRENT_SOURCE_DIR}/JingOSControlKit1Macros.cmake"
    DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
    COMPONENT Devel
)

#install(EXPORT JingOSControlKit1Targets
#    DESTINATION "${CMAKECONFIG_INSTALL_DIR}"
#    FILE JingOSControlKit1Targets.cmake
#    NAMESPACE JOS1::
#)


ecm_setup_version(${KF5_VERSION}
    VARIABLE_PREFIX JINGOSCONTROLKIT1
    VERSION_HEADER "${CMAKE_CURRENT_BINARY_DIR}/jingoscontrolkit_version.h"
    PACKAGE_VERSION_FILE "${CMAKE_CURRENT_BINARY_DIR}/JingOSControlKit1ConfigVersion.cmake"
    SOVERSION 5
)

#use dbus on linux, bsd etc, but not andoid and apple stuff
if (UNIX AND NOT ANDROID AND NOT(APPLE) AND NOT(DISABLE_DBUS))
    find_package(Qt5DBus)
    add_definitions(-DJINGOSCONTROLKIT_ENABLE_DBUS)
endif()

if(BUILD_EXAMPLES AND ANDROID)
    find_package(Qt5AndroidExtras ${REQUIRED_QT_VERSION} REQUIRED)
endif()


ecm_find_qmlmodule(QtGraphicalEffects 1.0)

add_definitions(-DQT_DISABLE_DEPRECATED_BEFORE=0x050d00)
#add_subdirectory(src)
#if (NOT ANDROID)
#    add_subdirectory(templates)
#endif()

#if (BUILD_EXAMPLES AND BUILD_SHARED_LIBS)
#    add_subdirectory(examples)
#endif()

#if (BUILD_TESTING AND BUILD_SHARED_LIBS)
#    add_subdirectory(autotests)
#endif()

#if (IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/po")
#    ecm_install_po_files_as_qm(po)
#endif()

feature_summary(WHAT ALL INCLUDE_QUIET_PACKAGES FATAL_ON_MISSING_REQUIRED_PACKAGES)
