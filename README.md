# SQLAlchemy Cheat SHeet

## Getting started

````
pip install SQLAlchemy
````

## Boilerplate code

````python
import os

from sqlalchemy import create_engine


engine = create_engine(
    os.environ['DATABASE_URL'],
    connect_args={'sslmode':'require'}
)

````

## Raw SQL

Escaping variables in SQLAlchemy is a bit tricky. First you should notice that `session.execute` and `engine.execute` have different syntax [1]. Secondly, if you use escaping mechanism provided by `engine.execute`, you should notice that MySQL and PostgreSQL drivers have different way of escaping. MySQL uses `:name` and PostgreSQL uses `%(name)s`.

````python
# Escaping using engine.execute
# If you are using db.session, remember to use `db.session.get_bind()` to get engine.
q = engine.execute('''SELECT name FROM users WHERE name=%(name)s''', name='Jack')
for row in q:
    print(row)

# Backend neutral escaping: works both on MySQL and PostgreSQL
engine.execute(
    db.text('SELECT name FROM users WHERE name = :name'),
    name='Demo'
)
````

[1] Syntax differences:

- sqlalchemy.orm.session.Session.execute http://docs.sqlalchemy.org/en/latest/orm/session_api.html#sqlalchemy.orm.session.Session.execute
- sqlalchemy.engine.Connection.execute http://docs.sqlalchemy.org/en/latest/core/connections.html#sqlalchemy.engine.Connection.execute

## Subquery

````python
data = (
    db.session.query(
        User.id,
        db.select(
            [db.func.array_agg(Hobby.name)],
            from_obj=Hobby
        ).where(Hobby.created_by_user_id == User.id).correlate(User.__table__).label('hobbies_created_by_user')
    )
    .filter(User.country == 'Finland')
    .order_by(User.id.asc())
    .all()
)
````

## Delete

````python
User.query.filter(
    User.id.in_(ids)
).delete(synchronize_session='fetch')
````
