---
title: "[latex] Установка LaTeX в Fedora Workstation 30"
---

Коротко о том, как установить сам LaTeX в Fedora Linux, пакеты для поддержки русского языка, рецепт по установке отдельных пакетов и т.д.

## Установка

Установка базового набора из LaTeX, XeLaTeX (который я использую в своих работах), а также различных пакетов для поддержки кириллицы и цитирования литературы по ГОСТу:

```bash
sudo dnf install \
            texlive-base \
            texlive-t2 \
            texlive-xetex \
            texlive-polyglossia \
            texlive-xltxtra \
            texlive-gost \
            texlive-biblatex-gost \
            texlive-cyrillic \
            texlive-hyphen-russian
```

Установка отдельного модуля может производиться двумя путями.

Можно, зная название нужного модуля, попробовать поискать нужный пакет в dnf:

```bash
dnf search siunitx
```

В ответ получается примерно такой вывод:

```bash
➜  ~ dnf search siunitx
Last metadata expiration check: 1 day, 2:44:45 ago on Fri 17 Jan 2020 08:24:17 PM MSK.
============================================= Name & Summary Matched: siunitx ==============================================
texlive-siunitx-doc.noarch : Documentation for siunitx
================================================== Name Matched: siunitx ===================================================
texlive-siunitx.noarch : A comprehensive (SI) units package
```

Тогда, собственно, просто устанавливаем соответствующий пакет:

```bash
sudo dnf install -y texlive-siunitx
```

Другой способ пригодится, если вы не знаете имя модуля, в который входит необходимый файл типа `*.sty` или `*.cls`.
В таком случае можно выполнить в пакетом менеджере по имени нужного файла:

```bash
sudo dnf install 'tex(beamer.cls)' 
sudo dnf install 'tex(hyperref.sty)' 
```

Тогда пакетный менеджер найдет необходимый пакет, содержащий нужный файл модуля и установит его.

## Шрифты

В базовом виде LaTeX довольно капризный по отношению к кодировкам исходных файлов, плюс предлагает неудобный механизм работы со шрифтами.
Так как все мы в душе немного художники и любим поиграться со шрифтами, разумный путь перехода -- XeLaTeX.
Он поддерживает все юникодные символы (137,994 штук) и умеет использовать шрифты, установленные в системе в формате `*.ttf` и `*.otf`.

Для разнообразных работ в универ, например, требуют шрифт Times New Roman, потому что так _принято_, да и вообще в ГОСТе написано.
Но конкретно с ним под Windows 10 у меня не работают переносы слов, следовательно, слова не заполняют целиком предоставленную ширину строки, получается некрасиво.

![пример работы с Times New Roman](https://i.ibb.co/kyTYBsj/image.png)

При этом на других ttf шрифтах переносы работают без проблем, какая-то локальная нестыковка.
Пришлось искать альтернативу.

Вообще, [ГОСТ 7.32-2017](http://www.tsu.ru/upload/medialibrary/8cf/gost_7.32_2017.pdf) (который для отчетов по НИР, общий) написано, что Times является _рекомендованным_ для использования.
Так что можно брать что-то похожее на таймс и отстаивать свою позицию, ссылаясь на ГОСТ.

Задача создать бесплатную альтернативу таймсу стояла перед миром попенсорса давно, и многие команды над ней работали.
В результате появилось несколько свободных альтернатив:

- [Linux Libertine](https://sourceforge.net/projects/linuxlibertine/);
- [DejaVu Serif](https://dejavu-fonts.github.io/) (с некоторыми нареканиями);
- [PT Astra Serif](https://www.paratype.ru/fonts/pt/pt-astra-serif).

Последний, PT Astra Serif -- одна из недавних находок.
Разработан по заказу разработчика операционных систем «Astra Linux» АО «НПО РусБИТех» на основе начертаний шрифта PT Serif.
При этом он _специально_ сделан максимально близким к начертаниям таймса, чтобы быть пригодным к использованию в отечественном электронном документообороте.
Ну и вообще астрой пользуются силовые структуры.
Если можно им -- нам, mere студентам, можно и подавно.

![пример абзаца на PT Astra Serif](https://i.ibb.co/x6qXy4h/image.png)

## MWE

MWE -- minimal working example -- документ в LaTeX, обладающий минимально необходимыми параметрами, чтобы быть собранным в любой системе.
За годы конспектирования лекций в латех, я пришел к следующему простейшему шаблону:

```latex
\documentclass[11pt]{article}
\usepackage[margin=0.2in]{geometry}
   \geometry{a5paper}

\usepackage[quiet]{fontspec}
\usepackage{xunicode}
\usepackage{xltxtra}
\usepackage{polyglossia}
\setdefaultlanguage{russian}
\setmainfont[Ligatures=TeX, Mapping=tex-text]{PT Astra Serif}

% math formulas optimization
\usepackage{amsmath}
\usepackage{amssymb}

% simpler df/dx and brackets
\usepackage{physics}

% cool tables
\usepackage{booktabs}

% use SI units macros
\usepackage{siunitx}

% stretch table rows a little
\renewcommand{\arraystretch}{2}

\numberwithin{equation}{section}
\numberwithin{figure}{section}

\title{Название работы}
\author{ФИО}
\date{\today}
\makeindex
\begin{document}

\pagestyle{empty}
\maketitle

\input{your-doc-name}

\end{document}
```

Не совсем минимальный, конечно.
В нем подключены пакеты, с которыми обычно приходится работать -- `siunitx` для удобного обозначения каких-нибудь значений и `physics`, существенно упрощающий ввод производных, скобок и прочей математической ерунды.

Файлы лекций у меня обычно создаются по дате, например, `2020-01-10.tex` и включаются в документ командой `\input{}`.
При этом сквозная нумерация всего и вся работает, оглавление создается, перекрестные ссылки работают.

## References

- https://docs.fedoraproject.org/en-US/neurofedora/latex/
