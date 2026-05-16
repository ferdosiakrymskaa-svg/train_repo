## Лабораторная работа№6
---
### Homework
В данной лабораторной работе нам было необходимо создать пакеты для изменений, которые помечаются тэгами.
Пакет должен содержать приложение `solver` из предыдущего задания - лабораторной работы№3. Таким образом, каждый новый релиз будет состоять из следующих компонентов:
-  архивы с файлами исходного кода (`.tar.gz`, `.zip`)
-  пакеты с бинарным файлом `solver` (`.deb`, `.rpm`, `.msi`, `.dmg`)
---
### Разбор действий, происходящих в лабораторной работе
#### Файлы лабораторной работы№3
Файлы данной лабораторной работы не изменились, кроме `solver/CMakeLists.txt`:
Добавил в конец строку `install(TARGETS solver RUNTIME DESTINATION bin)`:
Команда `install(TARGETS solver RUNTIME DESTINATION bin)` сообщает CMake, что при установке проекта исполняемый файл `solver` нужно скопировать в каталог `bin` внутри установочного префикса.
Это означает:
- Когда вызывается `cmake --install` или **CPack** собирает пакет (DEB, RPM и т.д.), бинарник `solver` будет помещён в папку `bin`, которая внутри пакета соответствует `/usr/bin/` (для Linux) или другому системному пути.
- Без этой команды CPack не знал бы, что именно упаковывать, и пакет мог бы оказаться пустым.

Ключевое слово `RUNTIME` указывает, что цель `solver` — это исполняемый файл (а не библиотека). `DESTINATION bin` задаёт подкаталог относительно `CMAKE_INSTALL_PREFIX`. Например, при значении префикса `/usr/local` файл установится в `/usr/local/bin/solver`.
Таким образом, строка необходима для корректного формирования установочных пакетов с приложением `solver` внутри.

---
#### `CpackConfig.cmake`
```sh
include(InstallRequiredSystemLibraries)

set(CPACK_PACKAGE_CONTACT ferdosiakrymskaa@gmail.com)
set(CPACK_PACKAGE_VERSION_MAJOR ${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK ${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})
set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing and equation solver")

set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)
set(CPACK_WIX_LICENSE_RTF ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.rtf)
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)

# RPM
set(CPACK_RPM_PACKAGE_NAME "solver")
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "Applications/Engineering")
set(CPACK_RPM_CHANGELOG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)

# DEB
set(CPACK_DEBIAN_PACKAGE_NAME "solver")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")
set(CPACK_DEBIAN_PACKAGE_RELEASE 1)

include(CPack)
```
- `include(InstallRequiredSystemLibraries)` Подключает модуль, который определяет, какие системные библиотеки требуются для запуска собранного приложения на целевой системе.<br>
Далее идет присваивание значений переменным:<br>
- `set(CPACK_PACKAGE_CONTACT ferdosiakrymskaa@gmail.com)` присваивает имя переменной `CPACK_PACKAGE_CONTACT` - мою почту для связи.
- `set(CPACK_PACKAGE_VERSION_MAJOR ${PRINT_VERSION_MAJOR})` задает значение основной версии
- `set(CPACK_PACKAGE_VERSION_MINOR ${PRINT_VERSION_MINOR})` задает значение вспомогательной версии версии
- `set(CPACK_PACKAGE_VERSION_PATCH ${PRINT_VERSION_PATCH})` задает  номер исправление
- `set(CPACK_PACKAGE_VERSION_TWEAK ${PRINT_VERSION_TWEAK})` задает номер микроизменения
- `set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})` задает сложную версию пакетов, которая соберется в следующем файле из четырех предыдущих переменных
- `set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)` задает описание проекта
- `set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for printing and equation solver")` задает описание проекта
- `set(CPACK_RESOURCE_FILE_LICENSE ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE)` лицензия для остальных
- `set(CPACK_WIX_LICENSE_RTF ${CMAKE_CURRENT_SOURCE_DIR}/LICENSE.rtf)` лицензия для Windows(потребовалась, потому что когда ее не было, то в виртуальной машине, выделяемой GitHub происходила ошибка в ее наличии - ее отсутствие)
- `set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/README.md)` README.md файл
- `set(CPACK_RPM_PACKAGE_NAME "solver")` имя пакетов
- `set(CPACK_RPM_PACKAGE_LICENSE "MIT")` имя лицензии
- `set(CPACK_RPM_PACKAGE_GROUP "Applications/Engineering")` группа пакетов
- `set(CPACK_RPM_CHANGELOG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)` задает ChangeLog.md <br>
Далее обычное присваивание значениям переменных:
- `set(CPACK_RPM_PACKAGE_RELEASE 1)`
- `set(CPACK_DEBIAN_PACKAGE_NAME "solver")`
- `set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")`
- `set(CPACK_DEBIAN_PACKAGE_RELEASE 1)`
- `include(CPack)`:<br>
Команда `include(CPack)` в файле `CPackConfig.cmake` (или напрямую в `CMakeLists.txt`) делает следующее:
- Подключает встроенный модуль CPack из состава CMake.
- Активирует все заданные ранее переменные `CPACK_*` (имя пакета, версия, описание, генераторы, файлы лицензии и т.д.) – именно они будут использованы при создании пакетов.
- Добавляет цель `package`, которую можно вызвать через `cmake --build _build --target package` или через прямой вызов `cpack`.
- Без этой строки `cpack` не сможет собрать пакет, даже если все переменные в `CPackConfig.cmake` заданы верно.

В нашем случае `include(CPack)` завершает конфигурационный файл `CPackConfig.cmake` и финализирует настройки для генерации `.deb`, `.rpm`, `.dmg`, `.msi`. В корневом `CMakeLists.txt` мы подключаем этот файл строкой `include(CPackConfig.cmake)`, а уже внутри него происходит `include(CPack)`.

---

#### Корневой `CMakeLists.txt`

```sh
cmake_minimum_required(VERSION 3.10)
project(FormatterProject)

set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)
set(PRINT_VERSION ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})
set(PRINT_VERSION_STRING "v${PRINT_VERSION}")

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(formatter_lib)
add_subdirectory(formatter_ex)
add_subdirectory(solver_lib)
add_subdirectory(hello_world)
add_subdirectory(solver)

include(CPackConfig.cmake)
```
Присваивание переменным, описанным в `CPackConfig.cmake`, значений:<br>
```sh
set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)
```
- присваивает сложное значение версии строки `set(PRINT_VERSION ${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK})` и `set(PRINT_VERSION_STRING "v${PRINT_VERSION}")`.
- `include(CPackConfig.cmake)` подключает файл `CPackConfig.cmake`.<br>
Остальное все по стандарту
---
#### `ChangeLog.md`

`ChangeLog.md` нужен **специально для RPM-пакетов**. В нашем `CPackConfig.cmake` есть строка:
```cmake
set(CPACK_RPM_CHANGELOG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
```
Она указывает, что содержимое этого файла будет вставлено в секцию `%changelog` итогового `.rpm`-пакета. В RPM-формате changelog — обязательный элемент, в котором описывается история изменений по версиям (дата, автор, версия, краткое описание правок). Когда пользователь устанавливает пакет и смотрит его историю командой `rpm -q --changelog solver`, он видит информацию из этого файла.
Для `.deb`, `.dmg`, `.msi` этот файл не используется (там свои механизмы), поэтому без него сборка `cpack -G RPM` падала с ошибкой. После создания `ChangeLog.md` ошибка ушла.
<details>
<summary>Вывод ошибки </summary>
  
```sh
CMake Error at /usr/share/cmake-3.28/Modules/Internal/CPack/CPackRPM.cmake:1230 (message):
  CPackRPM:Warning: CPACK_RPM_CHANGELOG_FILE
  </home/ilyasov_ilya/ferdosiakrymskaa-svg/workspace/projects/lab06/lab06_training/Homework_lab03/ChangeLog.md>
  does not exist - ignoring
Call Stack (most recent call first):
  /usr/share/cmake-3.28/Modules/Internal/CPack/CPackRPM.cmake:1982 (cpack_rpm_generate_package)

CPackRPM: Will use GENERATED spec file: /home/ilyasov_ilya/ferdosiakrymskaa-svg/workspace/projects/lab06/lab06_training/Homework_lab03/_build/_CPack_Packages/Linux/RPM/SPECS/solver.spec
CPack Error: Error while execution CPackRPM.cmake
CPack Error: Problem compressing the directory
CPack Error: Error when generating package: FormatterProject
```

</details>

---
#### `.github/workflows/release.yml`

```sh
name: Release packages # Имя

permissions: # Выдает разрешение на запись в раздел Contents, так как изначально есть только права на его чтение
  contents: write

on:
  push:
    tags: ['v*'] # Триггерится, когда происходит пуш коммита с тегом 

jobs:
  build:
    name: ${{ matrix.os }} # матрица
    runs-on: ${{ matrix.os }} # Запуск будет производится на операционных системах из матрицы
    strategy:
      matrix:
        include: # задаются условия для соответсвующих операционных систем
          - os: ubuntu-latest
            generator: DEB;RPM # Это тип пакета который сгенерирует генератор
            pattern: |         # Шаблон для поиска собранных файлов, которые необходимо загрузить как артефакты
              _build/*.deb
              _build/*.rpm
            install_cmd: sudo apt-get update && sudo apt-get install -y rpm
          - os: macos-latest
            generator: DragNDrop
            pattern: _build/*.dmg
            install_cmd: ''
          - os: windows-latest
            generator: WIX
            pattern: _build/*.msi
            install_cmd: choco install wixtoolset -y

    steps: # Шаги выполнения действий
      - uses: actions/checkout@v3

      - name: Install dependencies # Шаг для проверки, не пустой ли matrix.install_cmd
        if: matrix.install_cmd != ''
        run: ${{ matrix.install_cmd }}

      - name: Configure CMake # Компиляция с генератором
        run: cmake -B _build -DCPACK_GENERATOR="${{ matrix.generator }}"

      - name: Build # Сборка
        run: cmake --build _build --config Release

      - name: Package # Шаг создаёт установочные пакеты для текущей операционной системы, указанной в матрице. -C Release задает конфигурациюс сборки.
        working-directory: _build
        run: cpack -G "${{ matrix.generator }}" -C Release

      - name: Upload artifacts (temporary)
        uses: actions/upload-artifact@v4
        with:
          name: packages-${{ matrix.os }}
          path: ${{ matrix.pattern }}

  release: # Релиз
    name: Publish Release
    runs-on: ubuntu-latest
    needs: build # Ждет завершение сборки
    steps:
      - name: Download all artifacts
        uses: actions/download-artifact@v4
        with:
          path: artifacts

      - name: Create Release and upload assets
        uses: softprops/action-gh-release@v1
        with:
          files: artifacts/**/*
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # В результате после успешного выполнения этого блока на странице релизов появится запись с прикреплёнными пакетами для Linux, macOS и Windows.
```
