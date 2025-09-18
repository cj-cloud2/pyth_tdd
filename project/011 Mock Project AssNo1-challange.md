## **Assignment 1: Basic Loan Application Intake**

### **1) Complete Assignment Description**

**Service goal:**
Accept and validate raw loan applications.

**Test Cases:**

1. Validate application with all required fields present.
2. Reject application missing mandatory field(s).
3. Ensure newly accepted applications are persisted (mock repository).

**Service class after this step:**
Handles intake and simple validation, persists loan applications.

***

***

## **📂 Project Structure for Assignment 1**

```
loan_app/
│
├── loan_service/
│   ├── __init__.py
│   ├── models.py
│   ├── repository.py
│   └── service.py
│
├── tests/
│   ├── __init__.py
│   └── test_service.py
│
└── main.py   # (optional entry point for manual runs)
```


## **4️⃣ tests/test_service.py**

```python
import unittest
from unittest.mock import Mock
from loan_service.service import LoanProcessingService
from loan_service.models import LoanApplication
from loan_service.repository import LoanRepository


class LoanProcessingServiceTest(unittest.TestCase):
    
    def setUp(self):
        # Create a mock repository for testing without a real database
        # Initialize LoanProcessingService with the mock repository


    def test_validate_application_with_all_required_fields(self):
        # Create a loan application object with all required valid details
        # Call the service method to accept the application
        # Assert that the application was accepted (True returned)
        # Verify that the repository's save method was called with the application


    def test_reject_application_missing_mandatory_fields(self):
        # Create a loan application object missing the applicant name
        # Call the service method to accept the application
        # Assert that the application was rejected (False returned)
        # Verify that save was NOT called


    def test_persist_newly_accepted_applications(self):
        # Create a valid loan application with correct details
        # Call the service method to accept the application
        # Assert that the application was accepted (True)
        # Verify that the repository save method was called once with the application


if __name__ == "__main__":
    # Run the unit tests
```


***

## **1️⃣ loan_service/models.py**

```python
# ========================
# Assignment 1 — New Code
# ========================


class LoanApplication:
    """Represents a customer's loan application."""
    def __init__(self, applicant_name: str, amount: float, term_months: int):
        # Store provided applicant name, requested amount, and loan term in months
```


***

## **2️⃣ loan_service/repository.py**

```python
# ========================
# Assignment 1 — New Code
# ========================


from .models import LoanApplication


class LoanRepository:
    """Interface for saving loan applications (to be mocked)."""
    def save(self, application: LoanApplication):
        # This is an abstract method that raises an error if not overridden in a subclass
```


***

## **3️⃣ loan_service/service.py**

```python
# ========================
# Assignment 1 — New Code
# ========================


from .models import LoanApplication
from .repository import LoanRepository


class LoanProcessingService:
    """Handles basic loan intake and validation (Assignment 1)."""
    
    def __init__(self, repository: LoanRepository):
        # Store the loan repository dependency for later use


    def accept_application(self, application: LoanApplication) -> bool:
        """
        Accept and validate a loan application.
        Ensure required fields are filled and add persistence.
        """
        # Check if the provided application is valid using internal validation
        # If invalid, return False to indicate rejection
        # If valid, save the application to the repository
        # Return True to indicate acceptance


    def _is_valid(self, application: LoanApplication) -> bool:
        """Basic field validation."""
        # Check if applicant name is provided (non-empty)
        # Check that the loan amount is a positive number
        # Check that the term in months is a positive integer
        # If all checks pass, return True; otherwise return False
```


***

***

## **5️⃣ Optional: main.py**

```python
from loan_service.models import LoanApplication
from loan_service.repository import LoanRepository
from loan_service.service import LoanProcessingService


class InMemoryLoanRepository(LoanRepository):
    def __init__(self):
        # Initialize an internal list to store loan applications


    def save(self, application: LoanApplication):
        # Append the loan application to internal storage
        # Print confirmation that the application has been saved


if __name__ == "__main__":
    # Create in-memory repository instance
    # Initialize LoanProcessingService with repository
    # Create a sample loan application
    # Submit the application to the service
    # Print whether the application was accepted
```


***

## **Run Tests**

From the **project root**:

```bash
python -m unittest discover tests
```


***
