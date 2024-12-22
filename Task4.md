# BigDataCourse

## Название команды: Команда номер 37

Состав команды:
* Бушмелев Дмитрий
* Чичулин Мирослав

## Последовательность действий для установки и настройки Apache Spark:
### Развертывание PostgreSQL на DataNode

1. Подключимся к NameNode под пользователем `hadoop`:
   ```bash
   ssh team-37-nn
   ```

2. Установим и распакуем Apache Spark:
   ```bash
   wget https://dlcdn.apache.org/spark/spark-3.5.4/spark-3.5.4-bin-without-hadoop.tgz
   tar -zxvf spark-3.5.4-bin-without-hadoop.tgz
   ```
3. Зададим окружение:
   ```bash
   export SPARK_HOME=/home/hadoop/spark-3.5.4-bin-without-hadoop
   export PATH=$PATH:$SPARK_HOME/bin
   export SPARK_DIST_CLASSPATH=$SPARK_HOME/jars/*:$(hadoop classpath)
   ```
4. Перейдем в папку со Spark и запустим мастер-узел с помощью команды
   ```bash
   sbin/start-master.sh
   ```
