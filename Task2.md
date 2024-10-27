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
            <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_HOM>    </property>
    </configuration>
    ```
* Повторите предыдущие два шага на `dataNode`s
* Перейдите в папку hadoop с помощью команды `cd ~/hadoop-3.4.0/` и запустите YARN с помощью команды `sbin/start-yarn.sh`
* Запустите historyserver с помощью команды `mapred --daemon start historyserver`
* Вернитесь на `jumpNode` под пользователем `team` и скопируйте конфиг по умолчанию с помощью команды `sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/nn`
* Отредактируйте конфигурационный файл с помощью команды `sudo nano /etc/nginx/sites-available/nn` следующим образом:
  * В поле `server` измените первую строчку с `listen 80 default_server;` на  `listen 9870 default_server;`; закомментируйте строчку `listen [::]:80 default_server;`
  * В поле `location` закомментируйте строчку `try_files $uri $uri/ =404;` и добавьте новую строчку `proxy_pass http://team-37-nn:9870;`
