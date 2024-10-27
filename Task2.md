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
* 
