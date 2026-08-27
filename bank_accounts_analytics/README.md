# Automated Bank Account Analytics (ABAA)


**Automated Bank Account Analytics (ABAA)** — это комплекс аналитических SQL-скриптов промышленного уровня, разработанный для глубокого аудита, мониторинга портфеля и выявления операционных рисков в банковских базах данных. Система автоматически сегментирует клиентскую базу, оценивает KPI менеджеров и выявляет проблемные кредитные счета.

---

## 🚀 Ключевые возможности

*   🎯 **Targeting & VIP-сегментация** — автоматический поиск наиболее прибыльных активных клиентов на основе скоринга баланса.
*   📊 **Анализ регионального покрытия** — расчет емкости клиентской базы, общего и среднего объема привлеченных средств в разрезе городов.
*   🔍 **Аудит спящих активов** — детекция неактивных счетов и классификация причин отсутствия транзакционной активности.
*   📈 **Оценка эффективности (KPI)** — расчет консолидированных метрик удержания клиентов и объемов портфелей для персональных менеджеров.
*   ⚠️ **Управление рисками (Risk Management)** — автоматическая система трекинга закредитованности и многоуровневого анализа рисков дефолта.

## 🛠️ Стек технологий

*   **СУБД:** PostgreSQL / MySQL / SQLite
*   **Язык запросов:** ANSI SQL (с использованием оконных функций, условной агрегации и подзапросов)
*   **Инструменты разработки:** DBeaver / DataGrip / Jupyter Notebook

## 📋 Структура аналитических модулей

Решения разбиты на изолированные модули, готовые к интеграции в BI-системы (например, Tableau или PowerBI):

### 1. Модуль VIP-маркетинга
Выгрузка активных клиентов с балансом, превышающим средний рыночный показатель по банку, для таргетированных предложений.
```sql
SELECT client_name, city, account_type, balance, currency, opened_at
FROM bank_accounts
WHERE status = 'active' AND city IS NOT NULL 
  AND balance > (SELECT AVG(balance) FROM bank_accounts WHERE status = 'active')
ORDER BY balance DESC;
```

### 2. Географическая аналитика (Регионы)
Оценка плотности покрытия рынка (для городов, имеющих от 3-х открытых счетов).
```sql
SELECT city, COUNT(*) AS number_of_accounts, SUM(balance) AS total_balance,
  ROUND(AVG(balance), 2) AS avg_balance,
  COUNT(CASE WHEN status = 'active' THEN 1 END) AS number_of_active_accounts
FROM bank_accounts WHERE city IS NOT NULL
GROUP BY city HAVING COUNT(*) >= 3 ORDER BY number_of_accounts DESC;
```

### 3. Мониторинг оттока (Churn Rate)
Выявление скрытого оттока пользователей на основе даты последней транзакции.
```sql
SELECT *, CASE WHEN last_transaction IS NULL THEN 'Нет операций' ELSE 'Операция была давно' END AS reason
FROM bank_accounts
WHERE (last_transaction IS NULL OR last_transaction < '2024-03-01') AND status != 'closed';
```

### 4. Контроль эффективности менеджеров
Агрегация метрик удержания и ведения баланса для выявления перегруженных или неэффективных сотрудников.
```sql
SELECT COALESCE(manager_name, 'Без менеджера') AS manager_name, COUNT(*) AS number_of_accounts,
  SUM(balance) AS total_balance, ROUND(AVG(balance), 2) AS avg_balance
FROM bank_accounts GROUP BY COALESCE(manager_name, 'Без менеджера');
```

### 5. Скоринг рисков (Risk Scoring)
Выявление критических кредитных позиций с использованием лимита более чем на 60% и ранжированием уровней угрозы дефолта.
```sql
SELECT account_id, client_name, balance, credit_limit,
  ROUND((ABS(balance) / credit_limit) * 100, 2) AS usage_limit_percent,
  CASE WHEN (ABS(balance) / credit_limit) * 100 > 90 THEN 'Высокий'
       WHEN (ABS(balance) / credit_limit) * 100 >= 75 THEN 'Средний' ELSE 'Умеренный' END AS risk_level
FROM bank_accounts
WHERE account_type = 'credit' AND balance < 0 AND ABS(balance) > 0.6 * credit_limit;
```

## 📦 Быстрый старт

### Развертывание структуры данных
Для тестирования аналитического ядра разверните схему таблицы `bank_accounts` со следующими ключевыми полями:
*   `account_id` (Primary Key) — уникальный идентификатор счета.
*   `client_name`, `city` — персональные данные клиента.
*   `balance`, `credit_limit`, `currency` — финансовые метрики аккаунта.
*   `status` (`active`, `blocked`, `closed`) — операционный статус счета.
*   `manager_name` — ответственный сотрудник.

### Запуск тестов
Скопируйте любой скрипт из папки репозитория и выполните его в консоли вашей СУБД для получения витрины данных. 

## 🤝 Вклад в развитие (Contributing)

Мы открыты для интеграции новых BI-метрик и оптимизации планов выполнения запросов (EXPLAIN). 

1. Сделайте Fork проекта.
2. Создайте ветку фичи (`git checkout -b feature/NewAnalyticalMetric`).
3. Зафиксируйте коммит (`git commit -m 'Add support for YoY metric'`).
4. Откройте Pull Request.

## 📄 Лицензия

Этот проект распространяется под лицензией MIT. Подробнее см. в файле [LICENSE](LICENSE).
