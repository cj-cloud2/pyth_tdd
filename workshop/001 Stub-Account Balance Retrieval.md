# Assignment: Account Balance Retrieval using TDD \& Stub (Step-by-Step Evolution)

**Domain:** Banking
**Difficulty:** Level 3
**Core Concept:** Using *stubs* with TDD to isolate service dependencies
**Language:** Python

***

## Scenario

Build `get_current_balance(account_id, get_account_data)` to:

- Fetch *account’s balance* from a data source (**stubbed** in tests).
- Return balance as integer.
- If account isn’t found, return `None`.

We'll evolve our solution through **3 TDD passes**, adding *one test case* per pass.

***

# Pass 1 – First Test Case

## Step 1 – RED

Start with a single test for account owner **Mary**.

**`test_banking.py`**

```python
import unittest

# STUB simulating external DB/API
def stub_get_account_data(account_id):
    if account_id == 111:
        return {"name": "Mary", "balance": 12000}
    return None

class TestBanking(unittest.TestCase):
    def test_balance_basic(self):
        bal = get_current_balance(111, stub_get_account_data)
        self.assertEqual(bal, 12000)  # Expect: 12000

if __name__ == "__main__":
    unittest.main()
```

If you run this:

```
NameError: name 'get_current_balance' is not defined
```

**RED!**

***

## Step 2 – GREEN

Implement the function.

**`banking.py`**

```python
def get_current_balance(account_id, get_account_data):
    data = get_account_data(account_id)
    if data is None:
        return None
    return int(data["balance"])
```

Run tests → **PASS** 🎉

***

## Step 3 – REFACTOR

Clean up and add reasoned names.

```python
def get_current_balance(account_id, get_account_data):
    """Return current account balance, or None if not found."""
    details = get_account_data(account_id)
    if details is None:
        return None
    balance = details.get("balance")
    return int(balance)
```

✅ **Pass 1 done**.

***

# Pass 2 – Add Second Account

## Step 1 – RED

Add a second test case for **Patricia**.

**Updated `test_banking.py`**

```python
import unittest

def stub_get_account_data(account_id):
    if account_id == 111:
        return {"name": "Mary", "balance": 12000}
    elif account_id == 222:
        return {"name": "Patricia", "balance": 24750}
    return None

class TestBanking(unittest.TestCase):
    def test_balance_basic(self):
        bal = get_current_balance(111, stub_get_account_data)
        self.assertEqual(bal, 12000)

    def test_balance_large(self):
        bal = get_current_balance(222, stub_get_account_data)
        self.assertEqual(bal, 24750)

if __name__ == "__main__":
    unittest.main()
```

If function is *hardcoded* for Mary, test will fail. If generalized, both pass.

***

## Step 2 – GREEN

Generalize the function (if needed), already done in previous step.

***

## Step 3 – REFACTOR

Add validation for missing `balance` key.

```python
def get_current_balance(account_id, get_account_data):
    """Return current account balance, or None if not found or invalid data."""
    details = get_account_data(account_id)
    if not details or "balance" not in details:
        return None
    balance = details["balance"]
    if not isinstance(balance, (int, float)):
        raise ValueError("Balance must be numeric")
    return int(balance)
```

✅ **Pass 2 done**.

***

# Pass 3 – Handle Not Found Account

## Step 1 – RED

Add a test for an account that doesn’t exist.

**Final `test_banking.py`**

```python
import unittest

def stub_get_account_data(account_id):
    if account_id == 111:
        return {"name": "Mary", "balance": 12000}
    elif account_id == 222:
        return {"name": "Patricia", "balance": 24750}
    elif account_id == 333:
        return {"name": "Jennifer", "balance": 8000}
    return None

class TestBanking(unittest.TestCase):
    def test_balance_basic(self):
        bal = get_current_balance(111, stub_get_account_data)
        self.assertEqual(bal, 12000)

    def test_balance_large(self):
        bal = get_current_balance(222, stub_get_account_data)
        self.assertEqual(bal, 24750)

    def test_balance_not_found(self):
        bal = get_current_balance(999, stub_get_account_data)
        self.assertIsNone(bal)

if __name__ == "__main__":
    unittest.main()
```

If not handled before, this will fail. If handled, passes.

***

## Step 2 – GREEN

If code already handles missing accounts, it passes.

***

## Step 3 – Final REFACTOR

Add robust docstring and validation.

**Final `banking.py`**

```python
def get_current_balance(account_id, get_account_data):
    """
    Return the current balance for a given account.

    Args:
        account_id (int): Unique account identifier.
        get_account_data (function): Stub/service for fetching account data.

    Returns:
        int: Account balance, rounded if float.
        None: If account not found or invalid data.
    """
    details = get_account_data(account_id)
    if not details or "balance" not in details:
        return None
    balance = details["balance"]
    if not isinstance(balance, (int, float)):
        raise ValueError("Balance must be a number")
    return int(balance)
```

✅ **Pass 3 done — production ready.**

***

## Summary

1. Add a test for one scenario (RED). Make it pass (GREEN). Clean up (REFACTOR).
2. Add another scenario, make sure all tests pass, refactor.
3. Cover negative cases (not found), ensure robust error handling.
4. Use **stubs** to isolate logic from infrastructure.
5. Copy–paste each code block stepwise for effective learning!

