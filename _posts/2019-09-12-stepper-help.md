---
title: "[blog] Проблема с созданием линейной рампы ускорения"
mathjax: true
---

Не так давно была поставлена задача разработать лабораторную центрифугу.
Заказчик где-то нашел центрифугу, построенную на шаговом двигателе и захотел такую же.
Обосновывалось это желание тем, что на шаговом двигателе можно легко и просто реализовать точное перемешивание образцов вокруг оси вала.

Поиск по патентам гугла интересных результатов не дал, а может, проводился неправильно.
Поэтому решили сделать макет и посмотреть, что получится `¯\_(ツ)_/¯`

## Постановка задачи

Создать макет лабораторной центрифуги.
Центрифуга должна работать в двух режимах:

- центрифугирование -- вращение на постоянной скорости + линейный разгон/торможение до максимальной скорости;
- перемешивание -- вращение в противоположные стороны с регулируемой частотой и амплитудой.

Максимальная частота вращения в режиме центрифугирования -- 40 кГц (примерно 1500 об/мин).

Шаговый двигатель должен работать в режиме микрошага 1/8 или 1/16.

## Выбор элементной базы

Для макета требуется, строго говоря, три элемента -- сам двигатель (ШД), драйвер к нему и микроконтроллер.

Делать новое устройство пока рано, а нормальных отладочных плат под рукой не было, поэтому взяли Arduino Uno на Atmega328P и зареклись писать на C-диалекте, чтобы позже перенести на микроконтроллер STM.

В качестве драйвера был взят драйвер MKS на базе микросхемы LV8727, опять же, потому что просто лежал под рукой.
Драйвер имеет несколько входов, в частности, `DIR` для задания направления вращения и `STEP` для выполнения шага двигателем.
`STEP` срабатывает по на логическую единицу, следовательно, чтобы сделать максимальную скорость шагания 40 кГц, ножкой МК придется дергать в 2 раза чаще -- 80 кГц.
Задача даже для ардуины посильная, но с портами ввода-вывода будем работать напрямую, чтобы гарантированно побыстрее.

## Математическое обоснование

Для реализации линейного разгона/торможения решил опираться на статьи David Austin -- [Generate stepper-motor speed profiles in real time](https://www.embedded.com/design/mcus-processors-and-socs/4006438/Generate-stepper-motor-speed-profiles-in-real-time) и аппнот от Atmel'a -- [AVR446](http://ww1.microchip.com/downloads/en/appnotes/doc8017.pdf).

Выглядит все замечательно просто.
Используется встроенный в МК счетчик, на него вешается прерывание.
По этому прерыванию дергаем ногу МК, к которому подключен вход STEP с драйвера.

Число, до которого должен досчитать счетчик на нужной частоте $$f_i$$ рассчитывается, согласно даташиту на МК:

$$
    n = \frac{F_{MCU}}{prescaler \cdot f_i} - 1.
$$

Нужная частота 40 кГц, частота работы микроконтроллера 16 МГц, предделитель -- 8.
Получается, что для вращения на 40 кГц, должно срабатывать прерывание при досчитывании до числа 49.

_НО_.
Надо же до этой частоты дойти, желательно быстро.
Поэтому считаем линейную рампу ускорения:

![профиль скорости при линейном разгоне и торможении (без "круиза" на макисмальной скорости)](http://avrdoc.narod.ru/olderfiles/1/fig446-3-2.jpg)

Так как при аппроксимации прямой в лоб присутствует серьезная погрешность, для ее устранения предлагается рассчитывать _первую задержку_ отдельно по формуле

$$
    n_0 = 0.676 \cdot f \cdot \sqrt{\frac{2 MS}{accel}},
$$

где `accel` - ускорение, `MS` - дробная часть шага, на которую нужно повернуть вал.

Следующие задержки рассчитываются инкрементально (вроде это слово), т.е. следующее значение вычисляется из предыдущего:

$$
    n_i = n_{i-1} \frac{2 n_{i-1}}{4 n + 1}.
$$

## Описание алгоритма работы

Объявим новый тип в виде структуры, содержащей всю информацию о состоянии ШД

{% highlight c %}

typedef struct {
    //! pinout configuration
    uint8_t dir_pin;
    uint8_t step_pin;
    uint8_t sleep_pin;

    //! current microstep level (1,2,4,8,...)
    uint8_t  config_ms;
    uint16_t config_rpm;
    uint16_t config_spr;
    
    //! speed ramp settings
    volatile speed_profile profile;
    //! current part of ramp
    volatile stepper_status status;

    //! current position
    long steps_count;
    //! to complete the current move (absolute value)
    long steps_remaining;
    //! steps to reach max (cruising) rpm
    long steps_to_cruise;
    //! steps needed to come to a full stop
    long steps_to_break;
    //! calculation remainder to be fed into successive steps to increase accuracy
    //! ref: Atmel DOC8017
    long rest;
    //! step pulse duration [ms]
    long step_delay;
    //! step pulse duration for constant speed section (max rpm)
    long step_delay_cruise;
} stepper_state;

{% endhighlight %}

После запуска программы создается объект `stepper` этого типа.
При вызове функции центрифугирования, в поле `speed_profile` записывается информация о том, что мы хотим линейного разгона.
Если же выбран режим перемешивания, в это поле записывается информация о том, что мы хотим ехать на постоянной скорости, до которой можно не разгоняться.

Шагать на заданное количество шагов будем с помощью функции `move`, которая вызовет расчет первой задержки таймера, а затем -- расчет остальных задержек (до тех пор, пока не закончатся шаги).
Естественно, при постоянной скорости, значение задержки сразу ставится максимальным и дальнейшего расчета не идет.

{% highlight c %}

void move(long steps) {
    startMoving(steps);
    while (calcStepPulse());
}

{% endhighlight %}

Считаем первую задержку:

{% highlight c %}
speed = stepper.config_rpm * stepper.config_spr / 60;

stepper.steps_to_cruise = stepper.config_ms * long(speed * speed / (2 * stepper.profile.accel));
stepper.steps_to_break = stepper.steps_to_cruise * stepper.profile.accel / stepper.profile.decel;

if (stepper.steps_remaining < stepper.steps_to_cruise + stepper.steps_to_break) {
    //! cannot reach max speed, will need to break earlier
    stepper.steps_to_cruise = stepper.steps_remaining * stepper.profile.decel /
        (stepper.profile.accel + stepper.profile.decel);
    stepper.steps_to_break = stepper.steps_remaining - stepper.steps_to_cruise;
}

stepper.step_delay = long(1e+6 * 0.676 * sqrt(2.0 / stepper.profile.accel / stepper.config_ms));
stepper.step_delay_cruise = long(1e+6 / double(speed) / stepper.config_ms);
{% endhighlight %}

Расчет остальных задержек:

{% highlight c %}
if (stepper.steps_count < stepper.steps_to_cruise) {
    stepper.step_delay = stepper.step_delay - (2 * stepper.step_delay + stepper.rest) /
        (4 * stepper.steps_count + 1);
    stepper.rest = (stepper.steps_count < stepper.steps_to_cruise) ?
        (2 * stepper.step_delay + stepper.rest) % (4 * stepper.steps_count + 1) : 0;
} else {
    stepper.step_delay = stepper.step_delay_cruise;
}
break;
{% endhighlight %}

Здесь же выполняется реакция на различные участки рампы -- разгон, вращение на максимальной скорости, торможение и остановка:

{% highlight c %}
switch (stepperGetCurrentStatus()) {

case ACCELERATING:  // разгон
    // смотри выше
    break;

case DECELERATING:  // торможение, заменяем знак в формулах для разгона
    stepper.step_delay = stepper.step_delay - (2 * stepper.step_delay + stepper.rest) /
        (-4 * stepper.steps_remaining + 1);
    stepper.rest = (2 * stepper.step_delay + stepper.rest) % (-4 * stepper.steps_remaining + 1);
    break;

case STOPPED:   // остановка
    TCCR1B &= ~(1 << CS11);
    PORTB &= ~(1 << PORTB1);
    toggle = 1;
    break;

case CRUISING:  // вращение на максимальной скорости
    break;
}
{% endhighlight %}

Расчет количества тактов МК на вычисление задержки не проводился (yet).

## Моделирование

Хотелось как-то проверить адекватность реализации алгоритма.
Решил прогнать этот же код на локальной машине (убрав все ардуино-рилейтед команды), пересчитать число задержки таймера обратно в частоту прерывания и записать результаты вычислений в текстовый файл, построить график.
Результат расстроил:

![результат выполнения кода на локальной машине](https://i.ibb.co/7Xvmjwr/calculation.png)

Линейностью и не пахнет, разгон происходит ступеньками.
Также, за 500 тыщ "шагов" алгоритм не достигает заданной задержки и начинает тормозить раньше.
Полное разочарование.

Совсем недавно универ открыл для меня Proteus, в котором есть симуляция работы ардуины.
Чудеса, учитывая, что отладки адекватной для ардуины нет.
Этот вопрос еще предстоит освоить подробнее, но вот промежуточные результаты.
Накидал простецкую схему:

![Схема симуляции](https://i.ibb.co/M7kGqQY/2019-09-17-20-17-10.png)

После "загрузки" в виртуальную ардуину исходников, можно посмотреть на сигнал, образующийся на выходах контроллера.
Я ожидал увидеть постепенно сужающийся прямоугольный сигнал.
Результат снова разочаровал:

![Результат симуляции](https://i.ibb.co/1Knx3dZ/2019-09-17-20-15-39.png)

## Прототип, его работа

Несмотря на то, что моделирование не принесло желаемого результата, решил попробовать запустить на реальной ардуинке, отправляя значения задержек в порт, чтобы потом снова построить график:

![результат выполнения кода на реальной ардуине (данные собраны через последовательный порт](https://i.ibb.co/Dt8m5wZ/arduino-set-3-freq.png)

И почему-то здесь _тот же самый_ код при _тех же самых_ параметрах работы досчитал таки до максимальной скорости вращения, хоть и не линейно.

Чертовшина какая-то

![](https://vignette.wikia.nocookie.net/anime-characters-fight/images/4/4e/Jim_Hawkins_%283%29.JPG/revision/latest?cb=20161006134430&path-prefix=ru)
