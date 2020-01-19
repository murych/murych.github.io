---
title: "[cmake] Генерируем Doxygen из CMake"
categories: cmake, c
---

# Постановка задачи

[Doxygen](http://www.doxygen.nl/) -- утилита для автоматического генерирования документации к исходному коду, обычно на языках C/C++, Java и др.

Обсуждать _необходимость_ написания документации, на мой взгляд, нет никакой _необходимости_.
Хотя бы минимальная, для понимания контекста выполнения той или иной функции, но она должна присутствовать.

В этом посте рассмотрим, что нужно для генерирования пакета документации на код в формате `*.html`.

# Подготовка конфига Doxygen

Допустим, у нас есть проект, написанный на C, исходники лежат в директории `Src`, заголовочные файлы в директории `Inc`.
Также в корне проекта есть файл `README.md`, который хочется использовать в документации.

Создадим в проекте директорию, например, `docs`, в ней создадим файл `Doxygen.in` командой

```bash
$ doxygen -g Doxygen.in
```

Он невероятно огромный, но в нем отражены все возможные настройки этой утилиты.
Полный список параметров [тут](http://www.doxygen.nl/manual/config.html).
Те из них, которые мы потенциально можем менять в ходе работы над проектом, заменим на теги для CMake, остальные захардкодим.
Например, параметр `INPUT_ENCODING` можно раз и навсегда оставить `UTF-8`, а вот файлы, которые будем кормить системе на вход `INPUT` лучше заменить на тег `@DOXYFILE_INPUT@`.

Базовый вариант конфига:

```doxyfile
PROJECT_NAME           = "@CMAKE_PROJECT_NAME@"
INPUT                  = @DOXYGEN_INPUT@
FILE_PATTERNS          = *.h \
                         *.cc
RECURSIVE              = YES
USE_MDFILE_AS_MAINPAGE = @DOXYGEN_USE_MDFILE_AS_MAINPAGE@
```

Сборка в html включена по умолчанию, на эту тему можно ничего не трогать.

# Вызов Doxygen из CMake

При вызове процесса сборки документации будет происходить следующее:

1. CMake возьмет шаблон конфига `./docs/Doxyfile.in`, в котором значения параметров будут заменены на теги;
2. CMake заменит теги на значения параметров, которые прописаны в `./CMakeLists.txt`;
3. новый `./cmake-build-debug/Doxyfile.out` будет отложен в директорию с результатами сборки;
4. на его основе будет вызвана команда `doxygen` и сгенерирован пакет документации.

```cmake
find_package(Doxygen)

if (DOXYGEN_FOUND)
    set(DOXYGEN_INPUT ${CMAKE_CURRENT_SOURCE_DIR}/README.md\ ${CMAKE_CURRENT_SOURCE_DIR}/Src\ ${CMAKE_CURRENT_SOURCE_DIR}/Inc)
    
    set(DOXYGEN_OUTPUT_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/docs/)
    
    set(DOXYGEN_IN ${CMAKE_CURRENT_SOURCE_DIR}/docs/Doxyfile.in)
    set(DOXYGEN_OUT ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile.out)
    
    configure_file(${DOXYGEN_IN} ${DOXYGEN_OUT} @ONLY)

    add_custom_target( docs ALL
            COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_OUT}
            WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
            COMMENT "Generating docs"
            VERBATIM )
else (DOXYGEN_FOUND)
    message("doxygen need to be installed")
endif(DOXYGEN_FOUND)
```

После сборки проекта, в директории с артефактами сборки (например, `cmake-build-debug`) должны появиться директории `docs/html`.
В ней находим `index.html` и радуемся жизни.

![пример html](https://i.ibb.co/hmLCk2W/image.png)

Если хочется собирать документацию только для Release-сборок, то конфиг выше надо обернуть в следующее условие:

```cmake
if (CMAKE_BUILD_TYPE MATCHES "^[Rr]elease")
    # build the docs
endif()
```

# References

- [FindDoxygen](https://cmake.org/cmake/help/latest/module/FindDoxygen.html)
- [Quick setup to use Doxygen with CMake](https://vicrucann.github.io/tutorials/quick-cmake-doxygen/)
- [Calling Doxygen from cmake](https://p5r.uk/blog/2014/cmake-doxygen.html)
- [Doxygen w/ CMake](https://aliceo2group.github.io/advanced/doxygen.html)
