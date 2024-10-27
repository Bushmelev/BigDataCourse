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
* 
