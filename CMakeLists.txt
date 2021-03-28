cmake_minimum_required (VERSION 3.5)

set (project BadEmailsAutogen)
project (
        ${project}
        DESCRIPTION "Generates C++ header file from the text file"
        LANGUAGES CXX
)

message ("Configuring ${project}...")

# Read providers file as individual strings
file (STRINGS "providers.txt" providers)

# Write the includes, and the set define
set (GENERATED_CPP_FILE "#ifndef BAD_EMAIL_PROVIDERS_GENERATED_H\n#define BAD_EMAIL_PROVIDERS_GENERATED_H\n\n#include <string>\n#include <unordered_set>\n\nstd::unordered_set<std::string> bad_providers({")

# Add all items from the list
foreach (provider ${providers})
    set (GENERATED_CPP_FILE "${GENERATED_CPP_FILE}\n\t\"${provider}\",")
endforeach ()

# Remove last comma
string (LENGTH ${GENERATED_CPP_FILE} generated_length)
math (EXPR generated_length "${generated_length} - 1")
string (SUBSTRING ${GENERATED_CPP_FILE} 0 ${generated_length} GENERATED_CPP_FILE)

# Add closing brcket and semicolon
set (GENERATED_CPP_FILE "${GENERATED_CPP_FILE}\n})\;\n\n#endif")

# Write to output
file (WRITE ${CMAKE_CURRENT_BINARY_DIR}/generated_providers.h ${GENERATED_CPP_FILE})
