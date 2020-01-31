---
title: "Конвертация XML в PList"
---

1. Берем исходный XML файл (пример из [википедии](https://ru.wikipedia.org/wiki/XML#%D0%9A%D0%BE%D1%80%D0%BD%D0%B5%D0%B2%D0%BE%D0%B9_%D1%8D%D0%BB%D0%B5%D0%BC%D0%B5%D0%BD%D1%82)):

```xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE recipe>
<recipe name="хлеб" preptime="5min" cooktime="180min">
   <title>
      Простой хлеб
   </title>
   <composition>
      <ingredient amount="3" unit="стакан">Мука</ingredient>
      <ingredient amount="0.25" unit="грамм">Дрожжи</ingredient>
      <ingredient amount="1.5" unit="стакан">Тёплая вода</ingredient>
   </composition>
   <instructions>
     <step>
        Смешать все ингредиенты и тщательно замесить. 
     </step>
     <step>
        Закрыть тканью и оставить на один час в тёплом помещении. 
     </step>
     <!-- 
        <step>
           Почитать вчерашнюю газету. 
        </step>
         - это сомнительный шаг...
      -->
     <step>
        Замесить ещё раз, положить на противень и поставить в духовку.
     </step>
   </instructions>
</recipe>
```

2. Конвертируем XML в JSON, например, на сайте [Code Beautify](https://codebeautify.org/xmltojson):

```json
{
    "recipe": {
        "title": "Простой хлеб",
        "composition": {
            "ingredient": [
                {
                    "_amount": "3",
                    "_unit": "стакан",
                    "__text": "Мука"
                },
                {
                    "_amount": "0.25",
                    "_unit": "грамм",
                    "__text": "Дрожжи"
                },
                {
                    "_amount": "1.5",
                    "_unit": "стакан",
                    "__text": "Тёплая вода"
                }
            ]
        },
        "instructions": {
            "step": [
                "Смешать все ингредиенты и тщательно замесить.",
                "Закрыть тканью и оставить на один час в тёплом помещении.",
                "Замесить ещё раз, положить на противень и поставить в духовку."
            ]
        },
        "_name": "хлеб",
        "_preptime": "5min",
        "_cooktime": "180min"
    }
}
```

3. Берем мак (почем мне знать, откуда)
4. Сохраняем JSON файл где-нибудь в загрузках
5. Открываем терминал, заходим в загрузки
6. Выполняем:

```bash
$ plutil -convert xml1 test.json -o test.plist
```

7. ?????

```plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>recipe</key>
    <dict>
        <key>title</key>
        <value>Простой хлеб</value>
        <dict>
            <key>composition</key>
            <dict>
                <key>ingredient</key>
                <array>
                    <dict>
                        <key>_amount</key>
                        <integer>3</integer>
                        <key>_unit</key>
                        <string>стакан</string>
                        <key>__text</key>
                        <string>Мука</string>
                    </dict>
                    <dict>
                        <key>_amount</key>
                        <real>0.25</real>
                        <key>_unit</key>
                        <string>грамм</string>
                        <key>__text</key>
                        <string>Дрожжи</string>
                    </dict>
                    ...
                </array>
            </dict>
            ...
        </dict>
    </dict>
</dict>
</plist>
```

8. PROFIT!
