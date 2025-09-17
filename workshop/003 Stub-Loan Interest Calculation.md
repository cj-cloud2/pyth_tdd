
# Assignment: Loan Interest Calculation using TDD \& Stubs (Step-by-Step Evolution)

**Domain:** Finance
**Difficulty:** Level 3
**Core Concept:** Using *stubs* with TDD to isolate service dependencies
**Language:** Python

***

## Scenario

Build `calculate_loan_interest(loan_id, get_loan_data)` to:

- Fetch *loan amount* and *interest rate* from a data source (**stubbed** in tests).
- Calculate and return the **interest amount** (simple interest for one year).
- If loan isn’t found, return `None`.

We'll evolve our solution through **3 TDD passes**, adding *one test case* per pass.

***

# Pass 1 – First Test Case

## Step 1 – RED

Start with a single test for loan holder **Tyler**.

**`test_finance.py`**

```python
import unittest

# STUB simulating external DB/API
def stub_get_loan_data(loan_id):
    if loan_id == 501:
        return {"borrower": "Tyler", "amount": 100000, "interest_rate": 0.05}
    return None

class TestFinance(unittest.TestCase):
    def test_interest_basic(self):
        interest = calculate_loan_interest(501, stub_get_loan_data)
        self.assertEqual(interest, 5000)  # 100000 * 5%

if __name__ == "__main__":
    unittest.main()
```

If you run this:

```
NameError: name 'calculate_loan_interest' is not defined
```

**RED!**

***

## Step 2 – GREEN

Implement the function.

**`finance.py`**

```python
def calculate_loan_interest(loan_id, get_loan_data):
    loan = get_loan_data(loan_id)
    if loan is None:
        return None
    amount = loan["amount"]
    rate = loan["interest_rate"]
    interest = amount * rate
    return int(interest)
```

Run tests → **PASS** 🎉

***

## Step 3 – REFACTOR

Clean up and add descriptive comments.

```python
def calculate_loan_interest(loan_id, get_loan_data):
    """
    Calculate simple interest for one year on a given loan.

    Args:
        loan_id (int): Unique identifier for the loan.
        get_loan_data (function): Function to fetch loan details.

    Returns:
        int: Interest amount for one year, rounded down.
        None: If loan not found.
    """
    loan = get_loan_data(loan_id)
    if loan is None:
        return None
    principal = loan["amount"]
    interest_rate = loan["interest_rate"]
    interest_amount = principal * interest_rate
    return int(interest_amount)
```

✅ **Pass 1 done**.

***

# Pass 2 – Add Second Loan

## Step 1 – RED

Add a second test case for loan holder **Linda**.

**Updated `test_finance.py`**

```python
import unittest

def stub_get_loan_data(loan_id):
    if loan_id == 501:
        return {"borrower": "Tyler", "amount": 100000, "interest_rate": 0.05}
    elif loan_id == 602:
        return {"borrower": "Linda", "amount": 200000, "interest_rate": 0.04}
    return None

class TestFinance(unittest.TestCase):
    def test_interest_basic(self):
        interest = calculate_loan_interest(501, stub_get_loan_data)
        self.assertEqual(interest, 5000)

    def test_interest_lower_rate(self):
        interest = calculate_loan_interest(602, stub_get_loan_data)
        self.assertEqual(interest, 8000)  # 200000 * 4%

if __name__ == "__main__":
    unittest.main()
```

If function was hardcoded earlier, this will fail. Otherwise, passes.

***

## Step 2 – GREEN

Make sure the function is generalized (already done).

***

## Step 3 – REFACTOR

Add validation for loan data and types.

```python
def calculate_loan_interest(loan_id, get_loan_data):
    loan = get_loan_data(loan_id)
    if not loan or "amount" not in loan or "interest_rate" not in loan:
        return None
    principal = loan["amount"]
    rate = loan["interest_rate"]
    if not isinstance(principal, (int, float)) or not isinstance(rate, (int, float)):
        raise ValueError("Amount and interest rate must be numbers")
    interest_amount = principal * rate
    return int(interest_amount)
```

✅ **Pass 2 done**.

***

# Pass 3 – Handle Not Found Loan

## Step 1 – RED

Add a test for loan ID that doesn’t exist.

**Final `test_finance.py`**

```python
import unittest

def stub_get_loan_data(loan_id):
    if loan_id == 501:
        return {"borrower": "Tyler", "amount": 100000, "interest_rate": 0.05}
    elif loan_id == 602:
        return {"borrower": "Linda", "amount": 200000, "interest_rate": 0.04}
    elif loan_id == 703:
        return {"borrower": "Patricia", "amount": 150000, "interest_rate": 0.045}
    return None

class TestFinance(unittest.TestCase):
    def test_interest_basic(self):
        interest = calculate_loan_interest(501, stub_get_loan_data)
        self.assertEqual(interest, 5000)

    def test_interest_lower_rate(self):
        interest = calculate_loan_interest(602, stub_get_loan_data)
        self.assertEqual(interest, 8000)

    def test_interest_not_found(self):
        interest = calculate_loan_interest(999, stub_get_loan_data)
        self.assertIsNone(interest)

if __name__ == "__main__":
    unittest.main()
```

Will pass if missing loan handled properly.

***

## Step 2 – GREEN

No change needed if already handling missing loans.

***

## Step 3 – Final REFACTOR

Add complete docstring and validations.

**Final `finance.py`**

```python
def calculate_loan_interest(loan_id, get_loan_data):
    """
    Calculate simple interest (1 year) on a loan by fetching loan data.

    Args:
        loan_id (int): Unique ID of the loan.
        get_loan_data (function): Function that returns a dict containing:
                                  "amount" (principal float/int),
                                  "interest_rate" (float).

    Returns:
        int: Interest amount rounded down to nearest integer.
        None: If loan not found or required data missing.
    """
    loan = get_loan_data(loan_id)
    if not loan or "amount" not in loan or "interest_rate" not in loan:
        return None

    principal = loan["amount"]
    rate = loan["interest_rate"]

    if not isinstance(principal, (int, float)) or not isinstance(rate, (int, float)):
        raise ValueError("Loan amount and interest rate must be numeric")

    interest = principal * rate
    return int(interest)
```

✅ **Pass 3 done — production ready.**

***

## Summary 

1. Start with one test case (RED) → implement code (GREEN) → clean up (REFACTOR).
2. Add more test cases incrementally and refactor.
3. Always use stubs to isolate your logic from external data sources.
4. Make sure to handle missing or malformed data gracefully.

