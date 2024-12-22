# BigDataCourse

## Название команды: Команда номер 37

Состав команды:
* Бушмелев Дмитрий
* Чичулин Мирослав

## Последовательность действий для установки и настройки Apache Spark:

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
5. Сделаем проксирование на `nameNode`:
   Под пользователем `team` на `jumpNode` сделайте копию файла с помощью команды
   ```bash
   sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/sp
   ```
   Отредактируйте конфигурационный, заменив номер порта на 8080. Создайте символическую ссылку с помощью команды
   `sudo ln -sf /etc/nginx/sites-available/sp /etc/nginx/sites-enabled/sp`
   
   Перезагрузите конфигурацию с помощью команды `sudo systemctl reload nginx`
6. Запустим worker. Для этого в папке со spark выполним
   ```bash
   mkdir worker
   sbin/start-worker.sh spark://team-37-nn:7077 -d /home/hadoop/spark-3.5.4-bin-without-hadoop/worker -h team-37-nn
   ```
7. Запустим интерактивный интерпретатор с помощью команды
   ```bash
   bin/spark-shell --master spark://team-37-nn:7077
   ```
