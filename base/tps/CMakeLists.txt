project(tps Java)

add_subdirectory(src)

# install files
add_subdirectory(setup)
add_subdirectory(shared/conf)
add_subdirectory(${APP_SERVER})

# TPS does not support admin console.
# Create links in /usr/share/pki/tps/webapps/tps/WEB-INF/lib.
# This can be customized for different platforms in RPM spec.

add_custom_target(pki-tps-links ALL
    COMMENT "Creating links for TPS")

add_custom_command(
    TARGET pki-tps-links
    COMMAND ${CMAKE_COMMAND} -E make_directory webapp/lib
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${SLF4J_API_JAR} webapp/lib/slf4j-api.jar
    COMMAND ${CMAKE_COMMAND} -E create_symlink ${SLF4J_JDK14_JAR} webapp/lib/slf4j-jdk14.jar
    COMMAND ln -sf /usr/share/java/pki/pki-certsrv.jar ${CMAKE_CURRENT_BINARY_DIR}/webapp/lib/pki-certsrv.jar
    COMMAND ln -sf /usr/share/java/pki/pki-cms.jar ${CMAKE_CURRENT_BINARY_DIR}/webapp/lib/pki-cms.jar
    COMMAND ln -sf /usr/share/java/pki/pki-cmsbundle.jar ${CMAKE_CURRENT_BINARY_DIR}/webapp/lib/pki-cmsbundle.jar
    COMMAND ln -sf /usr/share/java/pki/pki-tps.jar ${CMAKE_CURRENT_BINARY_DIR}/webapp/lib/pki-tps.jar
)

add_custom_target(pki-tps-man ALL
    COMMENT "Creating TPS manuals")

add_custom_command(
    TARGET pki-tps-man
    COMMAND ${CMAKE_COMMAND} -E make_directory ${CMAKE_CURRENT_BINARY_DIR}/man/man5
    COMMAND go-md2man -in ${CMAKE_SOURCE_DIR}/docs/manuals/man5/pki-tps-connector.5.md -out ${CMAKE_CURRENT_BINARY_DIR}/man/man5/pki-tps-connector.5
    COMMAND go-md2man -in ${CMAKE_SOURCE_DIR}/docs/manuals/man5/pki-tps-profile.5.md -out ${CMAKE_CURRENT_BINARY_DIR}/man/man5/pki-tps-profile.5
)

# install directories
install(
    DIRECTORY
        shared/
    DESTINATION
        ${SHARE_INSTALL_PREFIX}/${APPLICATION_NAME}/${PROJECT_NAME}
    PATTERN
        "CMakeLists.txt" EXCLUDE
    PATTERN
        listappletdates EXCLUDE
)

install(
    DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}/webapp/lib/
    DESTINATION
        ${DATA_INSTALL_DIR}/tps/webapps/tps/WEB-INF/lib
)

# install empty directories
install(
    DIRECTORY
    DESTINATION
        ${VAR_INSTALL_DIR}/lock/pki/tps
)

install(
    DIRECTORY
    DESTINATION
        ${VAR_INSTALL_DIR}/run/pki/tps
)

# install man pages generated by md2man
install(
    DIRECTORY
        ${CMAKE_CURRENT_BINARY_DIR}/man/
    DESTINATION
        ${MAN_INSTALL_DIR}
    FILE_PERMISSIONS
        OWNER_READ OWNER_WRITE
        GROUP_READ
        WORLD_READ
)
