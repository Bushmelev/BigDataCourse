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
6. Установим необходимые пакеты python
   ```bash
   python3 -m venv venv
   source venv/bin/activate
   pip install pySpark onetl
   ```
7. Если отсутвуют файлы для работы, необохимо их установить
```bash
   wget https://raw.githubusercontent.com/ADBondarenko/BigDataProjectOPS/refs/heads/main/housing.csv](https://raw.githubusercontent.com/Bushmelev/BigDataCourse/refs/heads/main/california-housing.csv
   hdfs dfs -put california-housing.csv /input
   ```
8. Создадим файл hw4_script.py со следущим содержанием:
```python
from pyspark.sql import SparkSession
from onetl.connection import SparkHDFS
from onetl.file import FileDFReader
from onetl.file.format import CSV
from pyspark.sql.functions import udf, avg, count, log, round, col
from pyspark.sql.types import StringType, DoubleType
import logging
import sys


logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s %(message)s',
    handlers=[
        logging.StreamHandler(sys.stdout)
    ]
)
logger = logging.getLogger(__name__)


spark = (
    SparkSession.builder
    .master("yarn")
    .appName("spark-with-yarn")
    .config("spark.sql.warehouse.dir", "/user/hive/warehouse")
    .config("spark.hive.metastore.uris", "thrift://team-37-nn:9084")
    .config("spark.sql.hive.metastore.version", "3.1.2")
    .config("spark.sql.hive.metastore.jars", "maven")
    .enableHiveSupport()
    .getOrCreate()
)

hdfs = SparkHDFS(host="team-37-nn", port=9000, spark=spark, cluster="test")
logger.info(hdfs.check())
logger.info(spark)

reader = FileDFReader(
    connection=hdfs,
    format=CSV(delimiter=",", header=True),
    source_path="/input"
)

df = reader.run(["california-housing.csv"])

df.printSchema()


df = df.withColumn("median_income", df["median_income"].cast(DoubleType()))

def income_category(income):
    if income < 3:
        return "low"
    elif income < 5:
        return "medium"
    elif income < 7:
        return "high"
    else:
        return "very_high"

income_category_udf = udf(income_category, StringType())

df_new = df.withColumn("income_category", income_category_udf(col("median_income")))

agg_df = df_new.groupBy("income_category").agg(
    avg("median_house_value").alias("avg_house_value"),
    avg("total_rooms").alias("avg_rooms"),
    avg("population").alias("avg_population"),
    count("*").alias("count_records")
)

agg_df = agg_df.withColumn("log_avg_house_value", log(col("avg_house_value")))

agg_df = agg_df.withColumn("rounded_avg_rooms", round(col("avg_rooms"), 2))

spark.sql("CREATE DATABASE IF NOT EXISTS hw_4_new_part")

agg_df.write \
    .format("parquet") \
    .mode("overwrite") \
    .partitionBy("income_category") \
    .saveAsTable("hw_4_new_part.c_housing")
   ```
9. Запустим скрипт и убедимся, что все успешно записалось
```bash
   python hw4_script.py
   hdfs dfs -ls /user/hive/warehouse/hw_4_new_part.db/c_housing
   ```
