---
title: "[qt] Установка Qt и QtCreator в Windows 10"
categories: qt, qtcreator, windows 
---

В этой заметке описывается последовательность действий, необходимых для установки и запуска пакета Qt на Windows 10 с компилятором MSVC2015.

Шаг 1 -- скачиваем установщики:

- [Qt](https://www.qt.io/download-qt-installer) -- с последних версий установщик прячут за тонной просьб купить пакет, если занимаешься коммерческой разработкой. В ближайшем будущем грозятся разрешать скачивание только после регистрации, посмотрим.
- [Visual Studio Community](https://visualstudio.microsoft.com/)

Шаг 2 -- устанавливаем Qt:

> ![Maintenance tool: выбор набор библиотек под компилятор MSVC 2015](https://i.ibb.co/mX2CgfT/Maintenance-Tool-4c9-LC2ues-C.png)
>
> _Maintenance tool: выбор набор библиотек под компилятор MSVC 2015_

Можно набрать дополнительных штук:

> ![Maintenance tool: выбор редактора, отладчика и дополнительных штук](https://i.ibb.co/YThMXXw/Maintenance-Tool-Fg-Fi-Fi-Tmrt.png)
>
> _Maintenance tool: выбор редактора, отладчика и дополнительных штук_

Шаг 3 -- устанавливаем MSVC 2015, Visual Studio и опционально дополнительные пакеты:

> ![Выбор дополнительных компонентов в Visual Studio Installer](https://i.ibb.co/M69BtFb/vs-installershell-Zr-ILcs-N2-E5.png)
>
> _Выбор дополнительных компонентов в Visual Studio Installer_

Шаг 3.5 -- можно пойти в обход и на главной странице установщика импортировать конфигурацию, чтобы не искать нужные галочки:

> ![Импорт конфигурации установки Visual Studio](https://i.ibb.co/KykrLVY/image.png)
> 
> _Импорт конфигурации установки Visual Studio_

Содержимое файла `.vsconfig`:

```
{
  "version": "1.0",
  "components": [
    "Microsoft.VisualStudio.Component.CoreEditor",
    "Microsoft.VisualStudio.Workload.CoreEditor",
    "Microsoft.VisualStudio.Component.NuGet",
    "Microsoft.VisualStudio.Component.Roslyn.Compiler",
    "Microsoft.VisualStudio.ComponentGroup.WebToolsExtensions",
    "Microsoft.Component.MSBuild",
    "Microsoft.VisualStudio.Component.TextTemplating",
    "Microsoft.VisualStudio.Component.IntelliCode",
    "Microsoft.Component.PythonTools",
    "Microsoft.VisualStudio.Component.VC.CoreIde",
    "Microsoft.VisualStudio.Component.VC.Tools.x86.x64",
    "Microsoft.VisualStudio.Component.Graphics.Tools",
    "Microsoft.VisualStudio.Component.VC.DiagnosticTools",
    "Microsoft.VisualStudio.Component.Windows10SDK.18362",
    "Component.CPython2.x86",
    "Microsoft.VisualStudio.Component.Debugger.JustInTime",
    "Microsoft.VisualStudio.Component.VC.Redist.14.Latest",
    "Microsoft.VisualStudio.ComponentGroup.NativeDesktop.Core",
    "Microsoft.VisualStudio.ComponentGroup.WebToolsExtensions.CMake",
    "Microsoft.VisualStudio.Component.VC.CMake.Project",
    "Microsoft.VisualStudio.Component.VC.ATL",
    "Microsoft.VisualStudio.Component.VC.TestAdapterForBoostTest",
    "Microsoft.VisualStudio.Component.VC.TestAdapterForGoogleTest",
    "Microsoft.VisualStudio.Component.VC.Llvm.Clang",
    "Microsoft.Component.VC.Runtime.UCRTSDK",
    "Microsoft.VisualStudio.Component.VC.140",
    "Microsoft.VisualStudio.Workload.NativeDesktop",
    "Component.MDD.Linux",
    "Component.Linux.CMake",
    "Microsoft.VisualStudio.Workload.NativeCrossPlat"
  ]
}
```

Шаг 4 -- прописываем в `PATH` путь к установленному компилятору:

> ![Установка пути к файлам компилятора в переменную PATH](https://i.ibb.co/p4x7Qwc/System-Properties-Advanced-YSTgl7-Gjy-Y.png)
> 
> _Установка пути к файлам компилятора в переменную PATH_

Нужный путь можно уточнить, зайдя в папку `C:\Program Files (x86)\Windows Kits` и выполнить поиск файла `rc.exe`, и папку с его расположением добавляем в `PATH`:

> ![Поиск rc.exe](https://i.ibb.co/ZmLsXfr/image.png)
> 
> _Поиск `rc.exe`_

Шаг 5 -- настраиваем "набор" для сборки в QtCreator -> Настройки:

> ![QtCreator автоматически настроил комплект сборки](https://i.ibb.co/JCXV1YN/qtcreator-XW9-DHWhpio.png)
> 
> _QtCreator автоматически настроил комплект сборки_

Шаг 6 -- готово, можно создавать новый проект!
