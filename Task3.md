# BigDataCourse

## Название команды: Команда номер 37

Состав команды:
* Бударин Павел*
* Бушмелев Дмитрий
* Уландаев Владислав*
* Чичулин Мирослав

## Последовательность действий для установки и настройки Hive Hadoop:
### Развертывание PostgreSQL на DataNode

1. Подключимся к NameNode:
   ```bash
   ssh team-37-nn
   ```

2. Установим PostgreSQL:
   ```bash
   sudo apt install postgresql
   ```

3. Создадим и настроим базу данных для Hive:
   ```bash
   sudo -i -u postgres
   psql
   CREATE USER hive WITH PASSWORD 'hiveMegaPass';
   CREATE DATABASE metastore WITH OWNER hive;
   ALTER DEFAULT PRIVILEGES IN SCHEMA public GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO hive;
   GRANT ALL PRIVILEGES ON DATABASE "metastore" TO hive;
   ```

4. Настроем сетевой доступ для базы данных:
   - Открываем файл `/etc/postgresql/16/main/postgresql.conf` и настраиваем:
     ```plaintext
     listen_addresses = 'team-37-dn-00'
     ```

   - Откроем файл `/etc/postgresql/16/main/pg_hba.conf` и пропишем наш IP:
     ```plaintext
     host metastore hive 192.168.1.151/32 password
     ```

5. Перезапустим PostgreSQL:
   ```bash
   sudo systemctl restart postgresql
   ```

### Установка и настройка Hive на NameNode

1. Подключимся к JumpNode:
   ```bash
   ssh team-37-jn
   ```

2. Установим клиент PostgreSQL:
   ```bash
   sudo apt install postgresql-client-16
   psql -h team-37-nn -p 5432 -U hive -W -d metastore
   ```

3. Переключимся на пользователя hadoop и установим Hive:
   ```bash
   sudo -i -u hadoop
   wget https://dlcdn.apache.org/hive/hive-4.0.1/apache-hive-4.0.1-bin.tar.gz
   tar -xvzf apache-hive-4.0.1-bin.tar.gz
   ```

4. Устанавим драйвер PostgreSQL:
   ```bash
   cd apache-hive-4.0.1-bin/lib
   wget https://jdbc.postgresql.org/download/postgresql-42.7.4.jar
   ```

5. Создадим файл `hive-site.xml`:
   ```xml
   <configuration>
     <property>
       <name>hive.server2.authentication</name>
       <value>NONE</value>
     </property>
     <property>
       <name>hive.server2.thrift.bind.host</name>
       <value>0.0.0.0</value>
     </property>
     <property>
       <name>hive.server2.thrift.port</name>
       <value>5433</value>
     </property>
     <property>
       <name>hive.metastore.warehouse.dir</name>
       <value>/user/hive/warehouse</value>
     </property>
     <property>
       <name>javax.jdo.option.ConnectionUserName</name>
       <value>hive</value>
     </property>
     <property>
       <name>javax.jdo.option.ConnectionPassword</name>
       <value>hiveMegaPass</value>
     </property>
     <property>
       <name>javax.jdo.option.ConnectionURL</name>
       <value>jdbc:postgresql://team-37-dn-00:5432/metastore</value>
     </property>
     <property>
       <name>javax.jdo.option.ConnectionDriverName</name>
       <value>org.postgresql.Driver</value>
     </property>
   </configuration>
   ```


6. Насторем переменные среды:
   ```bash
   export HIVE_HOME=/home/hadoop/apache-hive-4.0.1-bin
   export HIVE_CONF_DIR=$HIVE_HOME/conf
   export HIVE_AUX_JARS_PATH=$HIVE_HOME/lib/*
   export PATH=$PATH:$HIVE_HOME/bin
   ```

### Инициализация и запуск Hive

1. Создадим каталоги в HDFS:
   ```bash
   #Т.к. у нас hdfs на NameNode
   hdfs dfs -mkdir -p hdfs://team-37-nn:9000/user/hive/warehouse
   hdfs dfs -chmod g+w /tmp
   hdfs dfs -chmod g+w /hdfs://team-37-nn:9000/user/hive/warehouse
   ```

2. Инициализируем базу данных для Hive:
   ```bash
   cd $HIVE_HOME
   ./bin/schematool -dbType postgres -initSchema
   ```

3. Запустим сервис HiveServer:
   ```bash
   hive --hiveconf hive.server2.enable.doAs=false \
        --hiveconf hive.security.authorization.enabled=false \
        --hiveconf hive.server2.thrift.bind.host=0.0.0.0 \
        --hiveconf hive.server2.thrift.port=5433 \
        --service hiveserver2 1>> /tmp/hs2.log 2>> /tmp/hs2.log &
   ```

4. Подключаемся к Hive через консоль и создаем базу данных:
   ```bash
   beeline -u jdbc:hive2://team-37-jn:5433
   CREATE DATABASE test;
   ```

5. Загрузим данные:

   - Сделаем директорию для данных и загрузим их:
     ```bash
     hdfs dfs -mkdir /input
     hdfs dfs -put companies.csv /input
     ```

   - Подключимся к базе данных Hive и создадим таблицу:
     ```bash
     beeline -u jdbc:hive2://team-37-jn:5433

      CREATE TABLE IF NOT EXISTS test.company(
        company_number string,
        company_type string,
        office_address string,
        incorporation_date string,
        jurisdiction string,
        company_status string,
        account_type string,
        company_name string,
        sic_codes string,
        date_of_cessation string,
        next_accounts_overdue string,
        confirmation_statement_overdue string,
        owners string,
        officers string,
        average_number_employees_during_period string,
        current_assets string,
        last_accounts_period_end string,
        company_url string)
        ROW FORMAT DELIMITED FIELDS TERMINATED BY '|';
     ```
     - Загрузим данные в таблицу:
     ```bash
      LOAD DATA INPATH '/input/companies.csv' INTO TABEL test.company;
     ```

