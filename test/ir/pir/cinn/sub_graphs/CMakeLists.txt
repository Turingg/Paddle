if(WITH_GPU)
  set(STATIC_BUILD_TESTS
      test_sub_graph_42 test_sub_graph_squeeze_unsqueeze test_sub_graph_56
      test_sub_graph_73 test_sub_graph_79 test_sub_graph_82)

  file(
    GLOB DYNAMIC_BUILD_TESTS
    RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}"
    "test_*.py")
  string(REPLACE ".py" "" DYNAMIC_BUILD_TESTS "${DYNAMIC_BUILD_TESTS}")
  #list(REMOVE_ITEM DYNAMIC_BUILD_TESTS STATIC_BUILD_TESTS)

  foreach(static_test ${STATIC_BUILD_TESTS})
    list(REMOVE_ITEM DYNAMIC_BUILD_TESTS ${static_test})
  endforeach()

  foreach(cinn_sub_graph_test_name ${STATIC_BUILD_TESTS})
    add_test(
      NAME ${cinn_sub_graph_test_name}
      COMMAND
        ${CMAKE_COMMAND} -E env
        PYTHONPATH=${CMAKE_BINARY_DIR}:${CMAKE_BINARY_DIR}/python/:$ENV{PYTHONPATH}
        FLAGS_cinn_new_group_scheduler=1 FLAGS_enable_pir_api=1
        FLAGS_cinn_bucket_compile=1 FLAGS_group_schedule_tiling_first=1
        FLAGS_cudnn_deterministic=true ${PYTHON_EXECUTABLE}
        ${CMAKE_CURRENT_SOURCE_DIR}/${cinn_sub_graph_test_name}.py
      WORKING_DIRECTORY ${CMAKE_BINARY_DIR})
    set_tests_properties(${cinn_sub_graph_test_name} PROPERTIES LABELS
                                                                "RUN_TYPE=CINN")
    set_tests_properties(${cinn_sub_graph_test_name} PROPERTIES TIMEOUT 300)
  endforeach()

  foreach(cinn_sub_graph_test_name ${DYNAMIC_BUILD_TESTS})
    add_test(
      NAME ${cinn_sub_graph_test_name}
      COMMAND
        ${CMAKE_COMMAND} -E env
        PYTHONPATH=${CMAKE_BINARY_DIR}:${CMAKE_BINARY_DIR}/python/:$ENV{PYTHONPATH}
        FLAGS_prim_enable_dynamic=1 FLAGS_cinn_new_group_scheduler=1
        FLAGS_check_infer_symbolic=1 FLAGS_enable_pir_api=1
        FLAGS_cinn_bucket_compile=1 FLAGS_group_schedule_tiling_first=1
        FLAGS_cudnn_deterministic=true ${PYTHON_EXECUTABLE}
        ${CMAKE_CURRENT_SOURCE_DIR}/${cinn_sub_graph_test_name}.py
      WORKING_DIRECTORY ${CMAKE_BINARY_DIR})
    set_tests_properties(${cinn_sub_graph_test_name} PROPERTIES LABELS
                                                                "RUN_TYPE=CINN")
    set_tests_properties(${cinn_sub_graph_test_name} PROPERTIES TIMEOUT 600)
  endforeach()

  # set_tests_properties(test_sub_graph_3 PROPERTIES TIMEOUT 300)
  # set_tests_properties(test_sub_graph_54 PROPERTIES TIMEOUT 300)
  # set_tests_properties(test_sub_graph_30 PROPERTIES TIMEOUT 300)

endif()
