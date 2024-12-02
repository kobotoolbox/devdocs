---
title: KPI Backend Development
---

## Code style

- Always run the black autoformatter on new and modified code. KPI has one custom rule set, prefer `'` over `"`.
- Avoid unnecessary branching if statements when possible. For example, break early.

Bad

```python
def foo():
  if is_ok:
    do_the_thing()
```

Good

```python
def foo():
  if not is_ok:
    return

  do_the_thing()
```

- Avoid unnecessary Django migrations. New pull requestions should contain no more than 1 migration, unless necessary.
 
