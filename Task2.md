# BigDataCourse

## Название команды: Команда номер 37

Состав команды:
* Бударин Павел
* Бушмелев Дмитрий
* Уландаев Владислав
* Чичулин Мирослав

Последовательность действий для разворачивания YARN:
* Под пользователем `hadoop` перейдите на `nameNode`
* С помощью команды `nano hadoop-3.4.0/etc/hadoop/mapred-site.xml` измените конфигурацию на
    ```
    <configuration>
        <property>
            <name>mapreduce.framework.name</name>
            <value>yarn</value>
        </property>
        <property>
            <name>mapreduce.application.classpath</name>
            <value>$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*</value>
        </property>
    </configuration>
    ```
* С помощью команды `nano hadoop-3.4.0/etc/hadoop/yarn-site.xml` задайте конфигурацию YARN на
    ```
    <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
        <property>
            <name>yarn.nodemanager.env-whitelist</name>
            <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOME
        </property>
    </configuration>
    ```
* Повторите предыдущие два шага на `dataNode`s
* Перейдите в папку hadoop `nameNode` с помощью команды `cd ~/hadoop-3.4.0/` и запустите YARN с помощью команды `sbin/start-yarn.sh`
* Запустите historyserver с помощью команды `mapred --daemon start historyserver`
* Вернитесь на `jumpNode` под пользователем `team` и скопируйте конфиг по умолчанию с помощью команды `sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/nn`
* Отредактируйте конфигурационный файл с помощью команды `sudo nano /etc/nginx/sites-available/nn` следующим образом:
  * В поле `server` измените первую строчку с `listen 80 default_server;` на  `listen 9870 default_server;`; закомментируйте строчку `listen [::]:80 default_server;`
  * В поле `location` закомментируйте строчку `try_files $uri $uri/ =404;` и добавьте новую строчку `proxy_pass http://team-37-nn:9870;`
* Создайте символическую ссылку на конфигурационный файл с помощью команды `sudo ln -sf /etc/nginx/sites-available/nn /etc/nginx/sites-enabled/nn`
* Перезагрузите конфигурацию с помощью команды `sudo systemctl reload nginx`
* Скопируйте конфигурационные файлы для YARN и historyserver с помощью команд `sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/ya` и `sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/dh`
* Отредактируйте конфигурационный файл веб-интерфейса YARN, открыв его с помощью команды `sudo nano /etc/nginx/sites-available/ya`, и заменив порт 9870 на 8088 для полей `server` и `location`
* Отредактируйте конфигурационный файл веб-интерфейса historyserver, открыв его с помощью команды `sudo nano /etc/nginx/sites-available/dh`, и заменив порт 9870 на 19888 для полей `server` и `location`
* Включите веб-хосты YARN и historyserver с помощью команд `sudo ln -sf /etc/nginx/sites-available/ya /etc/nginx/sites-enabled/ya` и `sudo ln -sf /etc/nginx/sites-available/dh /etc/nginx/sites-enabled/dh`
* Перезагрузите конфигурацию с помощью команды `sudo systemctl reload nginx`
* Установите `apache2-utils` с помощью команды `sudo apt install apache2-utils`
* На `jumpNode` перейдите в директорию `/etc` и создайте новую директорию с помощью команды `mkdir apache2`
* С помощью команды `sudo htpasswd -c /etc/apache2/.htpasswd team` создайте пользователя `team` и задайте к нему пароль (используйте тот же пароль, что и для доступа к кластеру)
* Для несанкционированного доступа к кластеру с помощью веб-интерфейсов YARN и historyserver для поднятых страниц в поне `server` добавьте
  ```
  auth_basic           "Web UI";
  auth_basic_user_file /etc/apache2/.htpasswd;
  ```
* Перезагрузите nginx с помощью команды `sudo systemctl reload nginx`
