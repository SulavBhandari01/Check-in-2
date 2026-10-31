

**Project:** International Student Finance Tracker

**Student:** Sulav Bhandari(B369Q634)

**Due:** Week 11


## 1. Project overview

My project is to build an International Student Finance Tracker. It is designed for students who are living and studying in another country. The app helps them record income and expenses, organize spending, set monthly budgets and work with more than one currency.
The first version of my database used two tables: `Users` and `Transactions`. When I added categories, budgets and currency conversion, I expanded the design so the same information would not have to be stored repeatedly.

## 2. Original two-table design
These are my two original table that i have done in 1st checkin.

```text

Users(

user_id PK,

user_name,

email

)

  

Transactions(

transaction_id PK,

user_id FK,

user_name,

category_name,

amount,

currency_code,

exchange_rate,

converted_amount,

transaction_date,

description

)

```

The relationship was:

```text

Transactions.user_id -> Users.user_id

```


This design worked for a small example , but it had a weakness: the user's name, category information and currency information could be copied into many transaction rows. That could make the data difficult to update correctly.

  

## 3. Functional dependencies

These are the main ways the values in my database depend on one another:


```text

user_id -> name, username, email, base_currency

username -> user_id, name, email

email -> user_id, name, username

category_id -> user_id, category_name, category_type

currency_code -> currency_name, currency_symbol

exchange_rate_id -> from_currency, to_currency, rate, effective_date

transaction_id -> user_id, category_id, amount, currency_code,

base_amount, transaction_date, description

budget_id -> user_id, category_id, month, limit_amount

```

For example, `user_id` determines the user's account details, while `currency_code` determines the currency name and symbol. This is why those facts belong in separate tables instead of being repeated in every transaction.

  

## 4. Normalization process

### First Normal Form (1NF)

To satisfy 1NF:
- Every field contains one value.
- Each row represents one transaction.
- There are no lists of expenses inside one column.
- Every row has a primary key.
At this point, the data is atomic but repeated information still exists. For example, the same user name may appear in many transaction rows.

  
### Second Normal Form (2NF)

The original tables use single-column primary keys, so there are no partial dependencies on part of a composite key. However, the design still mixes information about different things in the same table.


For example:
```text

user_id -> user_name

currency_code -> currency information

category_name -> category information

```

These values describe users, currencies, and categories, not the individual transaction itself. They should be stored once and referenced when needed.


### Third Normal Form (3NF)  

To reach 3NF, I separated the independent information into different tables. Each non-key field now depends on the primary key of its own table.
The final tables are:

```text

Users

Categories

Currencies

ExchangeRates

Transactions

Budgets

```

  

## 5. Final database design

The following tables show the final 3NF design in a simple format. The ERD of the database is shown below.


[![[ERD.png]]](https://github.com/SulavBhandari01/Check-in-2/blob/main/ER%20Diagram.png)


## 7. Anomalies in the original design
### Update anomaly

If a user's name changes, the original design requires updating the name in every transaction row. If one row is missed, the database contains different versions of the user's name.

The normalized design stores the name once in `Users`, so it only needs one update.


### Insert anomaly
The original design cannot store a new category until a transaction exists. The normalized design allows a new category to be added independently to `Categories`.
### Delete anomaly
If the only transaction for a user is deleted, the original design could also lose the only stored copy of related user information. The normalized design stores users and categories separately.

  

## 8. Data-integrity rules

- Every table has a primary key.

- Foreign keys prevent records from becoming disconnected.

- Usernames and email addresses must be unique.

- Category names are unique for each user.

- A user cannot have two budgets for the same category and month.

- Currency codes must exist in the `Currencies` table.

- Money values use decimal numbers instead of floating-point numbers.

- Amounts, exchange rates, and budgets must be greater than zero.

- A budget must use an expense category.

- Transactions and budgets must belong to the correct user.

  
## 9.Why normalization preserves the data

Normalization does not remove the original transaction information. The information can be reconstructed with joins:

```sql

SELECT

t.id,

u.username,

c.name AS category,

t.amount,

t.currency_code,

t.base_amount,

t.transaction_date,

t.description

FROM transactions AS t

JOIN users AS u ON u.id = t.user_id

JOIN categories AS c ON c.id = t.category_id;

```

  

This query joins the normalized tables and displays the information needed by the app. The important point is that the user and category details are not duplicated in every transaction row.

  

## 10. Problem-solving reflection

The hardest part was deciding which information belonged directly to a transaction and which information described a different entity. The original design was easy to understand but it repeated the same values and could lead to inconsistent data.
I worked through this by identifying the functional dependencies first. Then I separated the user, category, currency, exchange-rate, transaction and budget information into their own tables. Finally, I connected the tables using primary keys and foreign keys.

This final 3NF design is easier to maintain because each fact has one main location. It still supports the original purpose of the app while also making future features such as reports, budgets and currency conversion easier to add.

