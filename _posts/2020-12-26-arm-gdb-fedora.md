---
title: "Установка GNU Debugger 10.1 для `arm-none-eabi` в Fedora 33"
categories: stm32, fedora, gdb, arm, qtcreator
---

В репозиториях моего любимого дистрибутива Fedora Linux по умолчанию отсутствует пакет отладчика для платформы ARM -- `arm-none-eabi-gdb`, хотя другие инструменты, вроде компилятора, присутствуют.
Существуют пользовательские репозитории, доступные через dnf copr, однако, они уже устаревшие и не обновлялись последние 2-3 года.

Как обычно бывает в таких ситуациях, придется устанавливать из исходников.
Данная заметка является шпаргалкой на тему -- как быстро установить, если в очередной раз обновил систему, и все забыл.

## Подготовка

Устаналиваем необходимые для компиляции библиотеки, а также их варианты с суффиксом `-devel`:

```bash
sudo dnf groupinstall "Development Tools"
sudo dnf install texinfo python3-devel ncurses ncurses-devel expat expat-devel \
	flex bison gmp-devel mpfr-devel libmpc-devel autoconf zlib-devel
```

Обратите внимание, что я планирую писать и отлаживать код через среду разработки QtCreator, которой требуется вариант отладчика с поддержкой питона.
Если Вам оно не надо, установку пакета `python3-devel` можно опустить.

## Установка

Скачиваем и распаковываем архив отладчика с официального сайта https://ftp.gnu.org/gnu/gdb/ :

```bash
$ cd ~/downloads
$ wget https://ftp.gnu.org/gnu/gdb/gdb-10.1.tar.gz
$ tar -xjvf gdb-10.1.tar.gz
```

Заходим в получившийся каталог и устанавливаем:

```bash
$ cd gdb-10.1
$ ./configure --prefix=/opt/arm/gdb-10.1 --with-expat --with-python --target=arm-none-eabi --enable-multilib --enable-interwork --disable-nls --disable-ibssp
$ make all
$ sudo make install
```

Готово!
Теперь в каталоге `/opt/arm/gdb-10.1/` будет находиться исполняемый файл отладчика.
Не забудьте добавить этот каталог в `$PATH`, чтобы не писать каждый раз путь целиком.
Проверить установку можно с помощью вызова ключа `--version`:

```bash
$ /opt/arm/gdb-10.1/bin/arm-none-eabi-gdb --version
GNU gdb (GDB) 10.1
Copyright (C) 2020 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
```

## Источники

- [Сборка инструментария ARM Toolchain (arm-none-eabi) в Fedora Linux](https://cxemotexnika.org/2013/12/sborka-instrumentariya-arm-toolchain-arm-none-eabi-v-fedora-linux/)
- [Bug 1859627 - Review Request: arm-none-eabi-gdb - GDB for (remote) debugging ARM bare-metal targets ](https://bugzilla.redhat.com/show_bug.cgi?id=1859627)
- [arm-none-eabi-gdb with python support on linux (Fedora 17)](https://false.ekta.is/2013/01/arm-none-eabi-gdb-with-python-support-on-linux-fedora-17/)
