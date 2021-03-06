# ROS узлы и топики (основное)

> [Ссылка на YouTube-видео - Узлы и топики](https://www.youtube.com/watch?v=5jxTVYdbVX0&list=PLdRYu473gKJQAJ6ifxldnc4vgpuisu5L5&index=5)

> [Google-презентация (слайды 1 - 4)](https://docs.google.com/presentation/d/1dAujwqXbiVhfxTfEow039pG2TtA19IUhpYOKfDjma68/edit#slide=id.g43adc50189_0_0)

Вот и подошел черед познакомиться с первыми базовыми вещами ROS - __узлами__. Под узлом подразумевается программа, которая производит некоторые действия. Узлы соединяются __топиками__, через которые передают __сообщения__. Таким образом, приходим к знакомой схеме типа графа.  

Для начала знакомства необходимо запомнить важную вещь!  
> В любой экосистеме ROS существует мастер (и он единственный), который работает с узлами. Фактически, он полностью организует их.  

Для более подробного описания предлагается поглядеть на офф страницы об [узлах](http://wiki.ros.org/Nodes) и [топиках](http://wiki.ros.org/Topics).

Начинаем с практики - запускаем следующее, чтобы создать простейшую систему с топиком и узлами:
- запуск ROS мастера, вся систем зиждется на нем и без него ничего не работает
`roscore`
- узел из пакета `rospy_tutorials` с именем `talker` командой
`rosrun rospy_tutorials talker __name:=talker`
- узел из пакета `rospy_tutorials` с именем `listener` командой
`rosrun rospy_tutorials listener __name:=listener`

> Знак '`:=`' означает присваивание и будет далее часто использоваться при мапировании, установки значений. Это стоит запомнить =) 

##### > Запустите указанные утилиты. Не забывайте про то, что каждая утилита (узлы и мастер) при запуске бесконечно работают, поэтому запуск производится в отдельных терминалах (всего насчитал 3 штуки)

`rosrun` - утилита запуска узла, имеет следующее описание:  
`rosrun [pkg_name] [node_name] [additional args]`  
- pkg_name - имя пакета, в котором узел содержится  
- node_name - имя узла в пакете  
- additional args - дополнительные аргументы (рассматривается позже, в нашем случае используется явное задание имени узла в экосистеме ROS (`__name`) )

---
## Основные инструменты

Открываем еще один терминал и начинаем анализ системы вместе с испытанием инструментов.

##### > Проверьте возможные команды утилиты `rosnode`, вспомните как вывести помощь по утилите =)

Посмотрим список узлов в системе с помощью

```bash
rosnode list
```

Также можно посмотреть информацию о конкретном узле, например `/talker`

```bash
rosnode info /talker
```

```
--------------------------------------------------------------------------------
Node [/talker]
Publications: 
 * /chatter [std_msgs/String]
 * /rosout [rosgraph_msgs/Log]

Subscriptions: None

Services: 
 * /talker/get_loggers
 * /talker/set_logger_level


contacting node http://user-vb:39115/ ...
Pid: 8733
Connections:
 * topic: /chatter
    * to: /listener
    * direction: outbound
    * transport: TCPROS
 * topic: /rosout
    * to: /rosout
    * direction: outbound
    * transport: TCPROS

```

Сделаем небольшой анализ увиденного:
- Узел зовется `/talker`
- Публикует в два топика: `/chatter` и `/rosout`, второй - топик для логирования, в него автоматически публикуют все узлы, более конкретно рассмотрим потом
- Подписок нет
- Имеет два сервиса, которые тоже создаются автоматически у каждого узла, о сервисах потом
- PID (Porcess ID) = 8733 - это идентификатор процесса в системе Linux, нам он не важен
- Имеет соединения, они должны сходиться с названиями подписок и публикаций, таким образом сообщается, что все публикуется и подписывается корректно (соединения между узлами настроены)

##### > Попробуйте проанализировать информацию об узле `/listener`

---
Теперь взглянем на возможности утилиты для анализа топиков

##### > Проверьте возможные команды утилиты `rostopic`, вспомните как вывести помощь по утилите =)

Посмотрим на список топиков в системе
```bash
rostopic list
```

Для вывода в более подробной форме воспользуемся опцией -v
```bash
rostopic list -v
```
```
Published topics:
 * /chatter [std_msgs/String] 1 publisher
 * /rosout [rosgraph_msgs/Log] 2 publishers
 * /rosout_agg [rosgraph_msgs/Log] 1 publisher

Subscribed topics:
 * /chatter [std_msgs/String] 1 subscriber
 * /rosout [rosgraph_msgs/Log] 1 subscriber
 ```

В выводе видна информация о подписках и публикациях топиков.

Взглянем на информацию о конкретном топике
```bash
rostopic info /chatter
```
```
Type: std_msgs/String

Publishers: 
 * /talker (http://user-vb:39115/)

Subscribers: 
 * /listener (http://user-vb:42123/)
```

По выводу можно определить, какие узлы подписаны на топик, а какие публикуют в него сообщения.

Далее можно показать сообщения, которые идут через этот топик. Для завершения нажмите Ctrl+C.

```bash
rostopic echo /chatter
```
```
data: "hello world 1536699023.89"
---
data: "hello world 1536699023.99"
---
...
```

##### > Найдите в справке утилиты `rostopic` аргумент измерения частоты публикации и измерьте частоту публикации в топик `/chatter`

---
Также существует утилита `rosmsg`, которая анализирует типы сообщений

##### > Проверьте возможные команды утилиты `rosmsg`

В информации о топике мы видели поле типа сообщений, которые идут через этот топик
```
Type: std_msgs/String
```
В типе можно видеть имя пакета типа `std_msgs` и само название `String`. Чтобы просмотреть, что содержит данный тип сообщения, воспользуемся командой
```bash
rosmsg show std_msgs/String
```
Результат:
```
string data
```
Как видно, данный тип сообщения содержит строковое поле `data`. Таким образом утилита анализирует информацию, связанную со строением (прототипом) сообщения, а не c самими данными в них.

А теперь посмотрим, какие есть сообщения в системе ROS =)
```bash
rosmsg list
```
Так как их много, вывод не приводится =)

Самый основной пакет сообщений - `std_msgs`, его типы можно глянуть по [ссылке](http://wiki.ros.org/std_msgs) или командой, которая показывает типы сообщений, определенные в данном пакете
```bash
rosmsg package std_msgs
```
```
std_msgs/Bool
std_msgs/Byte
std_msgs/ByteMultiArray
std_msgs/Char
std_msgs/ColorRGBA
std_msgs/Duration
std_msgs/Empty
std_msgs/Float32
std_msgs/Float32MultiArray
std_msgs/Float64
std_msgs/Float64MultiArray
std_msgs/Header
std_msgs/Int16
std_msgs/Int16MultiArray
std_msgs/Int32
std_msgs/Int32MultiArray
std_msgs/Int64
std_msgs/Int64MultiArray
std_msgs/Int8
std_msgs/Int8MultiArray
std_msgs/MultiArrayDimension
std_msgs/MultiArrayLayout
std_msgs/String
std_msgs/Time
std_msgs/UInt16
std_msgs/UInt16MultiArray
std_msgs/UInt32
std_msgs/UInt32MultiArray
std_msgs/UInt64
std_msgs/UInt64MultiArray
std_msgs/UInt8
std_msgs/UInt8MultiArray
```

##### > Пройдите все представленные шаги, просмотрите типы сообщений `std_msgs/ColorRGBA`, `geometry_msgs/Pose2D`

## Добавим графику

Запустите графическое представление графа экосистемы с помощью утилиты `rqt_graph`. Вы увидите графический интерфейс, на котором отображены узлы и топик в нашей системе. Также можно управлять настройкой группы `Hide`, что позволяет скрывать некоторые элементы в соответствии с настройкой.

##### > Включите `rqt_graph` и в группе `Hide` снимите галочку с `Debug`. Убедитесь в том, что представлены все узлы и топики (даже с отладкой) системы, с помощью утилит `rosnode` и `rostopic`. Как видно, узел `/rosout` и топик `/rosout` создаются для отладки и соединяются со всеми узлами через топик `/rosout`.

##### > Запустите узел пакета `rospy_tutorials` с именем `listener`, но присвойте имя `__name:=listener2`. Проверьте с помощью `rqt_graph`, что оба `listener` подключилиь на общий топик.

##### > Запустите узел пакета `rospy_tutorials` с именем `talker` и присвойте такое же имя `__name:=talker`. Убедитесь, что первый `talker` закрылся с сообщением о перехвате (`shutdown request: new node registered with same name`).

## В результате

- Научились запускать мастера.
- Научились запускать узлы с переопределением имени (более подробно об этом далее).
- Научились анализировать простейшую систему.
- Познакомились с утилитами `roscore`, `rosnode`, `rostopic`, `rosmsg`, `rqt_graph`.
