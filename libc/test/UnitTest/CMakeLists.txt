set(libc_test_srcs_common
  Test.h
  LibcTest.cpp
  CmakeFilePath.cpp
  LibcTest.h
  TestLogger.cpp
  TestLogger.h
)

set(libc_death_test_srcs ExecuteFunction.h)
if(${LIBC_TARGET_OS} STREQUAL "linux")
  list(APPEND libc_death_test_srcs 
       LibcDeathTestExecutors.cpp ExecuteFunctionUnix.cpp)
endif()

add_library(
  LibcUnitTest
  EXCLUDE_FROM_ALL
  ${libc_test_srcs_common}
  ${libc_death_test_srcs}
)

add_library(
  LibcHermeticTest
  EXCLUDE_FROM_ALL
  ${libc_test_srcs_common}
  HermeticTestUtils.cpp
)

add_library(
  LibcUnitTestMain
  EXCLUDE_FROM_ALL
  LibcTestMain.cpp
)
add_dependencies(LibcUnitTestMain LibcUnitTest)

add_library(
  LibcHermeticTestMain
  EXCLUDE_FROM_ALL
  LibcTestMain.cpp
)
add_dependencies(LibcHermeticTestMain LibcHermeticTest)

foreach(lib LibcUnitTest LibcUnitTestMain LibcHermeticTest LibcHermeticTestMain)
  target_include_directories(${lib} PUBLIC ${LIBC_SOURCE_DIR})
  target_compile_options(${lib} PRIVATE -fno-exceptions -fno-rtti)
  add_dependencies(${lib}
    libc.src.__support.c_string
    libc.src.__support.CPP.string
    libc.src.__support.CPP.string_view
    libc.src.__support.CPP.type_traits
    libc.src.__support.OSUtil.osutil
    libc.src.__support.uint128
  )
endforeach()

foreach(lib LibcHermeticTest LibcHermeticTestMain)
  target_include_directories(${lib} PRIVATE ${LIBC_BUILD_DIR}/include)
  target_compile_options(${lib}
      PRIVATE ${LIBC_HERMETIC_TEST_COMPILE_OPTIONS} -ffreestanding -nostdlib -nostdlib++)
endforeach()

add_header_library(
  string_utils
  HDRS
  StringUtils.h
  DEPENDS
    libc.src.__support.CPP.string
    libc.src.__support.CPP.type_traits
)

add_library(
  LibcFPTestHelpers
  FPExceptMatcher.cpp
  FPExceptMatcher.h
  FPMatcher.h
  RoundingModeUtils.cpp
  RoundingModeUtils.h
)
add_dependencies(
  LibcFPTestHelpers
  LibcUnitTest
  libc.test.UnitTest.string_utils
  libc.src.__support.FPUtil.fp_bits
  libc.src.__support.FPUtil.fenv_impl
  libc.test.UnitTest.string_utils
)

add_library(
  LibcMemoryHelpers
  MemoryMatcher.h
  MemoryMatcher.cpp
)
add_dependencies(
  LibcMemoryHelpers
  LibcUnitTest
  libc.src.__support.CPP.span
)

add_library(
  LibcPrintfHelpers
  PrintfMatcher.h
  PrintfMatcher.cpp
)
add_dependencies(
  LibcPrintfHelpers
  LibcUnitTest
  libc.src.__support.FPUtil.fp_bits
  libc.src.stdio.printf_core.core_structs
  libc.test.UnitTest.string_utils
)

add_library(
  LibcScanfHelpers
  ScanfMatcher.h
  ScanfMatcher.cpp
)
add_dependencies(
  LibcScanfHelpers
  LibcUnitTest
  libc.src.__support.FPUtil.fp_bits
  libc.src.stdio.scanf_core.core_structs
  libc.test.UnitTest.string_utils
)

foreach(lib LibcFPTestHelpers LibcMemoryHelpers LibcPrintfHelpers LibcScanfHelpers)
  target_include_directories(${lib} PUBLIC ${LIBC_SOURCE_DIR})
  target_compile_options(${lib} PRIVATE -fno-exceptions -fno-rtti)
target_link_libraries(${lib} LibcUnitTest)
endforeach()
