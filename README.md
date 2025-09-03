# UUID-compatible Typed IDs with prefixes
`[https://python-bitshares.readthedocs.io/en/latest/]`
[![Documentation](https://readthedocs.org/projects/typeid/badge/?version=latest)](https://typeid.readthedocs.io/en/latest/?badge=latest)

`typid` is a python library that allows to create specific id types much like
stripe uses them across their platform to distinguish different data models from
their id already. `typeid` is compatible to the UUID library and as such can act
as a drop-in replacement.

A type id has the following string representation:

    user_nnv5rr3rfk3hgry6xrygr
    └──┘ └───────────────────┘
    type    uuid (base58)

## How to use `typeid`

First generate the Types using generate(...), then use them like so:

```python
from uuid_extensions import uuid7
from typeid import generate

typeids = generate("sk", "pk", "user", "apikey")

# Now they can be used with SkId, PkId, UserId, and ApikeyId
assert str(typeids.SkId(uuid7().hex)).startswith("sk_")
assert str(typeids.PkId(uuid7().hex)).startswith("pk_")
assert str(typeids.UserId(uuid7().hex)).startswith("user_")
assert str(typeids.ApikeyId(uuid7().hex)).startswith("apikey_")

# Obviously, loading a prefixed type id in the correct Type works just fine
assert (typeids.UserId("user_nnv5rr3rfk3hgry6xrygr").hex == "0664c4135ed83faabd4bc0dc33839c9f")

# Use custom suffixes, just in case you need them
other_ids.generate("admin", suffix="IdUseWithCare")
assert str(other_ids.AdminIdUseWithCare(uuid7().hex)).startswith("admin_")
```

## SqlAlchemy Support

The library also allows to create classes specifically for the use with
sqlalchemy.

#### Example

```python
from typeid import generate

typeids = generate("user")

class User(Base):
    __tablename__ = "user"

    # use typeids.UserId in the Mapper[]
    id: Mapped[typeids.UserId] = mapped_column(
        # use typeids.UserId in the mapped_column!
        typeids.UserId, primary_key=True, default=uuid.uuid4
    )

    name: str = Column(String(128), unique=True)

user = User(name="John Doe")
(db,) = get_session()
db.add(user)
db.commit()
db.refresh(user)
print(type(user.id))
# <class 'typeid.UserId'>

print((user.id))
# user_qvnrsem3skqho2hgni2mfk
```
