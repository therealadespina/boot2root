# boot2root

Задача: У нас есть дистрибутив BornToSecHackMe-v1.1.iso со стороны 42.fr в котором нужно получить права суперпользователя root.

1) Запустив дистрибутив мы можем обнаружить что на этот раз ip-адрес не высвечивается как это было в snow-crash, поэтому нужно будет его найти.

   - Зупускаем VirtualBox с уже установленной Kali, но прежде чем запустить дистрибутив BornToSecHackMe-v1.1.iso и Kali идем в Tools -> Network и создаем сетевую      карту. По умолчанию она будет называться ```vboxnet0```, в данном случае хост системой будет выступать наш iMac / macBook, посредиником будет выступать          VirtualBox, а гостевыми системами будут Kali и BornToSecHackMe-v1.1, после того как мы сделали сетевую карту мы идем в настройки наших машин ведь нам            нужно будет настроить им адаптер Host-only, что делает этот адаптер? Адаптер создает сеть между хост-системой и виртуальной машиной, минуя                        физическую сетевую карту. На компьютере появляется программный сетевой интерфейс, служащий для обмена данными между виртуальными машинами и хост-системой.        Виртуальные машины могут соединятся друг с другом и хост-системой, как будто соединены через коммутатор. Как и в режиме внутренней сети, виртуальной              машине не предоставляется физический интерфейс, благодаря чему машины не могут взаимодействовать с внешней сетью.
     В хост-системе появляется устройство VirtualBox Host-Only Network. Оно имеет собственную подсеть 192.168.56.0 и шлюз с адресом - 192.168.56.1. Устройство        соединяет подсеть и хост-систему без прямого выхода во внешнюю сеть.
     
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
     Итак что мы имеем? Мы видим открытые порты и понимаем что можем сделать, но пока что остается вопрос где взять учетные записи и пароли к ним? 
     Давайте сходим на 80 порт и посмотрим что у нас там есть? Вбиваем в строку поиска адрес ```192.168.X.XX:80```
     Далее мы попадаем на обычную интернет страничку на которой написано 'HACK ME' давайте попробуем поискать тут информацию, открыв панель разработчика в            надежде найти что-то интересное в html разметке я нашел ровно нихуя.
     Мы же помним что мы сидим с Kali? Ок, выпускайте Nikto.
     
     Что такое Nikto? Nikto – это сканер с открытым исходным кодом (GPL) для веб-серверов, он выполняет комплексные тесты в отношении серверов по нескольким          направлениям, включая более 6700 потенциально опасных файлов/программ, проверка на устаревшие версии более 1250 серверов и проблемы, специфичные для версий      более чем 270 серверов. Сканер также проверяет элементы конфигурации сервера, такие как присутствие нескольких индексных файлов, серверные опции HTTP и          пытается определить имя и версии веб-сервера и программного обеспечения.

     На официальном сайте изменения замерли на 2.1.5 версии аж в 2012 году. Тем не менее, под руководством автора проект живёт на GitHub’е, пользователи              регулярно добавляют в базу данных и плагины изменения для сканирования новых уязвимостей, новых версий и т.д.

     Nikto не создавался быть незаметным. Он будет тестировать веб-сервер за самое быстрое возможное время, очевидно, что его активность попадёт в логи веб-          сервера и в поле зрение IPS/IDS (систем обнаружения/предотвращения вторжений). Тем не менее, имеется поддержка для анти-IDS методов из LibWhisker – на            случай, если вы захотите их попробовать (или протестировать вашу систему IDS).

     Не каждая проверка относится к проблеме безопасности, хотя большинство относятся. Некоторые пункты являются проверками типа «только для информации», которые      ищут вещи, может быть не имеющие брешей безопасности, но веб-мастер или инженер по безопасности могут не знать, что это присутствует на сервере. Обычно в        выводимой информации эти элементы помечены соответствующим образом. Есть также некоторые проверки на неизвестные элементы, которые были замечены в файлах        журналов.
     
     Теперь я прорекламировал Nikto и мне за это не заплатили.
     
     Вбиваем: ```nikto -h https://192.168.56.XX```
     
     Что мы получаем после? Мы получаем пару каталогов которые могут быть нам интересны, а это ```forum```, ```webmail```, ```phpMyAdmin```
     
     
     
