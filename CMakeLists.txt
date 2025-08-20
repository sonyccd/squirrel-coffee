cmake_minimum_required(VERSION 3.10)
project(squirrel_coffee)

# Set C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Find required packages
find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)

# Include directories
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/ast)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/ast/Declaration)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/ast/Expression)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/ast/Statement)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

# Generate parser and lexer files
BISON_TARGET(Parser 
    tjc.y 
    ${CMAKE_CURRENT_BINARY_DIR}/tjc.tab.cpp
    DEFINES_FILE ${CMAKE_CURRENT_BINARY_DIR}/tjc.tab.h
    COMPILE_FLAGS "-y -rall -fcaret -t"
)

FLEX_TARGET(Lexer 
    tjc.l  
    ${CMAKE_CURRENT_BINARY_DIR}/lex.yy.cpp
)

ADD_FLEX_BISON_DEPENDENCY(Lexer Parser)

# Collect all AST source files
file(GLOB AST_SOURCES 
    "ast/*.cpp"
    "ast/Declaration/*.cpp"
    "ast/Expression/*.cpp"
    "ast/Statement/*.cpp"
)

# Create AST static library
add_library(Ast STATIC ${AST_SOURCES})
target_include_directories(Ast PUBLIC 
    ${CMAKE_CURRENT_SOURCE_DIR}/ast
    ${CMAKE_CURRENT_SOURCE_DIR}/ast/Declaration
    ${CMAKE_CURRENT_SOURCE_DIR}/ast/Expression
    ${CMAKE_CURRENT_SOURCE_DIR}/ast/Statement
)

# Main compiler executable
set(COMPILER_SOURCES
    main_tjc.cpp
    AstPrinter.cpp
    SymbolTable.cpp
    SemanticCheck.cpp
    CodeGenerator.cpp
    ${BISON_Parser_OUTPUTS}
    ${FLEX_Lexer_OUTPUTS}
)

add_executable(tjc ${COMPILER_SOURCES})

# Link libraries
target_link_libraries(tjc 
    Ast
    ${FLEX_LIBRARIES}
)

# Set output directory for the executable
set_target_properties(tjc PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

# Installation rules (optional)
install(TARGETS tjc DESTINATION bin)

# Add custom target for cleaning generated files
add_custom_target(clean-generated
    COMMAND ${CMAKE_COMMAND} -E remove 
        ${CMAKE_CURRENT_BINARY_DIR}/tjc.tab.cpp
        ${CMAKE_CURRENT_BINARY_DIR}/tjc.tab.h
        ${CMAKE_CURRENT_BINARY_DIR}/lex.yy.cpp
    COMMENT "Cleaning generated parser and lexer files"
)

# Print configuration summary
message(STATUS "")
message(STATUS "Configuration Summary:")
message(STATUS "  C++ Compiler: ${CMAKE_CXX_COMPILER}")
message(STATUS "  C++ Standard: C++${CMAKE_CXX_STANDARD}")
message(STATUS "  Bison: ${BISON_EXECUTABLE}")
message(STATUS "  Flex: ${FLEX_EXECUTABLE}")
message(STATUS "  Build Type: ${CMAKE_BUILD_TYPE}")
message(STATUS "")