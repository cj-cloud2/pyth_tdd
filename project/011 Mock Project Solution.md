## 📂 Final `loan_app` Project Structure

```
loan_app/
│
├── loan_service/
│   ├── __init__.py
│   ├── models.py
│   ├── repository.py
│   ├── credit_service.py
│   ├── document_service.py
│   ├── notification_service.py
│   ├── audit_service.py
│   └── service.py
│
├── tests/
│   ├── __init__.py
│   ├── test_service.py
│   ├── test_service_credit.py
│   ├── test_service_documents.py
│   └── test_service_notification_audit.py
│
└── main.py
```


***

### **loan_service/__init__.py**

```python
# Package initializer for loan_service
```


***

### **loan_service/models.py**

```python
class LoanApplication:
    """Represents a customer's loan application."""
    def __init__(self, applicant_name: str, amount: float, term_months: int):
        self.applicant_name = applicant_name
        self.amount = amount
        self.term_months = term_months
```


***

### **loan_service/repository.py**

```python
from .models import LoanApplication

class LoanRepository:
    """Interface for saving loan applications (to be mocked)."""
    def save(self, application: LoanApplication):
        raise NotImplementedError("Subclasses or mocks must implement this method.")
```


***

### **loan_service/credit_service.py**

```python
class CreditScoreService:
    """Interface to fetch applicant's credit score."""
    def get_credit_score(self, applicant_name: str) -> int:
        raise NotImplementedError("Implement in subclass or mock for testing")
```


***

### **loan_service/document_service.py**

```python
class DocumentService:
    """Interface to verify applicant documents."""
    def verify_documents(self, applicant_name: str) -> dict:
        """
        Returns:
        {
            "status": "valid" | "missing" | "invalid",
            "details": "Reason or info"
        }
        """
        raise NotImplementedError("Implement in subclass or mock for testing")
```


***

### **loan_service/notification_service.py**

```python
class NotificationService:
    """Interface to send notifications to applicants."""
    def send(self, applicant_name: str, message: str):
        raise NotImplementedError("Implement in subclass or mock for testing")
```


***

### **loan_service/audit_service.py**

```python
class AuditService:
    """Interface to record audit logs for loan application processing."""
    def log(self, applicant_name: str, outcome: str, details: str = ""):
        raise NotImplementedError("Implement in subclass or mock for testing")
```


***

### **loan_service/service.py**

*(Final version — Assignments 1–4 combined)*

```python
from .models import LoanApplication
from .repository import LoanRepository
from .credit_service import CreditScoreService
from .document_service import DocumentService
from .notification_service import NotificationService
from .audit_service import AuditService

class LoanProcessingService:
    """
    Loan processing service — Assignment 1, 2, 3 & 4
    """

    def __init__(self, repository: LoanRepository,
                 credit_service: CreditScoreService = None,
                 document_service: DocumentService = None,
                 notification_service: NotificationService = None,
                 audit_service: AuditService = None):
        self.repository = repository
        self.credit_service = credit_service
        self.document_service = document_service
        self.notification_service = notification_service
        self.audit_service = audit_service

    def accept_application(self, application: LoanApplication):
        """
        Process a loan application end-to-end.
        Returns:
            True  → Approved
            "awaiting_documents" → Pending docs
            False → Rejected
        """
        outcome = None
        reason = ""

        # Assignment 1 — Basic field validation
        if not self._is_valid(application):
            outcome, reason = "rejected", "Missing mandatory fields"
            self._notify_and_audit(application.applicant_name, outcome, reason)
            return False

        # Assignment 2 — Credit score check
        if self.credit_service:
            try:
                score = self.credit_service.get_credit_score(application.applicant_name)
            except Exception:
                outcome, reason = "rejected", "Credit service failure"
                self._notify_and_audit(application.applicant_name, outcome, reason)
                return False
            if score < 600:
                outcome, reason = "rejected", "Low credit score"
                self._notify_and_audit(application.applicant_name, outcome, reason)
                return False

        # Assignment 3 — Document verification
        if self.document_service:
            doc_result = self.document_service.verify_documents(application.applicant_name)
            if doc_result["status"] == "missing":
                outcome, reason = "pending", "Awaiting documents"
                self._notify_and_audit(application.applicant_name, outcome, reason)
                return "awaiting_documents"
            elif doc_result["status"] == "invalid":
                outcome, reason = "rejected", "Invalid documents"
                self._notify_and_audit(application.applicant_name, outcome, reason)
                return False

        # Passed all checks → approve
        self.repository.save(application)
        outcome, reason = "approved", "Application approved successfully"
        self._notify_and_audit(application.applicant_name, outcome, reason)
        return True

    def _is_valid(self, application: LoanApplication) -> bool:
        if not application.applicant_name:
            return False
        if application.amount is None or application.amount <= 0:
            return False
        if application.term_months is None or application.term_months <= 0:
            return False
        return True

    def _notify_and_audit(self, applicant_name: str, outcome: str, details: str):
        """Send notification and write to audit trail."""
        if self.notification_service:
            self.notification_service.send(applicant_name, f"Your application is {outcome}. {details}")
        if self.audit_service:
            self.audit_service.log(applicant_name, outcome, details)
```


***

## **Tests**

We keep all the older test files **intact** and add the new one.

### **tests/__init__.py**

```python
# Package initializer for tests
```


***

### Existing tests:

- **Assignment 1:** `tests/test_service.py`
- **Assignment 2:** `tests/test_service_credit.py`
- **Assignment 3:** `tests/test_service_documents.py`

*(These are already defined from our previous steps and remain unchanged.)*

***

### **tests/test_service_notification_audit.py**

*(Assignment 4 tests)*

```python
import unittest
from unittest.mock import Mock
from loan_service.service import LoanProcessingService
from loan_service.models import LoanApplication
from loan_service.repository import LoanRepository
from loan_service.credit_service import CreditScoreService
from loan_service.document_service import DocumentService
from loan_service.notification_service import NotificationService
from loan_service.audit_service import AuditService

class LoanProcessingServiceNotificationAuditTest(unittest.TestCase):

    def setUp(self):
        self.mock_repo = Mock(spec=LoanRepository)
        self.mock_credit = Mock(spec=CreditScoreService)
        self.mock_docs = Mock(spec=DocumentService)
        self.mock_notify = Mock(spec=NotificationService)
        self.mock_audit = Mock(spec=AuditService)

        self.mock_credit.get_credit_score.return_value = 700
        self.mock_docs.verify_documents.return_value = {"status": "valid", "details": ""}

        self.service = LoanProcessingService(
            repository=self.mock_repo,
            credit_service=self.mock_credit,
            document_service=self.mock_docs,
            notification_service=self.mock_notify,
            audit_service=self.mock_audit
        )

    def test_send_approval_notification_and_audit(self):
        app = LoanApplication("John Approved", 5000, 12)
        result = self.service.accept_application(app)
        self.assertTrue(result)
        self.mock_notify.send.assert_called_once()
        self.mock_audit.log.assert_called_once()
        self.assertEqual(self.mock_audit.log.call_args[0][1], "approved")

    def test_send_rejection_notification_and_audit(self):
        app = LoanApplication("Jane LowScore", 8000, 24)
        self.mock_credit.get_credit_score.return_value = 550
        result = self.service.accept_application(app)
        self.assertFalse(result)
        self.mock_notify.send.assert_called_once()
        self.mock_audit.log.assert_called_once()
        self.assertEqual(self.mock_audit.log.call_args[0][1], "rejected")

    def test_audit_for_pending_due_to_missing_docs(self):
        app = LoanApplication("Tom MissingDocs", 6000, 18)
        self.mock_docs.verify_documents.return_value = {"status": "missing", "details": "ID Proof missing"}
        result = self.service.accept_application(app)
        self.assertEqual(result, "awaiting_documents")
        self.mock_notify.send.assert_called_once()
        self.mock_audit.log.assert_called_once()
        self.assertEqual(self.mock_audit.log.call_args[0][1], "pending")

if __name__ == "__main__":
    unittest.main()
```


***

### **main.py** (Optional manual run)

```python
from loan_service.models import LoanApplication
from loan_service.repository import LoanRepository
from loan_service.service import LoanProcessingService
from loan_service.credit_service import CreditScoreService
from loan_service.document_service import DocumentService
from loan_service.notification_service import NotificationService
from loan_service.audit_service import AuditService

class InMemoryLoanRepository(LoanRepository):
    def __init__(self):
        self.db = []
    def save(self, application: LoanApplication):
        self.db.append(application)
        print(f"Saved application for {application.applicant_name}")

class DummyCreditScoreService(CreditScoreService):
    def get_credit_score(self, applicant_name: str) -> int:
        return 700

class DummyDocumentService(DocumentService):
    def verify_documents(self, applicant_name: str) -> dict:
        return {"status": "valid", "details": ""}

class ConsoleNotificationService(NotificationService):
    def send(self, applicant_name: str, message: str):
        print(f"Notification to {applicant_name}: {message}")

class ConsoleAuditService(AuditService):
    def log(self, applicant_name: str, outcome: str, details: str = ""):
        print(f"AUDIT: {applicant_name} - {outcome} - {details}")

if __name__ == "__main__":
    repo = InMemoryLoanRepository()
    credit_service = DummyCreditScoreService()
    doc_service = DummyDocumentService()
    notify_service = ConsoleNotificationService()
    audit_service = ConsoleAuditService()

    service = LoanProcessingService(repo, credit_service, doc_service, notify_service, audit_service)
    app = LoanApplication("John Doe", 5000, 12)
    service.accept_application(app)
```


***

## 📦 How to Create ZIP Locally

1. Create the folder structure above.
2. Paste the code into the correct files.
3. From the directory containing `loan_app/`, run:

```bash
zip -r loan_app_assignments_1_to_4.zip loan_app
```

4. Run **all tests**:

```bash
python -m unittest discover loan_app/tests
```


***
