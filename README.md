# Technopark_Highload
Технопарк. Проектирование высоконагруженных систем. Семестровый проект "Discord"

Бесплатный проприетарный мессенджер Discord для голосового общения посредством голосовых конференций

## MVP
1. Регистрация и авторизация
2. Голосовые конференции

## Целевая аудитория
~150 миллионов пользователей в месяц со всего мира использует Discord. Подавляющая часть пользователей располагается в США, а также в Европе, и в России.

~14 миллионов пользователей в день посещают Discord.

Суммарно ~4 миллиарда минут ежедневно используется голосовыми конференциями. Пиковая нагрузка составляет ~2,78 миллионов пользователей, общающихся в голосовых конференциях одновременно.

Трафик голосовых конференций (битрейт) составляет 791,9 килобит в секунду на одну голосовую конференцию.

## Расчёт нагрузки
Можно пренебречь нагрузкой запросов на регистрацию и авторизацию, поскольку пользователи довольно редко выполняют их по сравнению с остальными запрсами, касающимися основной функциональности Discord.

### Продуктовые метрики
Средний размер хранилища одного пользователя, выделенный для его авторизационных данных, учитывая, что он залогинен в один момент с одного устройства, составит (были учтены размеры полей в БД):

    ~248 (б)

Средний размер хранилища, выделяемый для одной голосовой конференции, составит:

    ~32 (б)

Средний размер хранилища, выделяемый под информацию о том, что определённый пользователь находится в определённой конференции, составит:

    ~16 (б)

Есть следующие виды запросов: создание голосовой конференции, вход в голосовую конференцию, выход из голосовой конференции, удаление голосовой конференции. Можно учесть, что каждый пользователь в среднем создаёт голосовую конференцию один раз в неделю, удаляет одну голосовую конференцию один раз в месяц, входит в голосовую конференцию и выходит из неё по 2 раза в день соответственно.

### Технические метрики
Средний размер хранилища авторизационных данных всех пользователей составит:

    150 (миллионов зарегистрированных пользователей) * 248 (б) = 37200000000 (б) ~= 34,6 (Гб)

Средний размер хранилища голосовых конференций всех пользователей, если учесть пиковую нагрузку (примерно каждый 54-й пользователь находится в голосовой конференции в каждый момент) и что в среднем каждый пользователь является создателем двух голосовых конференций, составит:

    150 (миллионов зарегистрированных пользователей) * (1/54 (часть пользователей) * 16 (б) + 32 (б) * 2 (голосовые конференции)) ~= 9644444444,4 (б) ~= 9 (Гб)

Трафик, использованный голосовыми конференциями при пиковой нагрузке, составит:

    2780000 (пользователей, общающихся в голосовых конференциях одновременно) * 791,9 (килобит в секунду) ~= 2 (терабит в секунду)

Количество запросов, выполняемых на создание голосовых конференций в день, учитывая, что в среднем каждый седьмой посетивший Discord в определённый день пользователь создаёт одну конференцию, составит:

    14000000 (миллионов пользователей в день) * 1/7 (часть пользователей) = 2000000 (запроса в день)

Количество запросов, выполняемых на удаление голосовых конференций в день, учитывая, что в среднем каждый 30-й посетивший Discord в определённый день пользователь удаляет одну конференцию, составит:

    14000000 (миллионов пользователей в день) * 1/30 (часть пользователей) ~= 466667 (запросов в день)

Количество запросов, выполняемых на вход в голосовую конференцию и выход из неё, учитывая, что каждый пользователь в среднем по 2 раза в день выполняет каждый запрос соответственно, составит:

    14000000 (миллионов пользователей в день) * 4 (запроса) = 56000000 (запросов в день)

RPS, учитывая среднее количество выполнения каждого вида запроса, составит:

    (2000000 + 466667 + 56000000) / 24 (часа) / 60 (минут) / 60 (секунд) = 676 (RPS)

## Логическая схема

### ER-модель БД
![image](https://raw.githubusercontent.com/NikitaLobaev/Technopark_Highload/main/ER-model3.png)

#### Описание сущностей

Сущность USER — сильная и представляет собой пользователя. Она обладает следующим набором атрибутов: id (идентификатор), email, username (имя пользователя), password (пароль), birthday (дата рождения). Данная сущность идентифицируется по атрибуту id.

Сущность SESSION — слабая (зависит от сущности USER) и представляет собой авторизационную сессию пользователя. Она обладает следующим набором атрибутов: id (идентификатор), expires (дата окончания жизни сессии). Данная сущность идентифицируется по атрибуту id.

Сущность CONFERENCE — слабая (зависит от сущности USER) и представляет собой голосовую конференцию. Она обладает следующим набором атрибутов: id (идентификатор), slug (семантический URL). Данная сущность идентифицируется по атрибуту id.

Сущность USER_CONFERENCE — слабая (зависит от сущностей USER и CONFERENCE) и представляет собой участие пользователя в голосовой конференции. Она обладает следующим набором атрибутов: id (идентификатор). Данная сущность идентифицируется по атрибуту id.

#### Описание связей
Связь USER_SESSION типа «один ко многим» связывает сущности USER и SESSION. Участие сущности SESSION в данной связи является полным, USER — частичным.

Связь CONFERENCE_AUTHOR типа «многие к одному» связывает сущности CONFERENCE и USER. Участие сущности CONFERENCE в данной связи является полным, USER — частичным.

Связь UС_USER типа «многие к одному» связывает сущности USER_CONFERENCE и USER. Участие сущности USER_CONFERENCE в данной связи является полным, USER — частичным.

Связь UC_CONFERENCE типа «многие к одному» связывает сущности USER_CONFERENCE и CONFERENCE. Участие сущности USER_CONFERENCE в данной связи является полным, CONFERENCE — частичным.

### Реляционная модель БД
![image](https://raw.githubusercontent.com/NikitaLobaev/Technopark_Highload/main/relational-model2.png)

## Физическая схема
Учитывая, что при пиковой нагрузке размер хранимых данных в один момент не превышает даже 1 Тб, можно хранить всё сразу в одной БД. СУБД PostgreSQL является хорошим решением как одна из самых популярных реляционных СУБД с открытым исходным кодом в настоящее время. Поскольку подавляющая часть пользователей располагается в США, СУБД также следует разместить в США.

Для реализации потокового аудио, учитывая пиковую нагрузку, потребуется:

    2 (терабит в секунду) / 10 (гигабит в секунду) ~= 205 (сетевых карт)

Учитывая, что в среднем один сервер может располагать двумя сетевыми картами, потребуется:

    205 (сетевых карт) / 2 ~= 103 (сервера)

Учитывая 20%-й запас, потребуется:

    103 (сервера) + 20% ~= 124 (сервера)

Таким образом, в среднем в США, в Европе и в России следует разместить по ~41 серверов (в Европе серверы можно любым удобным способом равномерно распределить по нескольким странам, строгими распределениями можно пренебречь).

### Балансировка узлов голосовых конференций
Аудиопотоки каждой голосовой конференции проходят только через один определённый сервер (BACKEND #2). Выбор сервера происходит поэтапно:

- Пользователь отправляет запрос на DNS-сервер для определения IP-адреса сервера, на который необходимо отправить HTTP-запрос на создание голосовой конференции
- DNS-сервер определяет ближайший к пользователю сервер (FRONTEND #1) и в ответ на полученный запрос отправляет его IP-адрес
- Пользователь отправляет HTTP-запрос на создание голосовой конференции уже на определённый сервер (FRONTEND #1)
- Сервер (FRONTEND #1) получает данный запрос и выполняет его переадресацию на сервер BACKEND #1
- Сервер (BACKEND #1) получает данный переадресованный запрос, отправляет запрос в СУБД на создание голосовой конференции, в котором указывает свой IP-адрес (IP-адрес сервера указывается в поле host таблицы conference), и отвечает пользователю сообщением об успешном создании голосовой конференции (или сообщением об ошибке)

Подключение пользователей в любую голосовую конференцию будет происходить следующим образом (учитывая, что пользователю уже известно, к какой конкретной голосовой конференции ему необходимо подключиться, то есть, ему известен её идентификатор или slug):

- Пользователь отправляет запрос на DNS-сервер для определения IP-адреса сервера, на который необходимо отправить HTTP-запрос на подключение к голосовой конференции
- DNS-сервер определяет ближайший к пользователю сервер (FRONTEND #1) и в ответ на полученный запрос отправляет его IP-адрес
- Пользователь отправляет HTTP-запрос на подключение к голосовой конференции уже на определённый сервер (FRONTEND #1)
- Сервер (FRONTEND #1) получает данный запрос и выполняет его переадресацию на сервер BACKEND #1
- Сервер (BACKEND #1) получает данный переадресованный запрос, отправляет запрос в СУБД на получение информации об искомой голосовой конференции (IP-адреса сервера, через который осуществляется трансляция её потокового аудио; список пользователей, подключённых в данный момент к голосовой конференции; и т.п.), и отправляет полученные данные об искомой голосовой конференции (или сообщение об ошибке) в ответ на запрос пользователя
- Пользователь, зная IP-адрес сервера (BACKEND #2), через который осуществляется трансляция потокового аудио, устанавливает соединение с ним по протоколу UDP и начинает трансляцию и получение потокового аудио голосовой конференции

### Схема проекта

![image](https://raw.githubusercontent.com/NikitaLobaev/Technopark_Highload/main/project-scheme3.png)

#### Дополнительные пояснения по схеме

На схеме FRONTEND #1 и BACKEND #1 отвечают за авторизацию пользователей, FRONTEND #2 и BACKEND #2 отвечают за трансляцию голосовых конференций.

Связь между BACKEND #2 и PostgreSQL необходима для аутентификации пользователей.

#### Схема ролей

| Роль        | ОЗУ    | CPU | Жёсткий диск | Количество                                 |
| ----------- | ------ | --- | ------------ | ------------------------------------------ |
| FRONTEND #1 | 32 Гб  | 32  | 32 Гб        | 5                                          |
| BACKEND #1  | 32 Гб  | 96  | 32 Гб        | 5                                          |
| BACKEND #2  | 256 Гб | 96  | 32 Гб        | 124 (42 в США и по 41 в Европе и в России) |
| PostgreSQL  | 32 Гб  | 32  | 128 Гб       | 1 в США                                    |

## Ссылки на источники информации
- https://discord.com/brand-new/company
- https://vc.ru/services/73312-messendzher-dlya-geymerov-discord-oboshel-skype-i-zanyal-chetvertoe-mesto-po-auditorii-na-pk-v-rossii-v-aprele-2019-goda
- https://discordgid.ru/skolko-internet-trafika-tratit/
