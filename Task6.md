# BigDataCourse

## Название команды: Команда номер 37

Состав команды:
* Бушмелев Дмитрий
* Чичулин Мирослав

## Последовательность действий для работы с данными в GreenPlum:

1. Подключитесь к ВМ, где развернут GreenPlum:
   ```bash
   ssh user@91.185.85.179
   ```
2. Загрузите файлы на ВМ с помощью команды
   ```bash
   scp .\companies.csv user@91.185.85.179:/home/user/team-37-data
   ```
3. На ВМ запустите ```psql -d idp``` и выполните
   ```bash
   CREATE EXTERNAL TABLE team_37 (
    company_number text,
    company_type text,
    office_address text,
    incorporation_date text,
    jurisdiction text,
    company_status text,
    account_type text,
    company_name text,
    sic_codes text,
    date_of_cessation text,
    next_accounts_overdue text,
    confirmation_statement_overdue text,
    owners text,
    officers text,
    average_number_employees_during_period text,
    current_assets text,
    last_accounts_period_end text,
    company_url text)
   LOCATION('gpfdist://localhost:8037/companies.csv')
   FORMAT 'CSV' (DELIMITER ';' HEADER);
   ```
4. Чтобы "раздать" таблицу выполните команду
   ```bash
   gpfdist -d /home/user/team-37-data -p 8037
   ```
5. Убедиться, что все работает можно с помощью команды
   ```bash
   SELECT * FROM team_37;
   ```
6. Создайте табличку в GreenPlum с помощью команды
   ```bash
   CREATE TABLE team_37_internal_table AS SELECT * FROM team_37;
   ```
