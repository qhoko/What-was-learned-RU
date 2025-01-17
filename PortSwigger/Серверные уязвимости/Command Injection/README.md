Инъекция команд ОС
В этом разделе мы объясняем, что такое инъекция команд ОС, и описываем, как уязвимости могут быть обнаружены и использованы. Мы также показываем вам некоторые полезные команды и методы для различных операционных систем и описываем, как предотвратить инъекцию команд ОС.

![image](https://github.com/user-attachments/assets/7bbaad94-aee0-4bf2-ab1e-22fbe67b709a)

Что такое внедрение команд ОС?
Инъекция команд ОС также известна как инъекция оболочки. Она позволяет злоумышленнику выполнять команды операционной системы (ОС) на сервере, на котором запущено приложение, и обычно полностью скомпрометировать приложение и его данные. Часто злоумышленник может использовать уязвимость инъекции команд ОС, чтобы скомпрометировать другие части инфраструктуры хостинга и использовать доверительные отношения, чтобы перенаправить атаку на другие системы в организации.

Внедрение команд ОС
В этом примере приложение для покупок позволяет пользователю просматривать, есть ли товар на складе в определенном магазине. Доступ к этой информации осуществляется через URL:

https://insecure-website.com/stockStatus?productID=381&storeID=29
Чтобы предоставить информацию о запасах, приложение должно запросить различные устаревшие системы. По историческим причинам функциональность реализована путем вызова команды оболочки с идентификаторами продукта и магазина в качестве аргументов:

stockreport.pl 381 29
Эта команда выводит информацию о состоянии запасов указанного товара, которая возвращается пользователю.

Приложение не реализует никакой защиты от внедрения команд ОС, поэтому злоумышленник может отправить следующие входные данные для выполнения произвольной команды:

& echo aiwefwlguh &
Если эти входные данные отправлены в productIDпараметре, команда, выполняемая приложением, будет следующей:

stockreport.pl & echo aiwefwlguh & 29
Команда echoвызывает отображение предоставленной строки в выходных данных. Это полезный способ проверки некоторых типов внедрения команд ОС. Символ &является разделителем команд оболочки. В этом примере он вызывает выполнение трех отдельных команд, одну за другой. Вывод, возвращаемый пользователю, следующий:

Error - productID was not provided
aiwefwlguh
29: command not found
Три строки вывода демонстрируют, что:

Исходная stockreport.plкоманда была выполнена без ожидаемых аргументов, поэтому было возвращено сообщение об ошибке.
Введенная echoкоманда была выполнена, и предоставленная строка была отображена в выходных данных.
Исходный аргумент 29был выполнен как команда, что вызвало ошибку.
Размещение дополнительного разделителя команд &после введенной команды полезно, поскольку он отделяет введенную команду от всего, что следует за точкой введения. Это снижает вероятность того, что то, что следует, помешает выполнению введенной команды.

олезные команды
После того, как вы определили уязвимость внедрения команд ОС, полезно выполнить некоторые начальные команды, чтобы получить информацию о системе. Ниже приведено краткое изложение некоторых команд, которые полезны на платформах Linux и Windows:

Цель команды	линукс	Окна
Имя текущего пользователя	whoami	whoami
Операционная система	uname -a	ver
Конфигурация сети	ifconfig	ipconfig /all
Сетевые соединения	netstat -an	netstat -an
Запуск процессов	ps -ef	tasklist
Уязвимости слепого внедрения команд ОС
Многие случаи внедрения команд ОС являются слепыми уязвимостями. Это означает, что приложение не возвращает вывод команды в своем HTTP-ответе. Слепые уязвимости все еще могут быть использованы, но для этого требуются другие методы.

В качестве примера представьте себе веб-сайт, который позволяет пользователям отправлять отзывы о сайте. Пользователь вводит свой адрес электронной почты и сообщение для отзыва. Затем серверное приложение генерирует электронное письмо администратору сайта, содержащее отзыв. Для этого оно вызывает программу mailс отправленными данными:

Способы внедрения команд ОС
Для проведения атак путем внедрения команд ОС можно использовать ряд метасимволов оболочки.

Ряд символов выполняют функцию разделителей команд, позволяя объединять команды в цепочку. Следующие разделители команд работают как в системах Windows, так и в системах Unix:

&
&&
|
||
Следующие разделители команд работают только в системах на базе Unix:

;
Новая строка ( 0x0aили \n)
В системах на базе Unix вы также можете использовать обратные кавычки или символ доллара для выполнения встроенной команды внутри исходной команды:

`
введенная команда`
$(
введенная команда)
Различные метасимволы оболочки имеют тонкое разное поведение, которое может изменить то, работают ли они в определенных ситуациях. Это может повлиять на то, позволяют ли они извлекать вывод команды по полосе пропускания или полезны только для слепой эксплуатации.

Иногда входные данные, которыми вы управляете, появляются в кавычках в исходной команде. В этой ситуации вам необходимо завершить цитируемый контекст (используя "или ') перед использованием подходящих метасимволов оболочки для внедрения новой команды.

Как предотвратить атаки с внедрением команд ОС
Самый эффективный способ предотвратить уязвимости внедрения команд ОС — никогда не вызывать команды ОС из кода уровня приложения. Почти во всех случаях существуют различные способы реализации требуемой функциональности с использованием более безопасных API платформы.

Если вам нужно вызывать команды ОС с пользовательским вводом, то вы должны выполнить строгую проверку ввода. Вот некоторые примеры эффективной проверки:

Проверка по белому списку разрешенных значений.
Проверка того, что введенные данные являются числом.
Проверка того, что входные данные содержат только буквенно-цифровые символы, без другого синтаксиса или пробелов.
Никогда не пытайтесь очистить ввод, экранируя метасимволы оболочки. На практике это слишком подвержено ошибкам и уязвимо для обхода опытным злоумышленником.

mail -s "This site is great" -aFrom:peter@normal-user.net feedback@vulnerable-website.com
Вывод команды mail(если таковой имеется) не возвращается в ответах приложения, поэтому использование echoполезной нагрузки не сработает. В этой ситуации вы можете использовать множество других методов для обнаружения и эксплуатации уязвимости.
