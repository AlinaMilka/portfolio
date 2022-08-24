1.
Посчитайте, сколько компаний закрылось.


```python
SELECT COUNT(status)
FROM company
WHERE status = 'closed';
```

2.
Отобразите количество привлечённых средств для новостных компаний США. Используйте данные из таблицы company. Отсортируйте таблицу по убыванию значений в поле funding_total .


```python
SELECT SUM(funding_total)
FROM company
WHERE country_code = 'USA'
AND category_code = 'news'
GROUP BY funding_total, id
ORDER BY funding_total DESC;
```

3.
Найдите общую сумму сделок по покупке одних компаний другими в долларах. Отберите сделки, которые осуществлялись только за наличные с 2011 по 2013 год включительно.


```python
SELECT SUM(price_amount)
FROM acquisition
WHERE term_code = 'cash'
AND EXTRACT(YEAR FROM CAST(acquired_at AS DATE)) BETWEEN 2011 AND 2013;
```

4.
Отобразите имя, фамилию и названия аккаунтов людей в твиттере, у которых названия аккаунтов начинаются на 'Silver'.


```python
SELECT first_name,
       last_name,
       twitter_username
FROM people
WHERE twitter_username LIKE 'Silver%';
```

5.
Выведите на экран всю информацию о людях, у которых названия аккаунтов в твиттере содержат подстроку 'money', а фамилия начинается на 'K'.


```python
SELECT *
FROM people
WHERE twitter_username LIKE '%money%'
AND last_name LIKE 'K%';
```

6.
Для каждой страны отобразите общую сумму привлечённых инвестиций, которые получили компании, зарегистрированные в этой стране. Страну, в которой зарегистрирована компания, можно определить по коду страны. Отсортируйте данные по убыванию суммы.


```python
SELECT country_code,
       SUM(funding_total)
FROM company
GROUP BY country_code
ORDER BY SUM(funding_total) DESC;
```

7.
Составьте таблицу, в которую войдёт дата проведения раунда, а также минимальное и максимальное значения суммы инвестиций, привлечённых в эту дату.
Оставьте в итоговой таблице только те записи, в которых минимальное значение суммы инвестиций не равно нулю и не равно максимальному значению.


```python
SELECT CAST(funded_at AS DATE) AS dat,
       MIN(raised_amount),
       MAX(raised_amount)
FROM funding_round
GROUP BY dat
HAVING MIN(raised_amount) <> 0 AND MIN(raised_amount) <> MAX(raised_amount);
```

8.
Создайте поле с категориями:
Для фондов, которые инвестируют в 100 и более компаний, назначьте категорию high_activity.
Для фондов, которые инвестируют в 20 и более компаний до 100, назначьте категорию middle_activity.
Если количество инвестируемых компаний фонда не достигает 20, назначьте категорию low_activity.
Отобразите все поля таблицы fund и новое поле с категориями.


```python
SELECT *,
       CASE
           WHEN invested_companies < 20 THEN 'low_activity'
           WHEN invested_companies >= 20 AND invested_companies < 100 THEN 'middle_activity'
           WHEN invested_companies >= 100 THEN 'high_activity'
       END
FROM fund;
```

9.
Для каждой из категорий, назначенных в предыдущем задании, посчитайте округлённое до ближайшего целого числа среднее количество инвестиционных раундов, в которых фонд принимал участие. Выведите на экран категории и среднее число инвестиционных раундов. Отсортируйте таблицу по возрастанию среднего.


```python
SELECT 
       CASE
           WHEN invested_companies>=100 THEN 'high_activity'
           WHEN invested_companies>=20 THEN 'middle_activity'
           ELSE 'low_activity'
       END AS activity,
       ROUND(AVG(investment_rounds)) AS averrage
FROM fund
GROUP BY activity
ORDER BY averrage;
```

10.
Выгрузите таблицу с десятью самыми активными инвестирующими странами. Активность страны определите по среднему количеству компаний, в которые инвестируют фонды этой страны.
Для каждой страны посчитайте минимальное, максимальное и среднее число компаний, в которые инвестировали фонды, основанные с 2010 по 2012 год включительно.
Исключите из таблицы страны с фондами, у которых минимальное число компаний, получивших инвестиции, равно нулю. Отсортируйте таблицу по среднему количеству компаний от большего к меньшему.
Для фильтрации диапазона по годам используйте оператор BETWEEN.


```python
SELECT country_code,
       MIN(invested_companies) AS min_inv,
       MAX(invested_companies) AS max_inv,
       AVG(invested_companies) AS average
FROM fund
WHERE EXTRACT(YEAR FROM CAST(founded_at AS DATE)) BETWEEN 2010 AND 2012
GROUP BY country_code
HAVING MIN(invested_companies) <> 0
ORDER BY average DESC
LIMIT 10;
```

11.
Отобразите имя и фамилию всех сотрудников стартапов. Добавьте поле с названием учебного заведения, которое окончил сотрудник, если эта информация известна.


```python
SELECT p.first_name,
       p.last_name,
       edu.instituition
FROM people AS p
LEFT JOIN education AS edu ON p.id = edu.person_id;
```

12.
Для каждой компании найдите количество учебных заведений, которые окончили её сотрудники. Выведите название компании и число уникальных названий учебных заведений. Составьте топ-5 компаний по количеству университетов.


```python
SELECT com.name,
       COUNT(DISTINCT edu.instituition) AS count_univ
FROM people AS peo 
INNER JOIN education AS edu ON peo.id = edu.person_id
INNER JOIN company AS com ON peo.company_id = com.id
GROUP BY com.name
ORDER BY count_univ DESC
LIMIT 5;
```

13.
Составьте список с уникальными названиями закрытых компаний, для которых первый раунд финансирования оказался последним.


```python
SELECT DISTINCT name
FROM company AS com
INNER JOIN funding_round AS f_r ON com.id = f_r.company_id
WHERE com.status = 'closed'
AND is_first_round = 1
AND is_last_round = 1;
```

14.
Составьте список уникальных номеров сотрудников, которые работают в компаниях, отобранных в предыдущем задании.


```python
WITH 
i AS (SELECT DISTINCT(company_id) AS id 
      FROM funding_round 
      WHERE is_first_round = 1 
      AND is_last_round = 1 
      ORDER BY company_id), 
w AS (SELECT c.id 
      FROM i 
      LEFT JOIN company AS c ON i.id=c.id 
      WHERE c.status = 'closed') 
SELECT id 
FROM people
WHERE company_id IN (SELECT c.id
                     FROM i
                     LEFT JOIN company AS c ON i.id=c.id
                     WHERE c.status = 'closed')
```

15.
Составьте таблицу, куда войдут уникальные пары с номерами сотрудников из предыдущей задачи и учебным заведением, которое окончил сотрудник.


```python
WITH 
one AS (SELECT DISTINCT(company_id) AS id 
        FROM funding_round 
        WHERE is_first_round = 1 
        AND is_last_round = 1 
        ORDER BY company_id), 
two AS (SELECT c.id 
        FROM one 
        LEFT JOIN company AS c ON one.id=c.id 
        WHERE c.status = 'closed'), 
three AS (SELECT id 
          FROM people
          WHERE company_id IN (SELECT c.id
                               FROM one
                               LEFT JOIN company AS c ON one.id=c.id
                               WHERE c.status = 'closed'))
SELECT three.id,
       ed.instituition
FROM education AS ed
INNER JOIN three ON ed.person_id = three.id
GROUP BY three.id, ed.instituition;
```

16.
Посчитайте количество учебных заведений для каждого сотрудника из предыдущего задания.


```python
WITH 
one AS (SELECT DISTINCT(company_id) AS id 
        FROM funding_round 
        WHERE is_first_round = 1 
        AND is_last_round = 1 
        ORDER BY company_id), 
two AS (SELECT c.id 
        FROM one 
        LEFT JOIN company AS c ON one.id=c.id 
        WHERE c.status = 'closed'), 
three AS (SELECT id 
          FROM people
          WHERE company_id IN (SELECT c.id
                               FROM one
                               LEFT JOIN company AS c ON one.id=c.id
                               WHERE c.status = 'closed'))
SELECT three.id,
       COUNT(ed.instituition) AS count_inst
FROM education AS ed
INNER JOIN three ON ed.person_id = three.id
GROUP BY three.id;
```

17.
Дополните предыдущий запрос и выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники разных компаний. Нужно вывести только одну запись, группировка здесь не понадобится.


```python
WITH 
one AS (SELECT DISTINCT(company_id) AS id 
        FROM funding_round 
        WHERE is_first_round = 1 
        AND is_last_round = 1 
        ORDER BY company_id), 
two AS (SELECT c.id 
        FROM one 
        LEFT JOIN company AS c ON one.id=c.id 
        WHERE c.status = 'closed'), 
three AS (SELECT id 
          FROM people
          WHERE company_id IN (SELECT c.id
                               FROM one
                               LEFT JOIN company AS c ON one.id=c.id
                               WHERE c.status = 'closed')),
four AS (SELECT three.id,
                COUNT(ed.instituition) AS count_inst
                FROM education AS ed
                INNER JOIN three ON ed.person_id = three.id
                GROUP BY three.id)
SELECT AVG(count_inst)
FROM four;
```

18.
Напишите похожий запрос: выведите среднее число учебных заведений (всех, не только уникальных), которые окончили сотрудники Facebook*.
*(сервис, запрещённый на территории РФ)


```python
WITH 
one AS (SELECT DISTINCT(company_id) AS id 
        FROM funding_round 
        --WHERE is_first_round = 1 
        --AND is_last_round = 1 
        ORDER BY company_id), 
two AS (SELECT c.id 
        FROM one 
        LEFT JOIN company AS c ON one.id=c.id 
        WHERE c.name LIKE '%Facebook%'), 
three AS (SELECT id 
          FROM people
          WHERE company_id IN (SELECT c.id
                               FROM one
                               LEFT JOIN company AS c ON one.id=c.id
                               WHERE c.name LIKE '%Facebook%')),
four AS (SELECT three.id,
                COUNT(ed.instituition) AS count_inst
                FROM education AS ed
                INNER JOIN three ON ed.person_id = three.id
                GROUP BY three.id)
SELECT AVG(count_inst)
FROM four;
```

19.
Составьте таблицу из полей:
name_of_fund — название фонда;
name_of_company — название компании;
amount — сумма инвестиций, которую привлекла компания в раунде.
В таблицу войдут данные о компаниях, в истории которых было больше шести важных этапов, а раунды финансирования проходили с 2012 по 2013 год включительно.


```python
SELECT DISTINCT f.name AS name_of_fund,
       c.name AS name_of_company,
       f_r.raised_amount AS amount
FROM investment AS i
INNER JOIN company AS c ON c.id = i.company_id
INNER JOIN fund AS f ON f.id = i.fund_id
INNER JOIN funding_round AS f_r ON f_r.id = i.funding_round_id
WHERE c.milestones > 6
AND EXTRACT(YEAR FROM f_r.funded_at) BETWEEN 2012 AND 2013;
```

20.
Выгрузите таблицу, в которой будут такие поля:
название компании-покупателя;
сумма сделки;
название компании, которую купили;
сумма инвестиций, вложенных в купленную компанию;
доля, которая отображает, во сколько раз сумма покупки превысила сумму вложенных в компанию инвестиций, округлённая до ближайшего целого числа.
Не учитывайте те сделки, в которых сумма покупки равна нулю. Если сумма инвестиций в компанию равна нулю, исключите такую компанию из таблицы.
Отсортируйте таблицу по сумме сделки от большей к меньшей, а затем по названию купленной компании в алфавитном порядке. Ограничьте таблицу первыми десятью записями.


```python
SELECT c.name,
       acq.price_amount,
       com.name,
       com.funding_total,
       ROUND(acq.price_amount / com.funding_total)
FROM acquisition AS acq
INNER JOIN company AS c ON acq.acquiring_company_id = c.id
INNER JOIN company AS com ON acq.acquired_company_id = com.id
WHERE acq.price_amount <>0
AND com.funding_total <>0
ORDER BY acq.price_amount DESC, com.name ASC
LIMIT 10;
```

21.
Выгрузите таблицу, в которую войдут названия компаний из категории social, получившие финансирование с 2010 по 2013 год включительно. Выведите также номер месяца, в котором проходил раунд финансирования.


```python
SELECT c.name,
       EXTRACT(MONTH FROM f_r.funded_at)
FROM company AS c
RIGHT JOIN funding_round AS f_r ON c.id = f_r.company_id
WHERE category_code = 'social'
AND EXTRACT(YEAR FROM f_r.funded_at) BETWEEN 2010 AND 2013
AND raised_amount <> 0;
```

22.
Отберите данные по месяцам с 2010 по 2013 год, когда проходили инвестиционные раунды. Сгруппируйте данные по номеру месяца и получите таблицу, в которой будут поля:
номер месяца, в котором проходили раунды;
количество уникальных названий фондов из США, которые инвестировали в этом месяце;
количество компаний, купленных за этот месяц;
общая сумма сделок по покупкам в этом месяце.


```python
WITH 
one AS (SELECT EXTRACT(MONTH FROM funded_at) AS mth, 
                COUNT(DISTINCT f.id) AS usa_funds_count 
         FROM   funding_round AS fr 
                JOIN investment i ON i.funding_round_id = fr.id 
                JOIN fund AS f ON f.id = i.fund_id 
         WHERE  f.country_code = 'USA' 
                AND EXTRACT(YEAR FROM funded_at) BETWEEN 2010 AND 2013 
         GROUP  BY mth), 
two AS (SELECT EXTRACT(MONTH FROM acquired_at) AS mth, 
                COUNT(acquired_company_id) AS acq_company_count, 
                SUM(price_amount) AS total 
         FROM acquisition
         WHERE  EXTRACT(YEAR FROM acquired_at) BETWEEN 2010 AND 2013
         GROUP  BY mth) 
SELECT one.mth, 
       one.usa_funds_count, 
       two.acq_company_count, 
       two.total 
FROM one JOIN two ON one.mth = two.mth;
```

23.
Составьте сводную таблицу и выведите среднюю сумму инвестиций для стран, в которых есть стартапы, зарегистрированные в 2011, 2012 и 2013 годах. Данные за каждый год должны быть в отдельном поле. Отсортируйте таблицу по среднему значению инвестиций за 2011 год от большего к меньшему.


```python
WITH
year_2011 AS (SELECT c.country_code,
                     AVG(c.funding_total) AS avg_2011
              FROM company AS c
              WHERE EXTRACT(YEAR FROM c.founded_at) = 2011
              GROUP BY c.country_code),
year_2012 AS (SELECT c.country_code,
                     AVG(c.funding_total) AS avg_2012
              FROM company AS c
              WHERE EXTRACT(YEAR FROM c.founded_at) = 2012
              GROUP BY c.country_code), 
year_2013 AS (SELECT c.country_code,
                     AVG(c.funding_total) AS avg_2013
              FROM company AS c
              WHERE EXTRACT(YEAR FROM c.founded_at) = 2013
              GROUP BY c.country_code)

SELECT year_2011.country_code,
       year_2011.avg_2011,
       year_2012.avg_2012,
       year_2013.avg_2013    
FROM year_2011
INNER JOIN year_2012 ON year_2011.country_code = year_2012.country_code
INNER JOIN year_2013 ON year_2012.country_code = year_2013.country_code
ORDER BY year_2011.avg_2011 DESC;
```
