# Путь к исходникам движка
set(engine_dir "${CMAKE_SOURCE_DIR}/dviglo2d")

# Указываем минимальную версию CMake
include(${engine_dir}/cmake/cmake_min_version.cmake)
cmake_minimum_required(VERSION ${dv_cmake_min_version})

# Название проекта
project(game)

# Подключаем директорию с движком
include(${engine_dir}/cmake/common.cmake)
add_subdirectory(${engine_dir} dviglo2d)

# Название таргета
set(target_name game)

# Папка для скомпилированной игры
set(result_dir "${CMAKE_BINARY_DIR}/result")

# Создаём папку result
file(MAKE_DIRECTORY ${result_dir})

# Создаём список файлов
file(GLOB_RECURSE source_files src/*.cpp src/*.hpp)

# Указываем папку для следующих скомпилированных экзешников
dv_set_bin_dir(${result_dir})

# Создаём приложение
add_executable(${target_name} ${source_files})

if(NOT DV_WIN32_CONSOLE)
    # Используем точку входа WinMain()
    set_property(TARGET ${target_name} PROPERTY WIN32_EXECUTABLE TRUE)
endif()

# Отладочная версия приложения будет иметь суффикс _d
set_property(TARGET ${target_name} PROPERTY DEBUG_POSTFIX _d)

# Подключаем библиотеку-движок к приложению
target_link_libraries(${target_name} dviglo)

# Копируем динамические библиотеки в папку с приложением
dv_copy_shared_libs_to_bin_dir(${target_name} ${result_dir} copy_shared_libs_to_app_dir)

# Список папок с ресурсами движка для копирования в result
set(dir_names engine_data samples_data)

# Копируем папки с ресурсами движка в result, если нужно
foreach(dir_name ${dir_names})
    if(NOT EXISTS ${result_dir}/${dir_name})
        message("!!!!!!!!! COPY ${engine_dir}/result/${dir_name} TO ${result_dir}/${dir_name}")
        dv_create_dir_link(${engine_dir}/result/${dir_name} ${result_dir}/${dir_name})
    endif()
endforeach()

# Копируем папку с ресурсами игры в result, если нужно
if(NOT EXISTS ${result_dir}/game_data)
    message("!!!!!!!!! COPY ${CMAKE_SOURCE_DIR}/result/game_data TO ${result_dir}/game_data")
    dv_create_dir_link(${CMAKE_SOURCE_DIR}/result/game_data ${result_dir}/game_data)
endif()

# Добавляем приложение в список тестируемых
add_test(NAME ${target_name} COMMAND ${target_name} -duration 5)

# Заставляем Visual Studio отображать дерево каталогов
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR}/src FILES ${source_files})

# В Visual Studio таргет будет назначен стартовым вместо ALL_BUILD,
# чтобы потом не делать это вручную при отладке игры
set_property(DIRECTORY ${CMAKE_SOURCE_DIR} PROPERTY VS_STARTUP_PROJECT ${target_name})
