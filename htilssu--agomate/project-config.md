---
trigger: always_on
description: TITLE: Simplify SQLAlchemy ORM Mapping with Mapped (Typed and Concise) - Python
---

TITLE: Simplify SQLAlchemy ORM Mapping with Mapped (Typed and Concise) - Python
DESCRIPTION: This snippet demonstrates a further simplified version of the SQLAlchemy ORM declarative models, leveraging Python type annotations with `Mapped` to imply column types and nullability. It shows how `mapped_column` can be omitted or simplified when type annotations provide sufficient information.
SOURCE: https://github.com/sqlalchemy/sqlalchemy/blob/main/doc/build/changelog/whatsnew_20.rst#_snippet_16

LANGUAGE: Python
CODE:
```
from typing import List
from typing import Optional
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import relationship


class Base(DeclarativeBase):
    pass


class User(Base):
    __tablename__ = "user_account"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(30))
    fullname: Mapped[Optional[str]]
    addresses: Mapped[List["Address"]] = relationship(back_populates="user")


class Address(Base):
    __tablename__ = "address"

    id: Mapped[int] = mapped_column(primary_key=True)
    email_address: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey("user_account.id"))
    user: Mapped["User"] = relationship(back_populates="addresses")
```

----------------------------------------

TITLE: Configuring Relationship Collection as Set with Mapped in SQLAlchemy ORM
DESCRIPTION: This snippet shows how to configure a one-to-many relationship to use a Python set as its collection type in a Declarative mapping using the `Mapped` annotation. The `Parent` class's `children` relationship will store `Child` objects in a set, suitable for collections where order doesn't matter and uniqueness is desired.
SOURCE: https://github.com/sqlalchemy/sqlalchemy/blob/main/doc/build/orm/collection_api.rst#_snippet_1

LANGUAGE: python
CODE:
```
from sqlalchemy import ForeignKey

from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import relationship


class Base(DeclarativeBase):
    pass


class Parent(Base):
    __tablename__ = "parent"

    parent_id: Mapped[int] = mapped_column(primary_key=True)

    # use a set
    children: Mapped[set["Child"]] = relationship()


class Child(Base):
    __tablename__ = "child"

    child_id: Mapped[int] = mapped_column(primary_key=True)
    parent_id: Mapped[int] = mapped_column(ForeignKey("parent.id"))
```

----------------------------------------

TITLE: Defining Declarative Mapping with DeclarativeBase and Mapped - SQLAlchemy Python
DESCRIPTION: This snippet illustrates the standard Declarative mapping approach in SQLAlchemy 2.0+. It defines a base class `Base` inheriting from `DeclarativeBase`, and a `User` class inheriting from `Base`. The `User` class defines database columns using type annotations with `Mapped` and the `mapped_column` function, along with the `__tablename__` class attribute.
SOURCE: https://github.com/sqlalchemy/sqlalchemy/blob/main/doc/build/orm/mapping_styles.rst#_snippet_0

LANGUAGE: Python
CODE:
```
from sqlalchemy import Integer, String, ForeignKey
from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column


# declarative base class
class Base(DeclarativeBase):
    pass


# an example mapping using the base
class User(Base):
    __tablename__ = "user"

    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    fullname: Mapped[str] = mapped_column(String(30))
    nickname: Mapped[Optional[str]]
```

----------------------------------------

TITLE: Defining Many-to-Many Relationship with Set using Mapped
DESCRIPTION: This snippet demonstrates how to define a many-to-many relationship using SQLAlchemy ORM's declarative style with type annotations (`Mapped`) and the `relationship.secondary` parameter. It specifically shows using a `Set` for the collection type. The `Parent` class is linked to a `Child` class via an `association_table` (not shown).
SOURCE: https://github.com/sqlalchemy/sqlalchemy/blob/main/doc/build/orm/basic_relationships.rst#_snippet_16

LANGUAGE: Python
CODE:
```
class Parent(Base):
    __tablename__ = "left_table"

    id: Mapped[int] = mapped_column(primary_key=True)
    children: Mapped[Set["Child"]] = relationship(secondary=association_table)
```

----------------------------------------

TITLE: Configuring Relationship Collection as List with Mapped in SQLAlchemy ORM
DESCRIPTION: This snippet demonstrates how to configure a one-to-many relationship to use a Python list as its collection type in a Declarative mapping using the `Mapped` annotation. The `Parent` class has a relationship named `children` that will hold `Child` objects in a list.
SOURCE: https://github.com/sqlalchemy/sqlalchemy/blob/main/doc/build/orm/collection_api.rst#_snippet_0

LANGUAGE: python
CODE:
```
from sqlalchemy import ForeignKey

from sqlalchemy.orm import DeclarativeBase
from sqlalchemy.orm import Mapped
from sqlalchemy.orm import mapped_column
from sqlalchemy.orm import relationship


class Base(DeclarativeBase):
    pass


class Parent(Base):
    __tablename__ = "parent"


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/htilssu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
