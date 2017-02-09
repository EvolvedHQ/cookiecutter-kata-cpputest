cmake_minimum_required (VERSION 2.8)
project ({{ cookiecutter.kata|lower }}-kata)

# Include CppUTest from /lib fodler. The flag is necessary to make it
# use the same flags as the top level (ie this) build
option(CPPUTEST_FLAGS OFF)
add_subdirectory(lib/cpputest)

# For the sake of form, and to give an idea of the sort of setup you'd
# find in a "real world" project, we're keeping the sources separate,
# and creating a separate archive
file (GLOB productionSources src/*.cpp)
add_library ({{ cookiecutter.kata|lower }} STATIC ${productionSources})
target_include_directories ({{ cookiecutter.kata|lower }} PRIVATE ${PROJECT_SOURCE_DIR}/include)

file (GLOB testSources test/*.cpp)
add_executable ({{ cookiecutter.kata|lower }}test ${testSources})
target_link_libraries ({{ cookiecutter.kata|lower }}test {{ cookiecutter.kata|lower }} CppUTest CppUTestExt)
target_include_directories ({{ cookiecutter.kata|lower }}test PRIVATE ${PROJECT_SOURCE_DIR}/lib/cpputest/include)
target_include_directories ({{ cookiecutter.kata|lower }}test PRIVATE ${PROJECT_SOURCE_DIR}/include)

# This enables the "make test" target, although for TDD it's likely
# you'll want to use the "make unit" target.
enable_testing ()

# Add the unit tests executable 
add_test (Unit {{ cookiecutter.kata|lower }}test)

{% if cookiecutter.etags == "y" %}
# Generate the tags file
file (GLOB headers ${PROJECT_SOURCE_DIR}/include/*.hpp)
set_source_files_properties(TAGS PROPERTIES GENERATED true)
add_custom_target(tags
  COMMAND etags --language=c++ --declarations --ignore-indentation
  ${productionSources} ${testSources} ${headers}
  WORKING_DIRECTORY ${CMAKE_BINARY_DIR})
{% endif %}

# And a custom target as a convenience for running "make unit" as a
# specific target to support the TDD cycle
add_custom_target(unit
  COMMAND ${CMAKE_BINARY_DIR}/{{ cookiecutter.kata|lower }}test -c
  DEPENDS {{ cookiecutter.kata|lower }}test {% if cookiecutter.etags == "y" %} tags {% endif %})
