# Copyright (c) 2018-present, Facebook, Inc. and its affiliates.
# All rights reserved.
#
# This source code is licensed under the BSD-style license found in the
# LICENSE file in the root directory of this source tree.

include_directories(${CMAKE_BINARY_DIR})
link_directories("${CMAKE_BINARY_DIR}/staging/usr/local/lib")

set(_adminapi_if_include_prefix "logdevice/admin/if")
set(_adminapi_binary_dir "${CMAKE_BINARY_DIR}/logdevice/admin/if")

file(MAKE_DIRECTORY ${CMAKE_BINARY_DIR}/${_adminapi_if_include_prefix})

set(_common_libs
  "maintenance-cpp2"
  "safety-cpp2"
  "settings-cpp2"
  "nodes-cpp2"
  "cluster_membership-cpp2"
  "admin_commands-cpp2"
  "logtree-cpp2"
  "common-cpp2"
  "exceptions-cpp2"
  )

if(NOT ${BUILD_SUBMODULES})
  list(APPEND _common_libs FBThrift::thriftcpp2)
endif()

foreach(THRIFT_SOURCE
        exceptions
        settings
        logtree
        nodes
        cluster_membership
        admin_commands
        safety
        maintenance
)

  ld_thrift_py3_library(
    ${THRIFT_SOURCE}
    ""
    "json"
    "${CMAKE_CURRENT_SOURCE_DIR}"
    "${_adminapi_binary_dir}"
    "${_adminapi_if_include_prefix}"
    PY3_NAMESPACE "logdevice/admin"
    CYTHON_INCLUDES
      "${CMAKE_BINARY_DIR}/logdevice/common/if/gen-py3/"
      "${CMAKE_BINARY_DIR}/logdevice/admin/if/gen-py3/"
      "${CMAKE_BINARY_DIR}/logdevice/common/membership/gen-py3/"
  )

  add_dependencies(${THRIFT_SOURCE}-cpp2-target fbthrift)
  target_link_libraries(${THRIFT_SOURCE}-cpp2 common-cpp2 Membership-cpp2)

  if(thriftpy3)
    target_link_libraries(
      "logdevice-admin-${THRIFT_SOURCE}-types-py3"
      PRIVATE
        admin-cpp2
        ${_common_libs})
  endif()
endforeach()

ld_thrift_py3_library(
  "admin"
  "AdminAPI"
  "json"
  "${CMAKE_CURRENT_SOURCE_DIR}"
  "${CMAKE_BINARY_DIR}/logdevice/admin/if"
  "logdevice/admin/if"
  PY3_NAMESPACE "logdevice"
  CYTHON_INCLUDES "${CMAKE_BINARY_DIR}/logdevice/admin/if/gen-py3/"
    "${CMAKE_BINARY_DIR}/logdevice/common/if/gen-py3/"
    "${CMAKE_BINARY_DIR}/common/fb303/if/gen-py3"
    "${CMAKE_BINARY_DIR}/logdevice/admin/gen-py3"
    "${CMAKE_BINARY_DIR}/logdevice"
    "${CMAKE_BINARY_DIR}/fbthrift-prefix/src/fbthrift-build/thrift/lib/py3/cybld"
    "${CMAKE_BINARY_DIR}/logdevice/common/membership/gen-py3/"
  THRIFT_INCLUDE_DIRECTORIES
    "${FB303_INCLUDE_DIR}"
)

set(_languages cpp2)

if(thriftpy3)
  list(APPEND _languages py3)
  set_source_files_properties(
    "${CMAKE_BINARY_DIR}/common/fb303/if/gen-py3/fb303/clients_wrapper.cpp"
    PROPERTIES GENERATED TRUE
  )

  target_sources(
    "logdevice-admin-clients-py3"
    PRIVATE
      "${CMAKE_BINARY_DIR}/common/fb303/if/gen-py3/fb303/clients_wrapper.cpp")

  target_link_libraries(
    "logdevice-admin-clients-py3"
    PRIVATE
      admin-cpp2
      ${_common_libs})

  add_dependencies(admin-py3-target admin-cpp2)
  add_dependencies(logdevice-admin-types-py3
    fb303-types-py3
    logdevice-admin-nodes-types-py3
    logdevice-admin-cluster_membership-types-py3
    logdevice-admin-admin_commands-types-py3
    logdevice-admin-exceptions-types-py3
    logdevice-admin-logtree-types-py3
    logdevice-admin-safety-types-py3
    logdevice-admin-maintenance-types-py3
    logdevice-admin-settings-types-py3)

  add_dependencies(logdevice-admin-clients-py3
    fb303-clients-py3
    admin-py3-target
  )
endif()

add_dependencies(admin-cpp2-target fbthrift)

target_link_libraries(admin-cpp2
  ${_common_libs}
  ${THRIFT_DEPS}
  "fb303-cpp2"
)

foreach(_lang ${_languages})
add_dependencies(admin-${_lang}-target
                 nodes-${_lang}-target
                 cluster_membership-${_lang}-target
                 admin_commands-${_lang}-target
                 fb303-${_lang}-target
                 exceptions-${_lang}-target
                 logtree-${_lang}-target
                 safety-${_lang}-target
                 maintenance-${_lang}-target
                 settings-${_lang}-target)

add_dependencies(exceptions-${_lang}-target
                 common-${_lang}-target)
add_dependencies(logtree-${_lang}-target
                 common-${_lang}-target)
add_dependencies(maintenance-${_lang}-target
                 common-${_lang}-target
                 nodes-${_lang}-target
                 safety-${_lang}-target)
add_dependencies(nodes-${_lang}-target
                 Membership-${_lang}-target
                 common-${_lang}-target)
add_dependencies(safety-${_lang}-target
                 common-${_lang}-target
                 nodes-${_lang}-target)
add_dependencies(settings-${_lang}-target
                 common-${_lang}-target)
add_dependencies(cluster_membership-${_lang}-target
                 common-${_lang}-target
                 nodes-${_lang}-target)
endforeach()
