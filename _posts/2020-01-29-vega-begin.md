---
title: "[вега] Восстановление проигрывателя Вега ЭП-110"
---

# Лирическое вступление

Штош, первая сессия в магистратуре успешно закрыта, значит, можно на пару недель переключиться на свои давние хотелки.

> ![](https://lh4.googleusercontent.com/proxy/ArGW_w91GaMOjmEeAzeHw6jiALuuj5RUW7YThzZEu-qiVkouBk6W4RJTG_FC177rmkbYkzj8hpfsNWl5l9UzdIaITMMUk0c)
> 
> Объект хотелок (фото не мое)

Как-то летом в благородном порыве разобраться, почему же перестала работать моя старенькая Вега ЭП-110, очень увлекся процессом выкручивания винтов.
_`Please be patient, i have autism.jpg`_

В общем, разобрал до основания.

А то, что причина неисправности в порванном пассике дошло как-то не сразу.
Так он и остался с тех пор собирать пыль в дальнем углу до лучших времен.

Но лучшие времена сами не придут, их надо устраивать самому.
Да и пластинки с детскими сказками и Высоцким сами себя не послушают.
Поэтому в этом посте я декларирую план работ по почти полному восстановлению проигрывателя.

# Мелкие цели

Ради чего, собственно, такой сыр-бор?
Почему бы просто не купить новую вертушку или, на худой конец, не купить только новый ремень и успокоиться?
Основных причин всего две: жадность и гордость, задетая следующей [цитатой](https://datagor.ru/amplifiers/solid-state/1046-dorabotka-usilitelya-vega-50u-122s.html):

> Инженер, который не может починить неисправную Вегу, таковым в общем-то и не является

Помимо этого хочется:

- попрактиковаться в чтении принципиальных схем;
- оцифровать старую документацию на проигрыватель в современный ECAD;
- подружить KiCad с каким-нибудь реальным производством ПП, можно отечественным (если окажется дешевле китайских JlPCB и т.п.);
- самостоятельно заказать платы, а не возиться с ЛУТом;
- очень хочется ПП с черной защитной маской, twitter на эти темы умеет соблазнять;
- послушать наконец The Wall на виниле.

# Блок ЭПУ

В этом блоке, как минимум, уже порвался ремень, поэтому он точно под замену.

Также, руководствуясь опытом всемирной (хотя чаще отечественной) паутины, отмечены следующие пункты:

- замена электролитических конденсаторов на плате ЭПУ;
- замена смазки в узле вала диска;
- возможно добавление демпфирующей жидкости в микролифт;
- замена стробоскопирующей лампы на светодиод;
- замена фоторезистора энкодера на новый;
- замена лампы энкодера на светодиод;
- замена однополупериодного выпрямителя на диодный мост.

# Блок питания и управления

Тут хочется развернуться в полную силу, так как заводские платы одним своим видом наводят тоску.
Поэтому:

- принципиальные схемы блока питания, блока управления и блока предусилителя из руководства перерисовываются в KiCad;
- транзисторам и конденсаторам подбираются аналоги на замену;
- новые схемы моделируются в симуляторе, в proteus / LTSpice / что-там-недавно-стало-free;
- проектируются ПП с сохранением существующей геометрии;
- (возможно) схему предусилителя взять другую из описанных в сети.

Тут, кстати, уже есть некоторые наработки:

> ![Принципиальная схема блока управления](https://i.ibb.co/BBqBPvs/image.png)
> 
> Принципиальная схема блока управления

> ![Принципиальная схема блока питания](https://i.ibb.co/r6g34yQ/image.png)
> 
> Принципиальная схема блока питания

> ![Модель платы блока питания](https://i.ibb.co/XbtnG6W/image.png)
> 
> Модель платы блока питания

# Дополнительные модификации

Копаясь в ящиках нашел несобранный набор радиоконструктора СТАРТ 7249 -- блок индикации уровня сигнала на базе катодо-люминесцентного индикатора квазипикового уровня сигнала звуковой частоты со счетчиком.

> ![Фотка из интернета](http://s017.radikal.ru/i425/1302/28/fd4c2d231d8a.jpg)
> 
> Фотка радиоконструктора СТАРТ 7249 (не моя)

В общем, захотелось схему этого счастья тоже оцифровать, обновить и встроить в проигрыватель вместо усилителя для наушников, который мне там не нужен.
Будем пробовать.

# Внешний вид

Вот тут самое муторное, наверное, предстоит.
Как минимум, надо весь корпус очистить от грязи и пыли, часть которой старше меня будет.

Следующим шагом надо заменить все винты на человеческие с крестовым либо шестиугольным шлицем под метрическую резьбу.
Ибо унификация.
Те винты, которые ввинчиваются в пластик, удалить, вместо них вплавить втулки с резьбой.

Программа-максимум здесь -- перекрасить корпус в черный матовый цвет.
С деревянным коробом проблем возникнуть не должно, получится ли это сделать с блоком ЭПУ -- вопрос, но время покажет.

-------------------

Ну, в общем, план написан, можно смело ~~забить~~ приступать к осуществлению.

# References

- [Вертушка Вега ЭП-110. Учим старую собаку новым трюкам](https://datagor.ru/tuning/2624-hi-fi-vertushka-vega-ep-110-full-upgrade.html#hmenu-2)
- [Профилактика электропроигрывателя "Вега ЭП 110"](https://pikabu.ru/story/profilaktika_yelektroproigryivatelya_vega_yep_110_5296428)
- [Вариант 2010 простого усилителя-корректора для повторения - ММ и МС](http://forum.vegalab.ru/showthread.php?t=39651)
- [Простой транзисторный усилитель-корректор для магнитной головки звукоснимателя](http://www.vegalab.ru/content/view/77/55/1/2/)
