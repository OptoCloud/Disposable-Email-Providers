cmake_minimum_required(VERSION 3.5)

project(BadEmailsAutogen DESCRIPTION "Generates C++ header file from the text file" LANGUAGES CXX C)

message(STATUS "Configuring ${CMAKE_PROJECT_NAME}...")


# Read providers file as individual strings
file(STRINGS "providers.txt" providers)

list(LENGTH providers providers_len)
list(SORT providers)

# Write the includes, and the set define
set(GENERATED_CPP_FILE
    "#include <array>\n"
    "#include <string_view>\n"
    "\n"
    "static std::array<std::string_view, ${providers_len}> const bad_providers {"
)


# Add all items from the list
foreach(provider ${providers})
    set(GENERATED_CPP_FILE "${GENERATED_CPP_FILE}\n\t\"${provider}\",")
endforeach()

# Remove last comma
# string (LENGTH ${GENERATED_CPP_FILE} generated_length)
# math (EXPR generated_length "${generated_length} - 1")
# string (SUBSTRING ${GENERATED_CPP_FILE} 0 ${generated_length} GENERATED_CPP_FILE)

# Add closing brcket and semicolon
set(GENERATED_CPP_FILE
    "${GENERATED_CPP_FILE}\n"
    "}\;\n"
    "\n"
    "std::string_view const *get_bad_providers(std::size_t *size) noexcept\n"
    "{\n"
    "\tif (size) {\n"
    "\t\t*size = bad_providers.size()\;\n"
    "\t}\n"
    "\treturn bad_providers.data()\;\n"
    "}\n")

set(GENERATED_HPP_FILE
    "#ifndef BAD_EMAIL_PROVIDERS_GENERATED_HPP\n"
    "#define BAD_EMAIL_PROVIDERS_GENERATED_H\n"
    "#include <string_view>\n"
    "\n"
    "std::string_view const *get_bad_providers(std::size_t *size) noexcept\;\n"
    "\n"
    "#endif\n")

# Write to output
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/generated_providers.cpp ${GENERATED_CPP_FILE})
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/bad_emails/generated_providers.hpp ${GENERATED_HPP_FILE})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(${CMAKE_PROJECT_NAME})
target_sources(${CMAKE_PROJECT_NAME}
    PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/generated_providers.cpp
    PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/bad_emails/generated_providers.hpp)
target_include_directories(${CMAKE_PROJECT_NAME} PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/bad_emails/)

 