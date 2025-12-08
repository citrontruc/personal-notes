# Database

## ORMs

Some popular tools in order to deal with data in a database are Django with the django ORM and SQLAlchemy.

Example SQLAlchemy:
```python
from sqlalchemy import create_engine
from sqlalchemy import select

engine = create_engine("sqlite://", echo=True)

session = Session(engine)

stmt = select(User).where(User.name.in_(["spongebob", "sandy"]))

for user in session.scalars(stmt):
    print(user)
```

SQLAlchemy behaves in mostly the same way as django (need to commit changes before is all).

For Django ORM, see django cheatsheet.
