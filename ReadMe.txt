История моего запуска приложения.

Исходный git репозиторий: https://github.com/OSBI/saiku
Ссылка на документацию: https://saiku-documentation.readthedocs.io/en/latest/
Как по мне он поддерживается очень плохо. Отвечают на ошибки, баги крайне медленно.

Запустить приложение было не так уж и просто.

0. т.к. в тестовой конфигурации используется много относительных путей, то для запуска на локальном томкат (а не встроенном: смотри назначение saiku-server) пришлось скопировать каталоги data, repository из saiku-server/target/dist/saiku-server в каталог выше используемого томката (в моём случает это C:\Program Files\Apache Software Foundation.
1. При сборке выяснилось, что в описании saiku-beans.xml у бина connectionManager дублируется свойство <property name="userService" ref="userServiceBean"/>. Пришлось удалить.
2. В классе SessionService, методе login удалил все проверки на наличие лицензии.
3. Запускал на Java 8, а приложение оказалось к этому не готово: RuntimeException - пришлось удалить подтягивание библиотеки xercesImpl при подключении других зависимостей. Сейчас парсер xml встроен в JDK, поэтому есть конфликт в runtime.
4. Удалил проект saiku-bi-platform-plugin-p7 - не увидел где используется.
5. Удалил из класса Licence обращение за лицензией и связанные с этим сеттеры описания в xml файле
6. В файле Setting.js удалил инициализацию модели var SettingsOverrideCollection = Backbone.Collection.extend т.к. обращается это дело по url info/ui-settings, а со стороны backend оброботчика нет.
7. Удалил проверку лицензии в файле Upgrade.js


Что удалось узнать по поводу того, как всё это работает.

Создаются различные соединения:
1. К тестовой БД h2 foodmart, earthquakes базе данных
2. К h2 бд с метаданными saiku
Конфигурации задаются как в web.xml, saiku-beans.properties, saiku-beans.xml
После создания БД она заполняется данными. Эта логика лежит в классе Database, метод init()


Назначение проектов:
saiku-server - используется при сбоке всего приложения - в его каталоге target/dist/saiku-server создаётся структуры (данные, БД, томкат) для запуска приложения через start-saiku.bat

Последующая история.
1. Удалил каталог .circleci (ссылка на проект https://circleci.com/). Предназначен проект для автоматической сбоки и публикации с GitHub, BitBucket
2. Удалил каталог saiku-bi-platform-plugin-p7.1. Считаю его лишним т.к. нигде не используется - только билдится. Видимо используется для подключения проекта к Pentaho.
3. Удалён каталог exporter. В нём был интересный скрипт по установлению сессии и получении данных:
https://curl.haxx.se/docs/manpage.html
Интересно, что утилите curl можно добавить залоговок: -H 'Content-Type: application/x-www-form-urlencoded'
Запостить данные --data 'language=en&username=admin&password=admin'
Записывает куки, после исполнения -c ./cookies.txt
Отправляет куки -b cookies.txt
-----------start script---------
#!/bin/bash
curl 'http://localhost:8080/saiku/rest/saiku/session' -H 'Content-Type: application/x-www-form-urlencoded' --data 'language=en&username=admin&password=admin' -c ./cookies.txt
curl 'http://localhost:8080/saiku/rest/saiku/admin/export/saiku/xls?file=/homes/home:admin/report1.saiku' -b cookies.txt  > export.xls
echo | mutt -a "./export.xls" -s "Scheduled Report Delivery" -- email@email.com
-----------end script---------
Попробовал подключиться аналогичный образом к другому проекту - всё получилось. Использовал curl внутри git
-----------start script---------
curl "http://10.4.134.157/login" -H "Content-Type: application/x-www-form-urlencoded" --data "language=en&username=Administrator@gromov.ru&password=Qwerty1" -c ./cookies.txt
curl "http://10.4.134.157/n_dashboard_space/68319361" -b cookies.txt
-----------end script---------
4. Удалил каталог saiku-legal, который содержал в себе файл с каким-то договором. По идее, проект распространяется по Apache License Version 2.0, так что никаких проблем быть не должно.
Перевод описания лицензии (http://www.dataved.ru/2011/03/apache-license-2.html)
5. Удалил saiku-repository т.к. не нашёл место использования. Каталог был весьма небольшим.
