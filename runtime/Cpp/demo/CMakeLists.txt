# -*- mode:cmake -*-
if(NOT UNIX)
  message(FATAL "Unsupported operating system")
endif()

set(CMAKE_C_COMPILER /usr/bin/gcc-5)
set(CMAKE_CXX_COMPILER /usr/bin/g++-5)

set(antlr4-demo-GENERATED_SRC
   ${PROJECT_SOURCE_DIR}/demo/generated/TLexer.cpp 
   ${PROJECT_SOURCE_DIR}/demo/generated/TParser.cpp
   ${PROJECT_SOURCE_DIR}/demo/generated/TParserBaseListener.cpp
   ${PROJECT_SOURCE_DIR}/demo/generated/TParserBaseVisitor.cpp
   ${PROJECT_SOURCE_DIR}/demo/generated/TParserListener.cpp
   ${PROJECT_SOURCE_DIR}/demo/generated/TParserVisitor.cpp
 )

foreach( src_file ${antlr4-demo-GENERATED_SRC} )
      set_source_files_properties(
          ${src_file}
          PROPERTIES
          GENERATED TRUE
          )
endforeach( src_file ${antlr4-demo-GENERATED_SRC} )

add_custom_target(GenerateParser
   COMMAND 
   ${CMAKE_COMMAND} -E make_directory ${PROJECT_SOURCE_DIR}/demo/generated/
   COMMAND
   "${Java_JAVA_EXECUTABLE}" -jar ${ANTLR_JAR_LOCATION} -Werror -Dlanguage=Cpp -listener -visitor -o ${PROJECT_SOURCE_DIR}/demo/generated/ -package antlrcpptest ${PROJECT_SOURCE_DIR}/demo/TLexer.g4 ${PROJECT_SOURCE_DIR}/demo/TParser.g4
   WORKING_DIRECTORY "${CMAKE_BINARY_DIR}"
   DEPENDS ${PROJECT_SOURCE_DIR}/demo/TLexer.g4 ${PROJECT_SOURCE_DIR}/demo/TParser.g4
   )

include_directories(
  ${PROJECT_SOURCE_DIR}/runtime/src
  ${PROJECT_SOURCE_DIR}/runtime/src/misc
  ${PROJECT_SOURCE_DIR}/runtime/src/atn
  ${PROJECT_SOURCE_DIR}/runtime/src/dfa
  ${PROJECT_SOURCE_DIR}/runtime/src/tree
  ${PROJECT_SOURCE_DIR}/runtime/src/support
  ${PROJECT_SOURCE_DIR}/demo/generated
  )

#file(GLOB antlr4-demo_SRC "${PROJECT_SOURCE_DIR}/demo/generated/*")
set(antlr4-demo_SRC 
  ${PROJECT_SOURCE_DIR}/demo/Linux/main.cpp
  ${antlr4-demo-GENERATED_SRC}
  )

foreach( src_file ${antlr4-demo_SRC} )
      set_source_files_properties(
          ${src_file}
          PROPERTIES
          COMPILE_FLAGS -Wno-overloaded-virtual
          )
endforeach( src_file ${antlr4-demo_SRC} )

add_executable(antlr4-demo
  ${antlr4-demo_SRC}
  )
#add_precompiled_header(antlr4-demo ${PROJECT_SOURCE_DIR}/runtime/src/antlrcpp-Prefix.h)

add_dependencies(antlr4-demo GenerateParser)

target_link_libraries(antlr4-demo antlr4_static)

install(TARGETS antlr4-demo 
        DESTINATION "share" 
        COMPONENT dev 
        )

