# boot2root

Задача: У нас есть дистрибутив BornToSecHackMe-v1.1.iso со стороны 42.fr в котором нужно получить права суперпользователя root.

1) Запустив дистрибутив мы можем обнаружить что на этот раз ip-адрес не высвечивается как это было в snow-crash, поэтому нужно будет его найти.

   - Зупускаем VirtualBox с уже установленной Kali, но прежде чем запустить дистрибутив BornToSecHackMe-v1.1.iso и Kali идем в Tools -> Network и создаем сетевую      карту. По умолчанию она будет называться ```vboxnet0```, в данном случае хост системой будет выступать наш iMac / macBook, посредиником будет выступать          VirtualBox, а гостевыми системами будут Kali и BornToSecHackMe-v1.1, после того как мы сделали сетевую карту мы идем в настройки наших машин ведь нам            нужно будет настроить им адаптер Host-only, что делает этот адаптер? Адаптер создает сеть между хост-системой и виртуальной машиной, минуя                        физическую сетевую карту. На компьютере появляется программный сетевой интерфейс, служащий для обмена данными между виртуальными машинами и хост-системой.        Виртуальные машины могут соединятся друг с другом и хост-системой, как будто соединены через коммутатор. Как и в режиме внутренней сети, виртуальной              машине не предоставляется физический интерфейс, благодаря чему машины не могут взаимодействовать с внешней сетью.
     В хост-системе появляется устройство VirtualBox Host-Only Network. Оно имеет собственную подсеть ```192.168.56.0``` и шлюз с адресом - ```192.168.56.1```.        Устройство соединяет подсеть и хост-систему без прямого выхода во внешнюю сеть.
     
   - Теперь нам нужно убедиться что мы действительно находимся в одной подсети, запускаем наши машины и все операции на этот раз мы будем производить с Kali,          аккуратный способ сделать это - пропинговать широковещательный адрес, который заполнит наш локальный кэш arp таблицы.
     ```
     ping 192.168.1.255
     ```
     Напоминаю: Широковещательный адрес — условный (не присвоенный никакому устройству в сети) адрес, который используется для передачи широковещательных пакетов      в компьютерных сетях.
     ```
     arp -a
     ```
     С нашей Kali можно сразу воспользоваться утилитой nmap для поиска уязвимостей и вбить следующее:
     ```
     nmap 192.168.X.XX-255
     ```
     Далее вычислив адрес BornToSecHackMe-v1.1, мы сканируем систему на наличие уязвимостей, а именно октрытых портов через nmap
     ```
     nmap 192.168.X.XX
     ```
     После это мы увидим следующее:
     ```
     Nmap scan report for 192.168.56.XX
     Host is up (0.0010s latency).
     Not shown: 994 closed ports
     PORT    STATE SERVICE
     21/tcp  open  ftp
     22/tcp  open  ssh
     80/tcp  open  http
     143/tcp open  imap
     443/tcp open  https
     993/tcp open  imaps
     ```
   - Итак что мы имеем? Мы видим открытые порты и понимаем что можем сделать, но пока что остается вопрос где взять учетные записи и пароли к ним? 
     Давайте сходим на 80 порт и посмотрим что у нас там есть? Вбиваем в строку поиска адрес ```192.168.X.XX:80```
     Далее мы попадаем на обычную интернет страничку на которой написано 'HACK ME' давайте попробуем поискать тут информацию, открыв панель разработчика в            надежде найти что-то интересное в html разметке я нашел ровно нихуя.
     Мы же помним что мы сидим с Kali? Ок, выпускайте ```Nikto```.
     
     Что такое Nikto? ```Nikto``` – это сканер с открытым исходным кодом (GPL) для веб-серверов, он выполняет комплексные тесты в отношении серверов по                нескольким направлениям, включая более 6700 потенциально опасных файлов/программ, проверка на устаревшие версии более 1250 серверов и проблемы, специфичные      для версий более чем 270 серверов. Сканер также проверяет элементы конфигурации сервера, такие как присутствие нескольких индексных файлов, серверные опции      HTTP и пытается определить имя и версии веб-сервера и программного обеспечения.

     На официальном сайте изменения замерли на 2.1.5 версии аж в 2012 году. Тем не менее, под руководством автора проект живёт на GitHub’е, пользователи              регулярно добавляют в базу данных и плагины изменения для сканирования новых уязвимостей, новых версий и т.д.

     Nikto не создавался быть незаметным. Он будет тестировать веб-сервер за самое быстрое возможное время, очевидно, что его активность попадёт в логи веб-          сервера и в поле зрение IPS/IDS (систем обнаружения/предотвращения вторжений). Тем не менее, имеется поддержка для анти-IDS методов из LibWhisker – на            случай, если вы захотите их попробовать (или протестировать вашу систему IDS).

     Не каждая проверка относится к проблеме безопасности, хотя большинство относятся. Некоторые пункты являются проверками типа «только для информации», которые      ищут вещи, может быть не имеющие брешей безопасности, но веб-мастер или инженер по безопасности могут не знать, что это присутствует на сервере. Обычно в        выводимой информации эти элементы помечены соответствующим образом. Есть также некоторые проверки на неизвестные элементы, которые были замечены в файлах        журналов.
     
     Теперь я прорекламировал Nikto и мне за это не заплатили.
     
     Вбиваем: ```nikto -h https://192.168.56.XX```
     
     Что мы получаем после? Мы получаем пару каталогов которые могут быть нам интересны, а это ```forum```, ```webmail```, ```phpMyAdmin```
     
     Теперь у нас есть маршрут, ну что погнали? Давайте походим по всем этим точками и первой точкой будет форум ```https://192.168.56.XX/forum/```, дальше            пойдем к почте ```https://192.168.56.XX/webmail/```, ну и на последок заглянем в БД ```https://192.168.56.XX/phpmyadmin/```.
     После того как мы походили по всем этим точкам, мы понимаем что нам также не хватает логинов и паролей, но у нас есть форум, где можно попробовать поискать      информацию. После серфинга по форуму удалось там обнаружить гения, который достаточно открыто показал свой пароль -> формулировка была следующей ```Failed        password for invalid user !q\]Ej?*5K5cy*AJ```, этим гением оказался ```lmezard```, давайте логиниться на форуме.
     
     Залогинились на форуме и давайте зайдем профиль, в профиле посмотрели информацию о пользователе и нашли там его электронный адрес почты.
     
     Теперь у нас есть почта, а это значит можно попробовать пойти посмотреть как дела у почты ```https://192.168.56.XX/webmail/```, как оказалось пароль остался      тем же ```!q\]Ej?*5K5cy*AJ```
     
     Залогинившись в почту внимание сразу падает на ```DB Access```, открываем и видим там логин и пароль ```root/Fg-'kKXBj87E:aJ$``` зная что у нас есть              ```phpMyAdmin``` сразу идем туда ```https://192.168.56.XX/phpmyadmin/```, на этот раз спросим как дела у БД.
     
   - Сейчас будут достаточно интересные моменты про ```SQL инъекции``` и что может быть если не защищаться от них.
     
     Одна из конечных целей взлома - это возможность получить оболочки для выполнения системных команд и владения целью или сетью . SQL-инъекция обычно связана        только с базами данных и их данными, но фактически может использоваться как вектор для получения командной оболочки. В качестве эксперимента мы будем            использовать простую ошибку SQL-инъекции для выполнения команд.
     
     В общем и целом как нам понять все это и что нужно делать?
     
     Нам нужно определить корневой каталог веб-сервера для загрузки нашей оболочки. В зависимости от приложения и типа используемого веб-сервера это может            варьироваться, особенно если администратор изменяет местоположение по умолчанию или имеются соответствующие разрешения. Для целей этой демонстрации мы            предположим, что корневой веб-каталог Apache по умолчанию ( / var / www / ) используется с общедоступными разрешениями на запись. Информацию о веб-сервере,      включая корневой каталог, обычно можно найти в файле «phpinfo.php», но в гашем случае мы просто выпустим на волю ```DirBuster```.
     
     Что такое ```DirBuster```?
     
     ```DirBuster``` — это многопотоковое Java приложение, предназначенное для брутфорса имён директорий и файлов веб-приложений и веб-серверов. DirBuster            пытается найти скрытые каталоги и файлы.

     Тем не менее, подобные инструменты в немалой степени ценны своими списками директорий и файлов. С этой программой поставляется несколько словарей, которые        были собраны из реальных названий файлов и директорий. Всего в DirBuster имеется 9 словарей, их описание будет дано ниже. Но если и этого мало, то DirBuster      умеет делать чистый брутфорс, от которого ничего не способно скрыться! Конечно, если у вас есть время 😉

     Домашняя страница: https://www.owasp.org/index.php/Category:OWASP_DirBuster_Project
     
     Что это и для чего это нужно? -> Запустили ```DirBuster``` и получили ответ.
     
     Вопрос: Есть две папки - ```templates_c``` и ```cache```, которые используются при использовании Smarty с кэшированием.
     
     Ответ: ```templates_c``` - используется для скомпилированных шаблонов, но без фактического содержимого, которое может быть динамически вставлено в них. С        другой стороны, папка cache используется для того, что вы знаете как кэшированные страницы - полные страницы, которые служили пользователю вместо того,          чтобы каждый раз заново компилировать их.
     
     Теперь мы понимаем что у нас есть ```templates_c``` - каталог, которому всегда нужно разрешение на запись.
     
     
