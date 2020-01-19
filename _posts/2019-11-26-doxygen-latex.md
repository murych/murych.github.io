---
title: "[cmake] Генерируем Doxygen в PDF"
categories: cmake, latex
---

# Постановка задачи

В [предыдущем посте](https://murych.github.io/doxygen-cmake/) рассматривался процесс генерирования документации из Doxygen в набор html файлов.
Однако результат этой сборки представляет собой директорию из 500+ мелких файлов, не самый переносимый вариант.

Вот бы было возможно собрать весь этот ужас в один красивый pdf файл, да еще чтобы процесс можно было упихать в CI... :thinking_face:

К счастью, в Doxygen встроен механизм генерирования документации через LaTeX, и мы с радостью этим воспользуемся.

# Подготовка окружения

Как обычно для работы с латехом, надо сначала установить хренову кучу пакетов.
Ничего не поделать, образ для докера еще не готов, а вот pdf-ку собрать надо уже сейчас.

Сделаю смелое предположение, что читатель пользуется Fedora и через [установку самого LaTeX'a](https://murych.github.io/latex-fedora/) с базовым набором пакетов уже продрался.
В таком случае, нужно установить следующие пакеты:

```bash
$ sudo dnf install -y \
    "tex(tabu.sty)" \
    "tex(multirow.sty)" \
    "tex(hanging.sty)"  \
    "tex(adjustbox.sty)"    \
    "tex(stackengine.sty)"  \
    "tex(listofitems.sty)"  \
    "tex(ulem.sty)" \
    "tex(wasysym.sty)"  \
    "tex(sectsty.sty)"  \
    "tex(tocloft.sty)"  \
    "tex(newunicodechar.sty)"   \
    "tex(etoc.sty)"
```

# Подготовка конфига Doxygen

Открываем файл `docs/Doxyfile.in` и добавляем в него следующие строки:

```doxyfile
GENERATE_LATEX         = @DOXYGEN_GENERATE_LATEX@
LATEX_OUTPUT           = @DOXYGEN_LATEX_OUTPUT@
LATEX_CMD_NAME         = xelatex
COMPACT_LATEX          = YES
PAPER_TYPE             = a4
PDF_HYPERLINKS         = YES
USE_PDFLATEX           = YES
```

Далее открываем `CMakeLists.txt` и где-нибудь перед строчкой `configure_file(...)` добавим:

```cmake
set(DOXYGEN_GENERATE_LATEX YES)
```

Теперь рядом с директорией `html` должна появиться директория `latex`.
Заходим в нее, отркываем `Makefile`.
Меняем значение `LATEX_CMD` с `latex` на `xelatex`.
После этого открываем директорию в терминале и выполняем `make` (для того, чтобы установились все перекрестные ссылки, можно выполнить 2-3 раза).

Все должно пройти нормально, получаем файл `refman.pdf` с нашей документацией, радуемся.

![пример pdf, у меня инверсная](https://i.ibb.co/tmTZrNb/image.png)

# TODO

По-хорошему, надо подготовить докер-контейнер специально для сборки pdf-ки.
Когда-нибудь `¯\_(ツ)_/¯`
