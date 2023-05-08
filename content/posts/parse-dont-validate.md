---
title: "Parse, Don't Validate"
date: 2023-05-08T20:27:18+03:00
draft: false
---

# Parse, don't validate

That the difference between parsing and validation? Well, let's give definitions for both of them.
- Parsing - converting some less structural data to some more structures. For example, converting `str` to `int` or json via `json.loads` it a parsing, not every `string` can be converted to them.
- Validation - checking some data is in some subset of valid format. For example, validation `str` is a number can be achived by checking that every character is a digit.

But what the significant difference between using one or another?

## Preserving validation information
Parsing means converting less structured information to more structured. It means parsing preserves that the data has correct format. Validation just returns the answer to question "Is the provided data has correct format or not?", but the data doesn't change.

We can look at this in terms of proof. If we parsed some raw data, then the parsed instance is a proof that the data is correct. If we validate some raw data, then the proof is implicit, there'is no way to express it in code.

```python
# returns True if phone is valid, but phone itself will be unchanged
def is_phone(raw: str) -> bool:
	...
if is_phone(phone):
	...

# `Phone` instance preserves that provided string is actual a phone number
class Phone(str):
	def __init__(self, raw: str) -> None:
		self._check_is_phone(raw)
		super().__init__(raw)
phone = Phone(phone)
...
```

## Sources
- [Original article, gives explanations on context of Haskell](https://lexi-lambda.github.io/blog/2019/11/05/parse-don-t-validate/)
- [Parsed value is a proof (comment on Reddit)](https://www.reddit.com/r/typescript/comments/zm9st3/comment/j0bbpa7/?utm_source=share&utm_medium=web2x&context=3)
