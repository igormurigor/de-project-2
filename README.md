# Проект 1
Опишите здесь поэтапно ход решения задачи. Вы можете ориентироваться на тот план выполнения проекта, который мы предлагаем в инструкции на платформе.
# Витрина RFM

## 1.1. Выясните требования к целевой витрине.

Постановка задачи выглядит достаточно абстрактно - постройте витрину. Первым делом вам необходимо выяснить у заказчика детали. Запросите недостающую информацию у заказчика в чате.

Зафиксируйте выясненные требования. Составьте документацию готовящейся витрины на основе заданных вами вопросов, добавив все необходимые детали.

-----------
{
1. Витрина должна располагаться в той же базе в схеме analysis 

2. Витрина должна состоять из таких полей:   
user_id   
recency (число от 1 до 5)  
frequency (число от 1 до 5)  
monetary_value (число от 1 до 5)

3. Глубина данных:с начала 2021 года
   
4. Назвать  витрину dm_rfm_segments
   
5. Обновления не нужны.
   
6. Closed - успешно выполненый заказ


}



## 1.2. Изучите структуру исходных данных.

Полключитесь к базе данных и изучите структуру таблиц.

Если появились вопросы по устройству источника, задайте их в чате.

Зафиксируйте, какие поля вы будете использовать для расчета витрины.

-----------

{ Для подсчета Recency используем данные в таблицах users (id) и orders(user_id,order_ts,status), сджоиним таблицы ,чтобы получить полный список клиентов тех кто делал заказ и кто его не делал никогда , отфильтруем данные по полю status, в представление попадут только Closed -заказы


Для продсчета Frequency используем таблицу orders, посчитаем количество заказов по order_id со статусом Closed для каждого клиента

Для подсчета Monetary Value используем таблицу orders, суммируем значение cost  по всем заказам для каждого покупателя 



}


## 1.3. Проанализируйте качество данных

Изучите качество входных данных. Опишите, насколько качественные данные хранятся в источнике. Так же укажите, какие инструменты обеспечения качества данных были использованы в таблицах в схеме production.

-----------

{ В схеме production  в таблице users   в колонке name есть  Null - значения  . В таблицах  orders, users полных дублей нет

}


## 1.4. Подготовьте витрину данных

Теперь, когда требования понятны, а исходные данные изучены, можно приступить к реализации.

### 1.4.1. Сделайте VIEW для таблиц из базы production.**

Вас просят при расчете витрины обращаться только к объектам из схемы analysis. Чтобы не дублировать данные (данные находятся в этой же базе), вы решаете сделать view. Таким образом, View будут находиться в схеме analysis и вычитывать данные из схемы production. 

Напишите SQL-запросы для создания пяти VIEW (по одному на каждую таблицу) и выполните их. Для проверки предоставьте код создания VIEW.

```SQL
CREATE VIEW view_orderitems
AS SELECT * FROM orderitems;

CREATE VIEW view_orders
AS SELECT * FROM orders;

CREATE VIEW view_orderstatuses
AS SELECT * FROM orderstatuses;

CREATE VIEW view_orderstatuslog
AS SELECT * FROM orderstatuslog;

CREATE VIEW view_products
AS SELECT * FROM products;

CREATE VIEW view_users
AS SELECT * FROM users;


```

### 1.4.2. Напишите DDL-запрос для создания витрины.**

Далее вам необходимо создать витрину. Напишите CREATE TABLE запрос и выполните его на предоставленной базе данных в схеме analysis.

```SQL
create table dm_rfm_segments (

user_id int unique ,
recency int ,
frequency int,
monetary_value int )

```

### 1.4.3. Напишите SQL запрос для заполнения витрины

Наконец, реализуйте расчет витрины на языке SQL и заполните таблицу, созданную в предыдущем пункте.

Для решения предоставьте код запроса.

```SQL
with tt as(

select vu.id as user_id , last_order::date ,orders_q, cost_sum
from view_users vu left join ( select  vo.user_id as user_id  ,
							           max(vo.order_ts::timestamp::date) as last_order,
							           count( distinct order_id) as orders_q,
							  		   sum(cost) as cost_sum
							   from view_orders vo
							  where vo.status=4
							 group by vo.user_id  ) as subvo ON vu.id=subvo.user_id )
							 
select user_id,
ntile(5) Over(Order by coalesce(last_order, '2021-01-01')) as Recency,
ntile(5) Over(Order by coalesce(orders_q, '0')) as Frequency,
ntile(5) Over(Order by coalesce(cost_sum, '0')) as Monetary
from tt


```
