# 30 задач по Pandas и SQL для собеседования

## Исходные данные

```python
import pandas as pd
import numpy as np

# Основная таблица с сотрудниками
df = pd.DataFrame({
    'emp_id': [101, 102, 103, 104, 105, 106, 107, 108],
    'name': ['Анна', 'Борис', 'Виктор', 'Галина', 'Дмитрий', 'Елена', 'Жанна', 'Захар'],
    'dept': ['IT', 'Sales', 'IT', 'HR', 'Sales', 'IT', 'HR', 'Sales'],
    'salary': [60000, 75000, 80000, 70000, 65000, 82000, 72000, 68000],
    'age': [25, 30, 35, 42, 28, 31, 29, 33],
    'hours': [120, 80, 150, 90, 60, 130, 110, 95],
    'city': ['Москва', 'СПб', 'Москва', 'Казань', 'СПб', 'Москва', 'Казань', 'СПб']
})

# Вторая таблица для merge
bonuses = pd.DataFrame({
    'dept': ['IT', 'Sales', 'HR', 'Marketing'],
    'bonus_percent': [0.2, 0.15, 0.1, 0.25]
})

# Третья таблица для merge
projects = pd.DataFrame({
    'emp_id': [101, 102, 103, 104, 105, 106, 107, 101, 103],
    'project': ['P1', 'P2', 'P3', 'P1', 'P2', 'P3', 'P4', 'P4', 'P1'],
    'project_hours': [120, 80, 150, 90, 60, 130, 110, 70, 140]
})
```

---

## 1. Найдите сотрудников с зарплатой выше 70000

**Pandas:**
```python
df[df['salary'] > 70000]
```

**SQL:**
```sql
SELECT * FROM employees 
WHERE salary > 70000;
```

---

## 2. Добавьте колонку с зарплатой в тысячах

**Pandas:**
```python
df['salary_k'] = df['salary'] / 1000
```

**SQL:**
```sql
SELECT *, salary / 1000 AS salary_k 
FROM employees;
```

---

## 3. Отсортируйте сотрудников по возрасту (от младших к старшим)

**Pandas:**
```python
df.sort_values('age')
```

**SQL:**
```sql
SELECT * FROM employees 
ORDER BY age;
```

---

## 4. Найдите среднюю зарплату по отделам

**Pandas:**
```python
df.groupby('dept')['salary'].mean()
```

**SQL:**
```sql
SELECT dept, AVG(salary) AS avg_salary
FROM employees
GROUP BY dept;
```

---

## 5. Посчитайте количество сотрудников в каждом городе

**Pandas:**
```python
df.groupby('city')['name'].count()
```

**SQL:**
```sql
SELECT city, COUNT(*) AS employee_count
FROM employees
GROUP BY city;
```

---

## 6. Категоризируйте сотрудников по возрасту: до 30 лет — 'young', 30 и старше — 'senior'

**Pandas:**
```python
df['age_group'] = df['age'].apply(lambda x: 'young' if x < 30 else 'senior')
```

**SQL:**
```sql
SELECT *, 
       CASE WHEN age < 30 THEN 'young' ELSE 'senior' END AS age_group
FROM employees;
```

---

## 7. Добавьте колонку со средней зарплатой по отделу (для каждой строки)

**Pandas:**
```python
df['dept_avg_salary'] = df.groupby('dept')['salary'].transform('mean')
```

**SQL (оконная функция):**
```sql
SELECT *, 
       AVG(salary) OVER (PARTITION BY dept) AS dept_avg_salary
FROM employees;
```

---

## 8. Оставьте только те отделы, где средняя зарплата выше 70000

**Pandas:**
```python
df.groupby('dept').filter(lambda x: x['salary'].mean() > 70000)
```

**SQL:**
```sql
SELECT * FROM employees
WHERE dept IN (
    SELECT dept FROM employees
    GROUP BY dept
    HAVING AVG(salary) > 70000
);
```

---

## 9. Добавьте к сотрудникам информацию о бонусе из таблицы bonuses

**Pandas:**
```python
df.merge(bonuses, on='dept', how='left')
```

**SQL:**
```sql
SELECT e.*, b.bonus_percent
FROM employees e
LEFT JOIN bonuses b ON e.dept = b.dept;
```

---

## 10. Найдите сотрудников из Москвы с зарплатой выше 70000

**Pandas:**
```python
df[(df['city'] == 'Москва') & (df['salary'] > 70000)]
```

**SQL:**
```sql
SELECT * FROM employees
WHERE city = 'Москва' AND salary > 70000;
```

---

## 11. Добавьте колонку с отклонением зарплаты от средней по отделу

**Pandas:**
```python
df['salary_diff'] = df['salary'] - df.groupby('dept')['salary'].transform('mean')
```

**SQL (оконная функция):**
```sql
SELECT *, 
       salary - AVG(salary) OVER (PARTITION BY dept) AS salary_diff
FROM employees;
```

---

## 12. Категоризируйте зарплату: 'high' (>70000), 'medium' (60000-70000), 'low' (<60000)

**Pandas:**
```python
df['salary_level'] = df['salary'].apply(
    lambda x: 'high' if x > 70000 else ('medium' if x >= 60000 else 'low')
)
```

**SQL:**
```sql
SELECT *, 
       CASE 
           WHEN salary > 70000 THEN 'high'
           WHEN salary >= 60000 THEN 'medium'
           ELSE 'low'
       END AS salary_level
FROM employees;
```

---

## 13. Для каждого отдела покажите среднюю зарплату и процент бонуса

**Pandas:**
```python
merged = df.merge(bonuses, on='dept')
merged.groupby('dept').agg(
    avg_salary=('salary', 'mean'),
    bonus_percent=('bonus_percent', 'first')
)
```

**SQL:**
```sql
SELECT e.dept, 
       AVG(e.salary) AS avg_salary,
       MAX(b.bonus_percent) AS bonus_percent
FROM employees e
LEFT JOIN bonuses b ON e.dept = b.dept
GROUP BY e.dept;
```

---

## 14. Оставьте только те отделы, где есть сотрудники младше 30 лет

**Pandas:**
```python
df.groupby('dept').filter(lambda x: (x['age'] < 30).any())
```

**SQL:**
```sql
SELECT * FROM employees
WHERE dept IN (
    SELECT DISTINCT dept FROM employees
    WHERE age < 30
);
```

---

## 15. Добавьте колонку с процентом, который зарплата составляет от суммы по отделу

**Pandas:**
```python
df['dept_salary_pct'] = df['salary'] / df.groupby('dept')['salary'].transform('sum') * 100
```

**SQL (оконная функция):**
```sql
SELECT *, 
       salary / SUM(salary) OVER (PARTITION BY dept) * 100 AS dept_salary_pct
FROM employees;
```

---

## 16. Посчитайте общую сумму зарплат по компании

**Pandas:**
```python
df['salary'].sum()
```

**SQL:**
```sql
SELECT SUM(salary) AS total_salary FROM employees;
```

---

## 17. Найдите сотрудников, которые не участвуют в проектах

**Pandas:**
```python
emp_with_projects = df.merge(projects[['emp_id']].drop_duplicates(), on='emp_id', how='left', indicator=True)
emp_with_projects[emp_with_projects['_merge'] == 'left_only']
```

**SQL:**
```sql
SELECT e.* FROM employees e
LEFT JOIN projects p ON e.emp_id = p.emp_id
WHERE p.emp_id IS NULL;
```

---

## 18. Для каждого отдела найдите сотрудника с максимальной зарплатой

**Pandas:**
```python
df.groupby('dept').apply(lambda x: x.nlargest(1, 'salary')).reset_index(drop=True)
```

**SQL (оконная функция):**
```sql
WITH ranked AS (
    SELECT *, 
           ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT * FROM ranked WHERE rn = 1;
```

---

## 19. Добавьте колонку с рангом сотрудника по зарплате внутри отдела (1 — самая высокая)

**Pandas:**
```python
df['dept_salary_rank'] = df.groupby('dept')['salary'].rank(ascending=False, method='dense')
```

**SQL (оконная функция):**
```sql
SELECT *, 
       RANK() OVER (PARTITION BY dept ORDER BY salary DESC) AS dept_salary_rank
FROM employees;
```

---

## 20. Переименуйте колонку 'hours' в 'work_hours'

**Pandas:**
```python
df.rename(columns={'hours': 'work_hours'}, inplace=True)
```

**SQL:**
```sql
-- В PostgreSQL:
ALTER TABLE employees RENAME COLUMN hours TO work_hours;

-- Для запроса:
SELECT emp_id, name, dept, salary, age, hours AS work_hours, city FROM employees;
```

---

## 21. Для каждого проекта найдите средний возраст участников

**Pandas:**
```python
projects.merge(df[['emp_id', 'age']], on='emp_id').groupby('project')['age'].mean()
```

**SQL:**
```sql
SELECT p.project, AVG(e.age) AS avg_age
FROM projects p
JOIN employees e ON p.emp_id = e.emp_id
GROUP BY p.project;
```

---

## 22. Оставьте только сотрудников из отделов, где больше 2 человек

**Pandas:**
```python
df.groupby('dept').filter(lambda x: len(x) > 2)
```

**SQL:**
```sql
SELECT * FROM employees
WHERE dept IN (
    SELECT dept FROM employees
    GROUP BY dept
    HAVING COUNT(*) > 2
);
```

---

## 23. Создайте колонку 'city_code' — первые 3 буквы города в верхнем регистре

**Pandas:**
```python
df['city_code'] = df['city'].apply(lambda x: x[:3].upper())
```

**SQL:**
```sql
SELECT *, 
       UPPER(LEFT(city, 3)) AS city_code
FROM employees;
```

---

## 24. Добавьте колонку с медианной зарплатой по отделу

**Pandas:**
```python
df['dept_median_salary'] = df.groupby('dept')['salary'].transform('median')
```

**SQL (с оконной функцией PERCENTILE_CONT):**
```sql
SELECT *, 
       PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) 
       OVER (PARTITION BY dept) AS dept_median_salary
FROM employees;
```

---

## 25. Добавьте к сотрудникам общее количество часов с проектов

**Pandas:**
```python
hours_sum = projects.groupby('emp_id')['project_hours'].sum().reset_index()
df.merge(hours_sum, on='emp_id', how='left', suffixes=('', '_total'))
```

**SQL:**
```sql
SELECT e.*, SUM(p.project_hours) AS total_hours
FROM employees e
LEFT JOIN projects p ON e.emp_id = p.emp_id
GROUP BY e.emp_id, e.name, e.dept, e.salary, e.age, e.hours, e.city;
```

---

## 26. Для каждого города найдите максимальную зарплату

**Pandas:**
```python
df.groupby('city')['salary'].max()
```

**SQL:**
```sql
SELECT city, MAX(salary) AS max_salary
FROM employees
GROUP BY city;
```

---

## 27. Оставьте только те отделы, где суммарное количество часов больше 200

**Pandas:**
```python
df.groupby('dept').filter(lambda x: x['hours'].sum() > 200)
```

**SQL:**
```sql
SELECT * FROM employees
WHERE dept IN (
    SELECT dept FROM employees
    GROUP BY dept
    HAVING SUM(hours) > 200
);
```

---

## 28. Удалите дубликаты по имени (если бы они были)

**Pandas:**
```python
df.drop_duplicates(subset='name')
```

**SQL:**
```sql
-- Удаление дубликатов:
DELETE FROM employees
WHERE emp_id NOT IN (
    SELECT MIN(emp_id)
    FROM employees
    GROUP BY name
);

-- Просто уникальные значения:
SELECT DISTINCT ON (name) * FROM employees;
```

---

## 29. Добавьте колонку с бонусом в деньгах (salary * bonus_percent)

**Pandas:**
```python
merged = df.merge(bonuses, on='dept', how='left')
merged['bonus_amount'] = merged['salary'] * merged['bonus_percent'].fillna(0)
```

**SQL:**
```sql
SELECT e.*, 
       COALESCE(b.bonus_percent, 0) AS bonus_percent,
       e.salary * COALESCE(b.bonus_percent, 0) AS bonus_amount
FROM employees e
LEFT JOIN bonuses b ON e.dept = b.dept;
```

---

## 30. Оставьте только тех сотрудников, чья зарплата выше средней по их отделу

**Pandas:**
```python
df[df['salary'] > df.groupby('dept')['salary'].transform('mean')]
```

**SQL (оконная функция):**
```sql
WITH dept_avg AS (
    SELECT *, 
           AVG(salary) OVER (PARTITION BY dept) AS dept_avg_salary
    FROM employees
)
SELECT * FROM dept_avg
WHERE salary > dept_avg_salary;
```