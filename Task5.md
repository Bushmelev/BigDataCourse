# BigDataCourse

## Название команды: Команда номер 37

Состав команды:
* Бушмелев Дмитрий
* Чичулин Мирослав

# Развернем AirFlow для автоматического партицирования 

1. Установим необоходимые зависимости: 

```python
pip install apache-airflow 
pip install apache-airflow-providers-apache-spark airflow 
db init airflow users create -r Admin -u hadoop -f Hadoop -l Hadoopych -e m.chichulin@edu.hse.ru
```

2. Сделаем настройку прокси для просмотра Dags:


Выходит в sudo-ера: 

cd /etc/nginx/sites-available/ sudo cp dn0 af sudo ln -s /etc/nginx/sites-available/af /etc/nginx/sites-enabled/af sudo nano af

Пропишем туда:

```bash
server { listen 9765 default_server;

    server_name af;

    location / {
            # First attempt to serve request as file, then
            # as directory, then fall back to displaying a 404.
            proxy_pass http://team-37-jn:9764;
            auth_basic "Restricted Access";
            auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
```

3. Подправим необохдимые кофиги:
```bash
sudo -i -u hadoop
```

- в .profile пропишем AIRFLOW_HOME:
```bash
nano ~/.profile
```

ПРОПИШЕМ [export AIRFLOW_HOME=~/airflow]

source ~/.profile source venv/bin/activate


4. Запустим Веб-сервер и планировщик заданий

```bash
nohup airflow webserver -p 9764 > ~/airflow/logs/webserver.log 2>&1 & nohup airflow scheduler > ~/airflow/logs/scheduler.log 2>&1 &
```

5. Добавим в код со Spark элементы air flow: 
```bash
cd $AIRFLOW_HOME

dir=$(cat airflow.cfg | grep dags_folder | awk -F'=' '{print $2}' | xargs) mkdir -p "$dir" 

cd "$dir" touch hw4_script.py nano hw4_script.py
```

Добавим в код строки: 
```python
from airflow import DAG 
from airflow.operators.bash import BashOperator
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator 
from datetime import datetime, timedelta

default_args = { 'owner': 'airflow', 'depends_on_past': False, 'email_on_failure': False, 'email_on_retry': False, 'retries': 1, 'retry_delay': timedelta(minutes=10), }

with DAG( dag_id='housing_data', default_args=default_args, description='Process and partition housing data using Spark', schedule_interval=None, start_date=datetime(2024, 12, 20)) as dag:

download_data = BashOperator(
    task_id='download_data',
    bash_command='wget -O /tmp/california-housing.csv https://raw.githubusercontent.com/Bushmelev/BigDataCourse/refs/heads/main/california-housing.csv',
)

process_data = SparkSubmitOperator(
    task_id='process_data',
    application='/home/hadoop/hw4_script.py',
    conn_id='spark_default',
    application_args=['--input', '/tmp/housing.csv', '--output', '/user/hive/warehouse/c_housing'],
    conf={
        "spark.sql.warehouse.dir": "/user/hive/warehouse",
        "spark.hive.metastore.uris": "thrift://team-37-jn:9084",
        "spark.sql.hive.metastore.version": "3.1.2",
        "spark.sql.hive.metastore.jars": "main"
    }
)

verify_results = BashOperator(
    task_id='verify_results',
bash_command='beeline -u "jdbc:hive2://localhost:5433/default" -n hive -p hivepass -e "SHOW PARTITIONS hw_4_new_part.c_housing;"', )
```

6. Запустим Dag для теста:  


airflow dags trigger housing_data airflow dags backfill -s 2024-12-21 housing_data
