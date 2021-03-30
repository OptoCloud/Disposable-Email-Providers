cmake_minimum_required(VERSION 3.5)

set(project BadEmailProviders)
project(${project} DESCRIPTION "Generates C++ header file from the text file" LANGUAGES CXX C)

message(STATUS "Configuring ${project}...")


# Read providers file as individual strings
file(STRINGS "providers.txt" providers)

list(LENGTH providers providers_len)
list(SORT providers)

# Write the includes, and the set define
set(GENERATED_CPP_FILE
    "#include \"bad_providers_generated.h\"\n"
    "\n"
    "#include <array>\n"
    "#include <algorithm>\n"
    "\n"
    "static std::array<std::string_view, ${providers_len}> const bad_providers {"
)

# Add all items from the list
foreach(provider ${providers})
    set(GENERATED_CPP_FILE "${GENERATED_CPP_FILE}\n\t\"${provider}\",")
endforeach()

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
    "}\n"
    "\n"
    "bool is_bad_provider(std::string_view provider) noexcept\n"
    "{\n"
    "\treturn std::binary_search(bad_providers.begin(), bad_providers.end(), provider)\;\n"
    "}\n")

set(GENERATED_H_FILE
    "#ifndef BAD_PROVIDERS_GENERATED_H\n"
    "#define BAD_PROVIDERS_GENERATED_H\n"
    "\n"
    "#include <string_view>\n"
    "\n"
    "std::string_view const *get_bad_providers(std::size_t *size) noexcept\;\n"
    "bool is_bad_provider(std::string_view provider) noexcept\;\n"
    "\n"
    "#endif // BAD_EMAIL_PROVIDERS_GENERATED_H\n")

# Write to output
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/bad_providers_generated.cpp ${GENERATED_CPP_FILE})
file(WRITE ${CMAKE_CURRENT_BINARY_DIR}/bad_email_providers/bad_providers_generated.h ${GENERATED_H_FILE})

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(${project})
target_sources(${project}
    PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/bad_providers_generated.cpp
    PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/bad_email_providers/bad_providers_generated.h)
target_include_directories(${project} PUBLIC ${CMAKE_CURRENT_BINARY_DIR}/bad_email_providers/)

 
