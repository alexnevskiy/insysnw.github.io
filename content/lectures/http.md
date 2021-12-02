+++
title = "Hypertext Transfer Protocol"
description = "Протокол прикладного уровня в качестве транспорта"
weight = 4
+++

До этого в курсе были рассмотрены самодостаточные протоколы прикладного уровня, т.е. они решали конечную задачу: передавали файлы, настраивали узлы сети, конвертировали доменные имена в IP адреса и наоборот.
В этой лекции мы выясним, что один из них ещё является отличным транспортом.

Ещё в 1989 Tim Berners-Lee начал работать над идеей передачи и отображения текста.
Но это должен быть не просто текст, а текст со ссылками на другие тексты.
Особенность этих ссылок в том, что текст может находиться на совсем другом ресурсе.
Такую концепцию назвали Hypertext, что подразумевало "больше чем текст".
Сердцем этой идеи является протокол **HyperText Transfer Protocol**.
Первая его версия ныне имеет номер **0.9**.
Она хорошо подходила для решения вышеописанной задачи.

Страницы легко отобразить даже без браузера (а их тогда под большинство платформ и не было):

```
$ telnet academy.ejiek.com 80
Trying 188.242.22.225...
Connected to academy.ejiek.com.
Escape character is '^]'.
GET /about_http/page.html
<html>
  <p>In the first age</p>
  <p>In the first 18 tags</p>
  <p>When the hypertext first lenghtened</p>
  <p>One stood</p>
</html>
Connection closed by foreign host.
```

Всё, что делает `telnet` — открывает TCP сессию до узла с IP адресом, в который резолвится `academy.ejiek.com`.
Дальше мы сами пишем `GET /about_http/page` и отправляем в поток, на что сервер выдаёт всё содержимое страницы и закрывает TCP сессию.

**Текст получен, задача выполнена!**

Однако у версии **0.9** есть ещё один трюк в рукаве - [поиск](https://www.w3.org/Addressing/Search.html)!
К сожалению, мы ещё не смогли воспроизвести его работу.
Для работы поиска на сервере должен быть создан специальный индекс файл.
Клиенту необходимо указать путь до индекс файла, как до обычной страницы, а затем передать список ключевых слов:

```
http://academy.ejiek.com/about_http/index?key+words
```

Такой способ построения запроса сейчас достаточно популярен.
Дополнительные параметры отделяются от основного пути вопросительным знаком, а сами дополнительные параметры (иногда их называют ключи) разделены знаками плюс.

# Версия 1.0

На сцену выходят браузеры!
А с ними и новые потребности.
Простого отображения текста оказалось мало, возникла **потребность во взаимодействии**.
Так появились **типы запросов** (POST, GET, PUT, DELETE).
Но для адекватного взаимодействия сервера и клиента только типов запросов недостаточно, нужен **стандартизированный, понятный машине способ обратной связи**, чтобы сигнализировать об успешном выполнение запроса или наоборот об ошибке.
Тело запроса для этого не подходит, ведь оно может быть каким угодно и ориентировано на человека.
Знакомьтесь, **коды состояния**!
Все из вас наверняка неоднократно видели код `404` ("страница не найдена").
Но мы редко видим коды `2**` или `1**`, хотя сталкиваемся с ними значительно чаще.

Это связано с тем, что эти коды ориентированы на взаимодействие машина-машинa.
Так браузер или любой другой клиент понимает, что что-то пошло не так и может это обработать, а для человека иногда эти коды дублируют в теле ответа.
К слову, это довольно распространённая ошибка: разработчики сервера возвращают правильный код состояния, например, `404`, но в тексте тела ответа, для человека, указывают что-то иное, например, `502`.
Это либо случайная ошибка, либо преднамеренное введение пользователя в заблуждение, что мы не одобряем.
Рядовому пользователю этот код совсем не нужен, в теле ответа достаточно просто показать, что это за ошибка (страница не найдена, нет прав доступа и или любая другая).
А коды успешного подтверждения совсем не принято дублировать в теле ответа, ведь пользователь и так получил то, что хотел.


Подробнее про коды состояний можно почитать в RFC, на сайте [httpstatuses.com](https://httpstatuses.com/) или на [Wikipedia](https://ru.wikipedia.org/wiki/%D0%A1%D0%BF%D0%B8%D1%81%D0%BE%D0%BA_%D0%BA%D0%BE%D0%B4%D0%BE%D0%B2_%D1%81%D0%BE%D1%81%D1%82%D0%BE%D1%8F%D0%BD%D0%B8%D1%8F_HTTP) ([EN](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)).
Мы лишь обсудим, что эти коды делятся на 5 типов.
У каждого типа своя первая цифра:

* 100 - информационные (самое заметное — смена протокола, используется для WebRTC и Web-сокетов)
* 200 - успешное выполнение
* 300 - перенаправление (http ->  https, страница переехала)
* 400 - клиентские ошибки
* 500 - серверные ошибки

## Составные страницы

Из-за активной популяризации браузеров, просто текста c ссылками быстро стало недостаточно.
Страницы начали содержать не только текст, но и медиа контент, например, картинки.
Но и это не просто медиа, это HyperMedia.
Это же замечательные новости!
Конечно, однако есть довольно большое "НО".

В **0.9** на каждый запрос открывается новое TCP соединение.
То есть для страницы, состоящие из множества ресурсов, необходимо установить столько tcp сессий, сколько на ней ресурсов.
Кажется, что это не очень страшно, пока ресурсов на странице единицы, но вполне обыденно, когда их сотни.
Это достаточно сложно и долго для клиента, открывающего один сайт, и почти невыносимо для сервера, обслуживающего тысячи клиентов.
Решение простое — не закрывать TCP сессию, ведь большинство ресурсов будут получены с одного и того же сервера.
Но нельзя просто так взять и поменять поведение протокола.
Это ломает обратную совместимость.
Нужно как-то договориться с сервером, чтобы он не закрывал соединение.
Обсудим, как именно это сделать, уже в следующем разделе.

# Заголовок

На данным момент мы перечислили три важным параметра, которые нужно где-то хранить в HTTP пакете:

* тип запроса
* код состояния
* просьбу не закрывать соединение

Название этого раздела говорит само за себя — Заголовок!
Это набор данных, передаваемый в начале HTTP сообщения, до тела.
Всё дополнительная информация помещается именно туда.

На данном этапе нам достаточно знать, что есть список утверждённых заголовков, но в мире используют не только они.
Иногда заголовки из второй категории попадают в первую.
А ещё они логически делятся на 4 группы:

* Request — специфичные для запросов
* Response — специфичные для ответов
* Entity — описывающие содержимое тела
* General — все остальные


**TODO**: описать Augmented BNF из [rfc 1945](https://tools.ietf.org/html/rfc1945).

В качестве стандартного General заголовка в версии **1.0** появился `connection: keep-alive`.
Такой заголовок в запросе отправляет клиент и если он будет и в ответе сервера, то переговоры удалась и сессия не будет закрыта.

Заголовки типа Entity позволили значительно легче передавать и отображать медиа файлы.
Самый известный из них — `Content-Type`, он хранит тип файла, MIME.
Именно он позволяет понять передаём содержится ли в теле текст, картинка или JSON.

# Версия 1.1

Несмотря на то, что это версия протокола появилась ещё в 1997 году, она до сих пор является основополагающей.
*Но от не стагнировал все эти годы, RFC подвергался обновлениям и полной замене, например, в 2014 году.

Помните `keep-alive`?
Теперь это поведение по умолчанию!

Среди стандартных заголовков появился `Host`.
Если раньше доменное имя терялось (оно не попадало в http сообщение), то теперь оно в нём дублируется.
Это открывает возможность размещать разные доменные адреса с разным содержимом на одном IP адресе.
Именно так работает сайт, который вы сейчас читаете, он дели свой IP в другими сайтами!
Возможность использования Reverse Proxy

# AJAX
При всей сложности составных страниц и крутезне передачи в одной TCP сессии, вопрос перехода на другую страницу остался не затронутым.
При переходе нас всё ещё ждёт загрузка новой страницы целиком.

Чем это плохо?
Когда-то браузеры сопровождали переход полной перерисовка, что на иногда короткое время выглядело как пустая белая страница.
Это особенно печальный пользовательский опыт, если использовать тёмную тему ночью.
Стоит заметить, что такую проблему мы уже давно не замечали.

А вот проблема нагрузки на сеть всё ещё актуальна.
Ведь новая страница потребует элементов пользовательского интерфейса, которые уже присутствуют на текущей.
CSS и некоторые другие ресурсы возможно кешировать, но не HTML.

[Asynchronous JavaScript and XML](https://en.wikipedia.org/wiki/Ajax_(programming)) общее название технологий и техник для динамического обновления сайтов с использованием XML и JS.
Отсюда начинается долгий путь к web приложениям.
Главное изменение - возможность перейти на другую страницу, загрузив только ту часть, которая изменится.

Часто под частью подразумевается, не кусочек страницы, а данные, необходимы для его формирования (без элементов пользовательского интерфейса).
Например, в почтовом ящике при переходе из одной папки в другую, у сервера будет запрошен список писем, а не HTML этим списком.

Подобная практика запроса данных стала достаточно популярной.
Разработчики нашли форматы хранения данных удобнее XML.
Так концепция AJAX осталась в прошлом, но дало начало использованию HTTP не как средства передачи HyperText или HyperMedia, а транспорта.
Сейчас на HTTP строят как тестовые, так и бинарные протоколы обмена данным.

# Single Page Application
SPA - следующая ступень развития оперирования данными.
Вместо, знакомых из AJAX, загрузки страницы сайта и последующего обновления лишь части, в SPA вместо сайта загружается приложение.
Это приложение содержит в себе множество страниц сайта и в некоторых случая работает оффлайн.
За переход между страницами ответственен компонент Router.
Он говорит приложению, какую страницу хочет увидеть пользователь, а браузеру - адрес.
Так без реальных переходов между страницами, а лишь изменением вида одной, создаётся иллюзия настоящего перехода.
Однако для таких манипуляций с историей требуется поддержка более глубокой интеграции со стороны браузера.

Удобство такого приложение в том, что оно использует HTTP как транспорт для данных.
Это, как и прежде, экономия сетевого канала, но и вероятное пересечение API c мобильной и desktop версией приложения.
Что в свою очередь упрощает и удешевляет backend разработку.

# HTTP 2
С точки зрения функциональности версия **1.1** уже удовлетворяла все потребности, но горизонты оптимизации ещё не были покорены.
Среди всего, что принесла вторая версия HTTP, два новшества особенно интересны:

* мультиплексирование
* бинарный заголовок

Разберёмся с каждой отдельно и по порядку.
В версии **0.9** обращение за каждым ресурсом требовало отдельную TCP сессию.
В **1.0** и **1.1** можно было все ресурсы от одного сервера получить в рамках одной TCP сессии, но это последовательный процесс.
Проблема заключается в том, что сервер может выдавать некоторые ресурсы значительно дольше других.
Так ресурсы, которые могли бы быть отправлены клиенты относительно быстро могут ждать в очереди, пока не будет передан долгий ресурс.
Долгим он может быть не только из-за объёма, но из-за того, что его процесс получения на сервере значительно дольше.
Например, его нужно получить из удалённой базы данных, что ощутимо дольше статических данных, находящихся в кеше веб-сервера.
Безусловно, можно выставить наиболее оптимальный порядок загрузки ресурсов, но целиком проблему это не решит, так как время получения ресурса часто не является константным.

Ответом на эту проблему в версии 2 стало мультиплексирование.
То есть разделение TCP потока между несколькими ресурсами.
Это позволяет запросить сразу несколько ресурсов и начать получить их параллельно.
Помимо того, что "долгие" ресурсы теперь не блокируют более быстрые, этап поиска их сервером теперь может происходить параллельно, что ещё больше ускоряет процесс.

Второе нововведение — бинарный заголовок.
Но стойте, не логичнее ли сделать бинарным тело HTTP сообщения?
Логично, ведь обычно оно значительно более объёмное, чем заголовок!
Вот только с появлением заголовком, особенно их типа Entity, тело могло быть бинарным и часто именно таковым и было.
Картинки, видео, аудио и так бинарные.
И нет ничего, что мешает вместо JSON'a использовать бинарный формат представления данных.
А вот заголовок всегда был просто текстом.
Теперь заголовок быстрее (де)кодируется и занимает меньше места!

# HTTP 3
Теперь, года бинаризировли и распараллели все, что было, настало время копать глубже!
Проблемы не заставила себя долго ждать.
Если в крутой TCP сессии с убирающим лишнее ожидание мультиплексированием возникают потери, то на помощь приходит механизм восстановления потерь.
Это процесс достаточно сильно замедляющий передачу, но главное — он ставит палки в колёса сразу всем передаваемым ресурсам, ведь они живут в рамках одного TCP потока.

Вторая интересная область установление сессии.
Что не так с установлением сессии в HTTP 2?
Его больше, чем нужно!
Сначала мы устанавливаем TCP сессию для работы транспортного уровня, а затем TLS сессию для обеспечения шифрования.

Мы нашли две проблемы указывающие на TCP.
А ещё в версии **0.9** говорилось, что для HTTP в целом подойдёт любой сессионный протокол, необязательно TCP.
Настало время пробовать что-то помимо TCP!
Проблема с чем-то помимо TCP лишь в том, что других поддерживаемых во всём мире сессионных протоколов транспортного уровня нет.
Это значит, что все нововведения будут плохо встречены из-за отсутствия поддержки на конечных устройствах, маршрутизаторах, фаеврволах и не только.
IPv6 предложили в декабре 1995, а он всё ещё не повсеместен.
Решение конкретно этой проблемы оказалось достаточно простым!
Использовать UDP как транспорт, его все уже поддерживают.

Так в недрах Google появился протокол **QUIC**, что когда-то было аббревиатурой от *Quick UDP Internet Connections*, но более таковой не является.
Аббревиатура подчёркивала игрой слов (QUIC читается как "quick") основную особенность, протокол должен быть быстрее.
От части он добивается он этого решением двух, упомянутых ранее, проблем.

Установление QUIC сессии подразумевает и установление шифрование.
Объединения двух рукопожатий не просто экономит время, но и позволяет кэшировать на клиенте поддерживаемые сервером алгоритмы шифрования, что позволяет ещё больше ускорить процесс установления сессии.

После установления сессии, QUIC позволяет создать множество потоков (streams), каждый из которых самостоятельно занимается обеспечением гарантий доставки.
Теперь, если вмешиваются проблемы в сети, то есть шансы, что пострадают не все потоки.
Дополнительный бонус вынесения сессии из транспортного уровня — смена IP адреса клиента не требует смены сессии, её можно перенести на новый IP.

**TODO**: расписать реализацию в UserSpace, а не в ОС, и сложность адаптации нового протокола.