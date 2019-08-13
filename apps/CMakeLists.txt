include_directories(${CMAKE_SOURCE_DIR}/kfusion/include)

function(add_lz target)
    get_target_property(TEMP ${target} COMPILE_FLAGS)
    if(TEMP STREQUAL "TEMP-NOTFOUND")
        set(TEMP "") # Set to empty string
    else()
        set(TEMP "${TEMP} ") # A space to cleanly separate from existing content
    endif()
    set(TEMP "${TEMP} -lz" )
    set_target_properties(${target} PROPERTIES COMPILE_FLAGS ${TEMP} )
endfunction()

#Adding CombinedSolver because the IDE provides code insight. Not actually needed.
add_executable(dynamicfusion demo.cpp ${CMAKE_SOURCE_DIR}/kfusion/include/opt/CombinedSolver.h)
message(STATUS ${TERRA_LIBRARIES})
set(LINK_LIBS
    ${OpenCV_LIBS}
    ${OPT_LIBRARIES}

    ${CUDA_CUDART_LIBRARY}
    kfusion
    #${ZLIB_LIBRARIES}
    )

add_library(terra_combo INTERFACE)
target_link_libraries(terra_combo INTERFACE ${TERRA_LIBRARIES} ${ZLIB_LIBRARIES} tinfo)


target_link_libraries(dynamicfusion ${LINK_LIBS} terra_combo)

#target_link_libraries(dynamicfusion ${OpenCV_LIBS} ${LINK_LIBS} ${LINK_LIBS})

if(SAVE_RECONSTRUCTION_FRAMES)
    file(MAKE_DIRECTORY ${CMAKE_SOURCE_DIR}/output/)
    target_compile_definitions(dynamicfusion PUBLIC OUTPUT_PATH=${CMAKE_SOURCE_DIR}/output/)
endif()

set_target_properties(dynamicfusion PROPERTIES
    DEBUG_POSTFIX "d"
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")


if(OPENNI_FOUND)
    add_executable(dynamicfusion_kinect dynamicfusion_kinect.cpp)
    set(LINK_LIBS
        #${ZLIB_LIBRARIES}
        ${OPENNI_LIBRARY}
        ${OpenCV_LIBS}
        kfusion
        )
    target_link_libraries(dynamicfusion_kinect ${LINK_LIBS} terra_combo)

    #target_link_libraries(dynamicfusion_kinect ${LINK_LIBS} ${LINK_LIBS})

    set_target_properties(dynamicfusion_kinect PROPERTIES
        DEBUG_POSTFIX "d"
        ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
    install(TARGETS dynamicfusion_kinect RUNTIME DESTINATION bin COMPONENT main)
endif()

install(TARGETS dynamicfusion RUNTIME DESTINATION bin COMPONENT main)
