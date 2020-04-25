## Геоинформационная система локализации пользователя в пространстве с использованием машинного обучения

### Идея 1.  
Hololens + 3DSmoothNet (https://arxiv.org/abs/1811.06879)  + Teaser++ (https://arxiv.org/abs/2001.07715)  
ИЛИ  
Hololens + ЗPointNetVLAD (https://arxiv.org/abs/1804.03492)  

Основной принцип - представить всё пространство в виде **облака точек**.  
Затем мы сможем выяснять своё положение относительно записанной глобальной карты, используя небольшой локальный скан ближайшего к нам пространства. Так, например, работает навигация в современных роботах.  
(https://github.com/raulmur/ORB_SLAM2  
https://www.researchgate.net/publication/308501428_An_efficient_scan-to-map_matching_approach_for_autonomous_driving)  
  
  
Метод, который я хочу для этого использовать, называется **point cloud registration**  
  
#### Что такое **point cloud registration**:  
![](https://raw.githubusercontent.com/neka-nat/probreg/master/images/filterreg_fpfh.gif)  
![](https://github.com/zgojcic/3DSmoothNet/raw/master/figures/demo.png?raw=true)
![](http://geometryhub.net/images/globalregistration.jpg)
  
  
  
#### Схема картографирования
Введём три понятия.  
**Точка интереса** - сведения о положении некоторой точки, а также дополнительной информации, относящейся к ней или к окружающему её пространству. Такие точки будут использоваться для сохранения тех мест и ориентиров, между которыми будет нужно строить маршруты.  
![Точка интереса](https://github.com/wndenis/Innav/raw/master/media/poi2.jpg "Точка интереса")  
  

**Точка навигации** - сведения о положении точки, через которую можно строить маршрут. Так как для простоты системы не производится семантический анализ геометрии для определения пола и стен, эти точки помогут строить корректные маршруты, которым бы пошел человек. То есть, по полу и через двери, а не по воздуху и через стены.  
![Точка навигации](https://github.com/wndenis/Innav/raw/master/media/navpoint2.jpg "Точка навигации")
  
**Геометрия карты** - все данные о пространстве, записанные Hololens. В дальнейшем будут приведены в вид Point cloud для хранения.  
  
Для создания карты пользователь активирует этот режим голосовой командой. После чего сведения о **геометрии карты** и **точках навигации** начинают сохраняться автоматически, пользователь должен просто пройти по всем местам, которые он хотел бы добавить на карту.  
![Точки навигации](https://github.com/wndenis/Innav/raw/master/media/navpoints.jpg "Точки навигации")
  
  
Для создания пунктов назначения пользователь создаёт **точки интереса**, используя жест "Air tap", направив взгляд на поверхность, на которой должна быть создана точка.  
<p align="center"><img src="https://docs.microsoft.com/ru-ru/hololens/images/hololens-air-tap.gif"></p>  
  
После этого он может задать имя точки, сохранить её или отменить её создание.  
При использовании этого жеста на уже созданной точке открывается интерфейс редактирования этой точки.  
![Редактирование точки интереса](https://github.com/wndenis/Innav/raw/master/media/poi.jpg "Редактирование точки интереса")  
  
  
В процессе картографирования пользователю доступна миникарта отсканированного пространства, на которой отмечено местоположение пользователя.  
![](https://github.com/wndenis/Innav/raw/master/media/minimap.jpg)
  
  
После окончания картографирования пользователь останавливает его с помощью голосовой команды, что автоматически вызывает приведение всех собранных данных в нужный для использования вид и их последующую отправку на сервер.  
![](https://github.com/wndenis/Innav/raw/master/media/map_creation.png)  
  
  
Фото пространства и фото того же самого пространства с точки зрения пользователя:  
![Как выглядит пространство](https://github.com/wndenis/Innav/raw/master/media/raw.jpg "Как выглядит пространство")  
![Готовая к отправке карта](https://github.com/wndenis/Innav/raw/master/media/navpoint.jpg "Готовая к отправке карта")  
  
  
#### Схема навигации
Для начала навигации пользователь использует голосовую команду, что влечет за собой автоматический сбор данных о **геометрии пространства** вокруг него. Пользователь получает уведомление о том, что нужно немного пройтись, чтобы отсканировать больше пространства для более точного совмещения. Когда данных о **геометрии пространства** достаточно, они автоматически отправляются на сервер для регистрации относительно карты.  
  
*Сервер необходим из-за ограниченных вычислительных возможностей мобильных устройств.*  
  
Сервер использует 3DSmoothNet (или иное решение, представленное выше), которое, в отличие от алгоритмических подходов, задействует машинное обучение и даёт более качественный результат.  
В качестве ответа сервер возвращает матрицу смещения для перевода локальной системы координат пользователя в систему координат карты, а также сведения о положениях **точек навигации** и **точек интереса**.  
Пользователь может корректно разместить эти точки относительно себя, используя предоставленную сервером матрицу, после чего взаимодействие с сервером может потребоваться только при изменении локальной системы координат. Такое бывает редко, но происходит, когда датчки Hololens оказываются в условиях недостаточной освещённости или их поле зрения чем-нибудь ограничено.  
![](https://github.com/wndenis/Innav/raw/master/media/map_usage.png)  
  

#### Дополнительно:
На данный момент решение не предусматривает длительного взаимодействия с сервером, однако оно с лёгкостью может быть доработано для создания Shared experience и мультипользовательских систем. Например, для совместного взаимодействия нескольких пользователей с одним и тем же AR-контентом. Можно отображать других пользователей, находящихся в том же здании. Кроме того, привязанный к точке интереса контент может быть динамическим и не только текстовым, что позволит в одной и той же точке интереса отображать различный и персональный контент для каждого пользователя.  

#### Плюсы:
- Новое решение
- Одно решения для создания и для использования карты
- Карта может обновляться автоматически
- Не чувствителен к небольшим изменениям пространства
- Начало навигации в любой точке
- Использует машинное обучение (соответствие тематике)
- Почти State of the art
- Все источники 2018-2020 года и на иностранном языке
- Мультидисциплинарная работа
- Задействован Hololens
- Потенциально можно удешевить, когда устройства, как Hololens, будут доступнее
  
#### Минусы:
- Сложная экономическая часть - сейчас это дорого
- Не уверен, что успею реализовать все аспекты этой системы
  
#### Прогресс:
##### Клиент, реализовано:
- Сбор данных о пространстве (mesh + point cloud)
- Сбор данных о точках навигации (автоматически расставляются под ногами)
- Интерфейс для создания, редактирования и удаления точек интереса
- Навигация с использованием локальной карты
- Визуализация миникарты пространства с указанием местоположения пользователя
  
##### Клиент, не реализовано:
- Обмен данными с сервером (так как пока сервер работает не так, как надо)
  
##### Сервер, реализовано:
- Чтение данных
- Протестирована работа на данных разработчиков
  
##### Сервер, не реализовано:
- Работа (совмещение point cloud) на своих данных
  
---
### Идея 2.  
ARCore

Основной принцип - разместить в пространстве заранее определённые метки (QR-коды), создать базу данных, в которой для метки будет сохранено её положение.
Для начала навигации нужно найти ближайшую метку, относительно которой будет строиться навигация.

#### Плюсы:
 - Легко реализовать
 - Дёшево
 - Доступно

#### Минусы:
 - Не требует дополнительного машинного обучения (Не соответствует тематике)
 - Должна быть нарисована карта в 2D
 - Только для навигации, карта создаётся вручную отдельно
 - При смещении меток будут появляться ошибки навигации
 - Метки должны быть расположены не слишком далеко друг от друга, так как при удалении от метки может начать накапливаться ошибка.
 - При потере трекинга нужно снова искать метку
 - Зависит от условий освещения
 - Подобное уже делали даже школьники (https://habr.com/ru/post/496356/)
