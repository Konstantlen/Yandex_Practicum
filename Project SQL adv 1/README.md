# Проект Stackoverflow

* Таблица stackoverflow.badges. Хранит информацию о значках, которые присуждаются за разные достижения. Например, пользователь, правильно ответивший на большое количество вопросов про PostgreSQL, может получить значок postgresql.
* Таблица stackoverflow.post_types. Содержит информацию о типе постов. Их может быть два: Question — пост с вопросом; Answer — пост с ответом.
* Таблица stackoverflow.posts. Содержит информацию о постах
* Таблица stackoverflow.users. Содержит информацию о пользователях
* Таблица stackoverflow.vote_types. Содержит информацию о типах голосов. Голос — это метка, которую пользователи ставят посту. Типов бывает несколько:
  - UpMod — такую отметку получают посты с вопросами или ответами, которые пользователи посчитали уместными и полезными.
  - DownMod — такую отметку получают посты, которые показались пользователям наименее полезными.
  - Close — такую метку ставят опытные пользователи сервиса, если заданный вопрос нужно доработать или он вообще не подходит для платформы.
  - Offensive — такую метку могут поставить, если пользователь ответил на вопрос в грубой и оскорбительной манере, например, указав на неопытность автора поста.
  - Spam — такую метку ставят в случае, если пост пользователя выглядит откровенной рекламой.
* Таблица stackoverflow.votes Содержит информацию о голосах за посты.

## 1. Найдите количество вопросов, которые набрали больше 300 очков или как минимум 100 раз были добавлены в «Закладки».

SELECT COUNT(p.id)  
FROM stackoverflow.posts AS p  
WHERE p.post_type_id = 1  
AND (p.score > 300 OR p.favorites_count >= 100);  

## 2. Сколько в среднем в день задавали вопросов с 1 по 18 ноября 2008 включительно? Результат округлите до целого числа.

SELECT ROUND(AVG(questions))
FROM
(SELECT creation_date::date as dt,
       COUNT(id) as questions
FROM stackoverflow.posts as p
WHERE post_type_id = 1
AND creation_date::date BETWEEN '2008-11-1' AND '2008-11-18'
GROUP BY dt) AS shit;

## 3. Сколько пользователей получили значки сразу в день регистрации? Выведите количество уникальных пользователей.

SELECT COUNT(u.id)
FROM stackoverflow.users AS u
JOIN stackoverflow.badges AS b ON u.id=b.user_id
WHERE u.creation_date::date = b.creation_date::date


## 4. Сколько уникальных постов пользователя с именем Joel Coehoorn получили хотя бы один голос?

SELECT COUNT(DISTINCT p.id)
FROM stackoverflow.posts as p
JOIN stackoverflow.votes as v ON p.id = v.post_id
JOIN stackoverflow.users as u ON p.user_id = u.id
WHERE u.display_name = 'Joel Coehoorn';

## 5. Выгрузите все поля таблицы vote_types. Добавьте к таблице поле rank, в которое войдут номера записей в обратном порядке. Таблица должна быть отсортирована по полю id.

SELECT *,
       RANK () OVER (ORDER BY id DESC)
FROM stackoverflow.vote_types 
ORDER BY id;

## 6. Отберите 10 пользователей, которые поставили больше всего голосов типа Close. Отобразите таблицу из двух полей: идентификатором пользователя и количеством голосов. Отсортируйте данные сначала по убыванию количества голосов, потом по убыванию значения идентификатора пользователя.

SELECT u.id as i,
       COUNT(v.id) as f
FROM stackoverflow.users as u
JOIN stackoverflow.votes as v ON u.id=v.user_id
JOIN stackoverflow.vote_types as vt ON vt.id = v.vote_type_id
WHERE vt.name = 'Close'
GROUP BY i
ORDER BY f DESC, i DESC
LIMIT 10;

## 7. Отберите 10 пользователей по количеству значков, полученных в период с 15 ноября по 15 декабря 2008 года включительно. Отобразите несколько полей:
* идентификатор пользователя;
* число значков;
* место в рейтинге — чем больше значков, тем выше рейтинг.
Пользователям, которые набрали одинаковое количество значков, присвойте одно и то же место в рейтинге.
Отсортируйте записи по количеству значков по убыванию, а затем по возрастанию значения идентификатора пользователя.

SELECT DISTINCT u.id as i,
       COUNT(b.id) as badges,
       DENSE_RANK() OVER (ORDER BY COUNT(b.id)DESC) 
FROM stackoverflow.users as u
JOIN stackoverflow.badges as b ON u.id=b.user_id
WHERE b.creation_date::date BETWEEN '2008-11-15' AND '2008-12-15'
GROUP BY i
ORDER BY badges DESC, i
LIMIT 10;

## 8. Сколько в среднем очков получает пост каждого пользователя?
Сформируйте таблицу из следующих полей:
* заголовок поста;
* идентификатор пользователя;
* число очков поста;
* среднее число очков пользователя за пост, округлённое до целого числа.
Не учитывайте посты без заголовка, а также те, что набрали ноль очков.

SELECT p.title t,
       u.id i,
       p.score s,
       ROUND(AVG(p.score) OVER (PARTITION BY u.id)) as avg
FROM stackoverflow.users as u
JOIN stackoverflow.posts as p ON u.id=p.user_id
WHERE p.title != ''
AND p.score != 0
GROUP BY t, i, s, p.id;

## 9. Отобразите заголовки постов, которые были написаны пользователями, получившими более 1000 значков. Посты без заголовков не должны попасть в список.

SELECT p.title t
FROM stackoverflow.posts AS p
WHERE p.user_id IN
(SELECT u.id
FROM stackoverflow.users AS u
JOIN stackoverflow.badges AS b ON u.id=b.user_id
GROUP BY u.id
HAVING COUNT(b.id)>1000)
AND p.title IS NOT NULL;

## 10. Напишите запрос, который выгрузит данные о пользователях из США (англ. United States). Разделите пользователей на три группы в зависимости от количества просмотров их профилей:
* пользователям с числом просмотров больше либо равным 350 присвойте группу 1;
* пользователям с числом просмотров меньше 350, но больше либо равно 100 — группу 2;
* пользователям с числом просмотров меньше 100 — группу 3.
Отобразите в итоговой таблице идентификатор пользователя, количество просмотров профиля и группу. Пользователи с нулевым количеством просмотров не должны войти в итоговую таблицу.

SELECT u.id,
       u.views,
       CASE
            WHEN u.views >= 350 THEN 1
            WHEN 350 > u.views AND u.views >= 100 THEN 2
            WHEN u.views < 100 THEN 3
       END as gr
FROM stackoverflow.users AS u
WHERE u.views != 0
AND u.location LIKE ('%United States%');

## 11. Дополните предыдущий запрос. Отобразите лидеров каждой группы — пользователей, которые набрали максимальное число просмотров в своей группе. Выведите поля с идентификатором пользователя, группой и количеством просмотров. Отсортируйте таблицу по убыванию просмотров, а затем по возрастанию значения идентификатора.

WITH g AS
(SELECT u.id,
       u.views,
       CASE
            WHEN u.views >= 350 THEN 1
            WHEN 350 > u.views AND u.views >= 100 THEN 2
            WHEN u.views < 100 THEN 3
       END as gr
FROM stackoverflow.users AS u
WHERE u.views != 0
AND u.location LIKE ('%United States%')
 )
SELECT id,
         views,
         gr,
         MAX(views) OVER (PARTITION BY g.gr) AS vmax
FROM g
ORDER BY views DESC, id;

## 12. Посчитайте ежедневный прирост новых пользователей в ноябре 2008 года. Сформируйте таблицу с полями:
* номер дня;
* число пользователей, зарегистрированных в этот день;
* сумму пользователей с накоплением.

SELECT DISTINCT EXTRACT(DAY FROM creation_date::date) as cd,
       COUNT(id) OVER (PARTITION BY creation_date::date) as n,
       COUNT(id) OVER (ORDER BY creation_date::date) s
FROM stackoverflow.users as u
WHERE creation_date::date BETWEEN '2008-11-01' AND '2008-11-30';

## 13. Для каждого пользователя, который написал хотя бы один пост, найдите интервал между регистрацией и временем создания первого поста. Отобразите:
* идентификатор пользователя;
* разницу во времени между регистрацией и первым постом.

SELECT DISTINCT users.id as u,
       MIN(posts.creation_date) OVER (PARTITION BY posts.user_id) - users.creation_date as dif
FROM stackoverflow.posts
JOIN stackoverflow.users ON posts.user_id=users.id
ORDER BY u;
