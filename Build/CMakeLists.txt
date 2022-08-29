#
#Recordar que variables como STATIC, CMAKE_BUILD_TYPE y alguna más, requiere de un CMAKE limpio para que surtan efecto, por lo menos borrado el CMakeCache.txt
#
cmake_minimum_required(VERSION 3.5)
project(FotoVision2 VERSION 0.1 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#
#modificar según necesidades
#

set(DESKTOP ON) #INDICA SI SE TRATA DE UNA APLICACIÓN DE ESCRITORIO O CONSOLA
set (STATIC ON) # PARA COMPILACIÓN de forma statíca o dinámica
set (CMAKE_BUILD_TYPE Release)

#Las variables PATH_LIB_STATIC y PATH_LIB_DINAMIC se agregarán a CMAKE_PREFIX_PATH, que es utilizada por find_package() find_library() etc
#cuando funcionan en modo configuración.  Recordar que en modo configuración también se busca en las variables de entorno.
#Se pueden agregar varias rutas separadas por ; Así se puede tener acceso a librerías de diferentes frameworks, biliotecas, etc.
#La ruta debe llevar hasta el directorio raiz desde el cual cuelgan los subdirectorios include, lib, bin. 
#Ya los métodos find_package() find_library().. en modo configuración encontrarán lo necesario.
#encontrarán lo que necesitan.
#
set (PATH_LIB_STATIC C:/Qt/6.3.1/Build_Static/mingw1120_64)
set (PATH_LIB_DINAMIC C:/Qt/6.3.1/mingw_64)


set (QT_ENABLE ON) #Para habilitar librerías y opciones del framework QT
set(MODULES_QT  #No tiene efecto si no se utiliza QT.
    Widgets
    Core5Compat
    Charts
)

#Puede que necesite incluir alguna ruta más para la búsqueda de cabeceras, en caso de agregar bibliotecas externas
set(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH};${CMAKE_SOURCE_DIR}/../src/include)
set(EXTERN_LIB #RUTAS DE ACCESO A BINARIOS BIBLIOTECAS EXTERNAS DE MANERA DIRECTA, SIN USAR FIND_PACKAGE(). TAMBIÉN HABRÁ QUE ASEGURARSE QUE ENCUETRA SU INCLUDE
    #ACTUALIZANDO LA LÍNEA ANTERIOR CMAKE_INCLUDE_PATH
)


#Puede que necesite incluir alguna ruta más para la búsqueda de módulos. Esto es para find_package() en modo módulo. Primero se buscan en las 
#rutas indicadas en esta variable y después en los módulos de la propio instalación de CMake.
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH};${CMAKE_SOURCE_DIR}/cmake)


#Aquí se agregan todos los tipos de ficheros fuentes

set(PROJECT_SOURCES_CPP
        ${CMAKE_SOURCE_DIR}/../src/cpp/main.cpp
        ${CMAKE_SOURCE_DIR}/../src/cpp/mainwindow.cpp
        ${CMAKE_SOURCE_DIR}/../src/cpp/image.cpp
        ${CMAKE_SOURCE_DIR}/../src/cpp/events.cpp
        ${CMAKE_SOURCE_DIR}/../src/cpp/brilloycontraste.cpp
)

set(PROJECT_SOURCES_HEADERS
    ${CMAKE_SOURCE_DIR}/../src/include/mainwindow.h
    ${CMAKE_SOURCE_DIR}/../src/include/brilloycontraste.h
    ${CMAKE_SOURCE_DIR}/../src/include/events.h
    ${CMAKE_SOURCE_DIR}/../src/include/image.h


)

set(PROJECT_SOURCES_FORMS
    ${CMAKE_SOURCE_DIR}/../src/forms/mainwindow.ui
    ${CMAKE_SOURCE_DIR}/../src/forms/brilloycontraste.ui

)

set(PROJECT_SOURCES_QRC
    ${CMAKE_SOURCE_DIR}/../src/qrc/recursos.qrc
)


#A partir de aquí, no hay que tocar nada

if(STATIC)
   set(CMAKE_PREFIX_PATH ${PATH_LIB_STATIC};${PATH_LIB_DINAMIC})
else()
   set(CMAKE_PREFIX_PATH ${PATH_LIB_DINAMIC})
endif()

# Let's nicely support folders in IDEs
set_property(GLOBAL PROPERTY USE_FOLDERS ON)
set(CMAKE_INCLUDE_CURRENT_DIR ON)


set(ARQUITECTURE 32Bits)

if (CMAKE_SIZEOF_VOID_P EQUAL 8)
    set(ARQUITECTURE 64Bits)

    elseif(CMAKE_SIZEOF_VOID_P EQUL 4)
    set(ARQUITECTURE 32Bits)

endif()

set(BIN ${CMAKE_CURRENT_SOURCE_DIR}/../Bin/${PROJECT_NAME}_${CMAKE_SYSTEM_NAME}_${ARQUITECTURE}_${CMAKE_BUILD_TYPE}_${CMAKE_CXX_COMPILER_ID}_${CMAKE_CXX_COMPILER_VERSION})

if (QT_ENABLE)

    set(CMAKE_AUTOUIC ON)
    set(CMAKE_AUTOMOC ON)
    set(CMAKE_AUTORCC ON)
    set(CMAKE_AUTOUIC_SEARCH_PATHS ${CMAKE_SOURCE_DIR}/../src/forms)
    set(CMAKE_INCLUDE_PATH ${CMAKE_SOURCE_DIR}/../src/include)

    find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS ${MODULES_QT})
    find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS ${MODULES_QT})

    foreach(MODULO ${MODULES_QT})
        message(STATUS "\n\nAgregando rutas include de: Qt${QT_VERSION_MAJOR}${MODULO}\nCon valor: ${Qt${QT_VERSION_MAJOR}${MODULO}_INCLUDE_DIRS}")
        set(CMAKE_INCLUDE_PATH ${CMAKE_INCLUDE_PATH};${Qt${QT_VERSION_MAJOR}${MODULO}_INCLUDE_DIRS})
    endforeach()

    set (BIN ${BIN}_with_Qt${QT_VERSION_MAJOR}.${QT_VERSION_MINOR})


endif()

add_executable(${PROJECT_NAME}
    ${PROJECT_SOURCES_CPP}
    ${PROJECT_SOURCES_HEADERS}
    ${PROJECT_SOURCES_FORMS}
    ${PROJECT_SOURCES_QRC}
)

target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_INCLUDE_PATH} )

#Agrego módulos de QT.
if (QT_ENABLE)
    foreach(MODULO ${MODULES_QT})
        message(STATUS "\n\nAgregando Binarios de biblioteca:  ${Qt${QT_VERSION_MAJOR}${MODULO}_LIBRARIES}")
        target_link_libraries(${PROJECT_NAME} PRIVATE ${Qt${QT_VERSION_MAJOR}${MODULO}_LIBRARIES})
    endforeach()

endif()

#Agrego librerías Externas necesarias
foreach(MODULO ${EXTERN_LIB})
    target_link_libraries(${PROJECT_NAME} PUBLIC ${EXTERN_LIB})
endforeach()


if (STATIC)
        set(BIN ${BIN}_STATIC)
        target_link_libraries(${PROJECT_NAME} PUBLIC -static-libgcc -static-libstdc++)
        if (MINGW)
            set (CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-Bstatic,--whole-archive -lwinpthread -Wl,--no-whole-archive")
        endif()
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE ${DESKTOP}
    WIN32_EXECUTABLE ${DESKTOP}
)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${BIN})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${BIN})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${BIN})
set(EXECUTABLE_OUTPUT_PATH ${BIN} CACHE PATH "BIN directory" FORCE)
set(LIBRARY_OUTPUT_PATH ${BIN} CACHE PATH "BIN directory" FORCE)


message(STATUS "\n\nBusqueda inicial para find_package(), find_program(), find_file() find_library() y find_path() en CMAKE_PREFIX_PATH: ${CMAKE_PREFIX_PATH}")
message(NOTICE "\n\nRutas para ficheros .h,hpp(cabecera/include) CMAKE_INCLUDE_PATH: \n${CMAKE_INCLUDE_PATH}")
message(NOTICE "\n\nRutas para ficheros .CMAKE(plugins/modules) CMAKE_MODULE_PATH:\n${CMAKE_MODULE_PATH}")

message(STATUS "\n\nTipo de compilado CMAKE_BUILD_TYPE:  ${CMAKE_BUILD_TYPE}")
message(STATUS "\n\nEnlace de bibliotecas estático:  ${STATIC}")
message(STATUS "Directorio para contruir CMAKE_BINARY_DIR: ${CMAKE_BINARY_DIR}")
message(STATUS "Binario en: ${BIN}")


