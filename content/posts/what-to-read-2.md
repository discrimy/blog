---
title: "What to Read 2"
date: 2024-05-21T08:58:10+03:00
draft: false
---

# What to read #2
List of links of interest materials to read with short description.


## [Code is run more than read](https://olano.dev/blog/code-is-run-more-than-read)
Don't forget about priorities in software development.

> **user > ops > dev**
>
> **biz > ops > dev**
>
> **biz ≹ user**

## [Use `object` instead of `Any`](https://adamj.eu/tech/2021/05/07/python-type-hints-use-object-instead-of-any/)
In Python typing, `Any` **disables** checks. If you don't care about type of variable (for example passing `*args, **kwargs` to `super().method(*args, **kwargs)` and do not use them youself), use `object`

```python
from example import Widget


class BlueWidget(Widget):
    def __init__(self, *args: object, blueness: int = 50, **kwargs: object) -> None:
        super().__init__(*args, **kwargs)
        self.blueness = blueness
```

## [Python errors as values](https://www.inngest.com/blog/python-errors-as-values)
Throwing exceptions as error handling is implicit for caller. Use union types with exception as function result type to explicitly define errors the function can return.

```python
# Define a function that returns a union of a User and an error
def get_user(user_id: str) -> User | Exception:
    rows = users.find(user_id=user_id)
    if len(rows) == 0:
        return Exception("user not found")
    return rows[0]

def rename_user(user_id: str, name: str) -> User | Exception:
    # Consume the function
    user = get_user(user_id)
    if isinstance(user, Exception):
        return user
    user.name = name
    return user
```