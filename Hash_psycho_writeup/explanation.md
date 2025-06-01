# BYUCTF 2025 — Hash Psycho Write-up

> A hash collision CTF challenge based on Python’s internal hashing of integers.

---

## Challenge Overview

| Field         | Value                                           |
|---------------|--------------------------------------------------|
| Name          | Hash Psycho                                     |
| Category      | Misc / Exploitation / Python Internals          |
| Points        | 50                                               |
| Server        | `nc psycho.chal.cyberjousting.com 1353`         |
| Flag Format   | `byuctf{...}`                                    |

This challenge mimics a corporate onboarding scenario where users are assigned IDs, and authorization is handled poorly—via Python’s built-in `hash()` function. Exploiting a known property of Python’s integer hashing, we’re able to impersonate the admin and retrieve the flag.

---

## Step-by-Step Solution

### 1. The Flow

The Python script behind the server works like this:

code:
```python
class User:
    def __init__(self, username, id):
        self.username = username
        self.id = id

    def __hash__(self):
        return hash(self.id)

ADMIN = User("admin", 1337)
...
YOURUSER = User(name, user_input_id)
...
if choice == 2 and hash(YOURUSER) == hash(ADMIN):
    print(FLAG)
```
The access check hinges solely on hash(id), so a hash‑collision on 1337 is enough to impersonate the admin.

### 2. How hash(int) Works in CPython
For non‑negative integers (except -1), CPython implements hash(n)=nmod(2^61−1)

```text
>>> MOD = 2**61 - 1
>>> hash(1337)
1337
>>> x = 1337 + MOD
>>> hash(x) == hash(1337)
True
```
Hence every
x = 1337 + k·(2^61 − 1) ( k ∈ ℤ ) collides with the admin.

### 3.Exploitation Steps

Connect: nc psycho.chal.cyberjousting.com 1353
Name: anything (e.g., pwner)
ID: 2305843009213695288
Door: option 2 (admin’s office)

```python
$ nc psycho… 1353
Welcome to onboarding! …
2305843009213695288
*Choose a door*
2
byuctf{who_kn3w_h4sh_w3snt_h4sh}
```

### Flag:

byuctf{who_kn3w_h4sh_w3snt_h4sh}


