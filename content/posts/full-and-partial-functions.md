---
title: "Full and Partial Functions"
date: 2023-05-08T20:29:03+03:00
draft: false
---

## Full and partial functions 

Functions in programming can be splitted into two categories:
1. Partial function - there is at least one combination of arguments that has no defined result. Usually it leads to undefined behaviour or raised exception
![](/images/full-and-partial-functions-partial.png)
```python
# What if `raw_number` cannot be parsed to integer? ValueError will be raised, but it's implicit result
number = int(raw_number)
```
2. Full function - every possible combination of arguments has it's own result or action:
![](/images/full-and-partial-functions-full.png)
```python
# the function returns either int or None explicitly
def int_or_none(raw: str) -> int | None:
	try:
		return int(raw)
	except ValueError:
		return None
```

## How partial function can be converted to full?
It's hard to work with partial functions. You can't trust their contract (input and output types), so you have to read through it's source to make what it can make for undefined inputs, what kind of side effects produces and which exceptions rises. Let's take an example:

```python
# that if `email` is not valid email?
# `delete_older_than` must has a format 'YYYY-MM-DD', but what if it's not?
def cleanup_email_folder(email: str, delete_older_than: str) -> None:
	...
```

### Weaking result type
We can map undefined input combinations to different input type like `None` or `Result[Error]`. 
```python
# returns `ValueError` if `email` or `delete_older_than` have incorrect format
def cleanup_email_folder(email: str, delete_older_than: str) -> Result[None, ValueError]:
	...
```
It's a simple solution, but has some problems:
1. We should don't forget to check return type of function. In this example, it's easy to just call it and don't check the result.
2. That if caller already checked provided arguments elsewhere? In this case, function's checks become redundant.
3. The command is responsible for checking input arguments checking and the cleanuping itself.

### Strengthing input types
```python
class Email(str):
	def __init__(self, input: str) -> None:
		self._chech_is_valid_email(input)
		super().__init__(input)

# the function recieves `Email` instance, so it can be sure it's valid email
# `delete_older_than` is `datetime`, so the function can format it inside and can be sure it's actually some point in time
def cleanup_email_folder(email: Email, delete_older_than: datetime.datetime) -> None:
	...
```

Strengthing input types means eliminating undefined input arguments, so function can't be called using them. Usually it accomplished using more specific type or creating custom type that can be created if all checks passed already.

The second appoach solves all three problems of previous solution:
1. Caller cannot forget to check input arguments because instance of specific instance is a proof the argument is correct.
2. Caller can create instance somewhere in the code and use it any times, so there is no check duplication
3. The function itself is no longer responsible for format checking argument's correctness.

But this way of converting partial functions to full has it's own disadvantage. Sometimes it's hard to split value checking from it's actual usage. For example, `email` can doesn't exist in email server, so we can combine both methods to make function full and more explicit:
```python
# we state the function can returns `EmailServerError` explicitly
def cleanup_email_folder(email: Email, delete_older_than: datetime.datetime) -> Result[None, EmailServerError]:
	...
```

## Sources
- [Original article, gives explanations on context of Haskell](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
