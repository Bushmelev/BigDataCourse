# BigDataCourse

## Название команды: Команда номер 37

Состав команды:
* Бударин Павел
* Бушмелев Дмитрий
* Уландаев Владислав
* Чичулин Мирослав

Последовательность действий для разворачивания Hadoop:
* Авторизуйтесь на виртуальной машине с именем пользователя `team` и адресом `176.109.91.36`. Команда выглядит следующим образом: `team@176.109.91.36`
* Добавьте пользователя `hadoop` с помощью команды `sudo adduser hadoop`
* Откройте новую сессию оболочки для пользователя `hadoop` с помощью команды `sudo -i -u hadoop`
* Сгенерируйте ключи для jump-node с помощью команды `ssh-keygen`. Можно оставить ключ без пароля, нажав Enter
* Скачайте дистрибутив с помощью команды `wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz`
* Вернитесь в сессионное окно пользователя `team` с помощью команды `exit`
* С помощью команды `sudo nano /etc/hosts` отредактируйте файл `hosts`, предварительно удалив из него все данные и добавив туда следующие строки
  ```
  192.168.1.150    team-37-jn
  192.168.1.151    team-37-nn
  192.168.1.152    team-37-dn-0
  192.168.1.153    team-37-dn-1
  ```
* Пеереключитель на машину `nameNode` с помощью команды `ssh team-37-nn`
* Отредактируйте файл `hosts`, добавив те же данные, что и шагом выше
* Создайте пользователя с помощью команды `sudo adduser hadoop` и переключитесь на него с помощью `sudo -i -u hadoop`
* Сгенерируйте ключи с помощью команды `ssh-keygen`
* Повторите шаги по редактированию файла `hosts`, созданию нового пользователя и генерацию ключей для двух оставшихся `dataNode`'s
* Вернитесь на `jumpNode`, переключитесь на пользователя `hadoop` с помощью команды `sudo -i -u hadoop`. Добавьте публичные ключи от всех нод в файл `authorized_keys` с помощью команды `nano .ssh/authorized_keys`
* Повторите предыдущий шаг для всех остальных нод
* С помощью команды `scp hadoop-3.4.0.tar.gz team-37-nn:/home/hadoop` скопируйте дистрибутив на все остальные виртуальные машины
* С  помощью команды `tar -xvzf hadoop-3.4.0.tar.gz` распакуйте дистрибутив на всех машинах
* Перейтите на `nameNode` и в файле `.profile` с помощью команды `nano ~/.profile` задайте следующие переменные
  ```
  export HADOOP_HOME=/home/hadoop/hadoop-3.4.0
  export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
  export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
  ```
  Повторите данное действие на всех `dataNode`, либо же скопируйте с помощью команды `scp ~/.profile team-37-dn-0:/home/hadoop`
* Добавьте строку `JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64` в файл с помощью команды `nano hadoop-3.4.0/etc/hadoop/hadoop-env.sh`. Повторите данное действие и на остальных `dataNode`.
* С помощью команды `nano hadoop-3.4.0/etc/hadoop/core-site.xml` задайте следующую конфигурацию
  ```
  <configuration>
      <property>
          <name>fs.defaultFS</name>
          <value>hdfs://team-37-nn:9000</value>
      </property>
  </configuration>
  ```
* С помощью команды `nano hadoop-3.4.0/etc/hadoop/hdfs-site.xml` задайте следующую конфигурацию
  ```
  <configuration>
      <property>
          <name>dfs.replication</name>
        <value>3</value>
      </property>
  </configuration>
  ```
* С помощью команды `nano hadoop-3.4.0/etc/hadoop/workers` установите в файле значения
  ```
  team-37-nn
  team-37-dn-00
  team-37-dn-01
  ```
