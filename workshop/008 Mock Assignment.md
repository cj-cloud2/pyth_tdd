
## **TDD Cycle 1 — RED → GREEN → REFACTOR for test 1**

### Step 1 — RED (Write the first failing test)

```python
# test_s3.py
import unittest
from unittest.mock import MagicMock

from app.devops_script import create_s3_bucket  # Not implemented yet

class TestS3BucketCreation(unittest.TestCase):
    def test_create_bucket_calls_once_with_correct_args(self):
        mock_s3_client = MagicMock()
        mock_s3_client.create_bucket.return_value = {"ResponseMetadata": {"HTTPStatusCode": 200}}
        
        bucket_name = "my-test-bucket"
        response = create_s3_bucket(mock_s3_client, bucket_name)  # Will fail (function missing)
        
        mock_s3_client.create_bucket.assert_called_once_with(Bucket=bucket_name)
        self.assertEqual(response["ResponseMetadata"]["HTTPStatusCode"], 200)

if __name__ == "__main__":
    unittest.main()
```

⬆️ **Run now → FAIL** (`ImportError` or `NameError` because `create_s3_bucket` not implemented).

***

### Step 2 — GREEN (Make it pass minimally)

```python
# devops_script.py
def create_s3_bucket(s3_client, bucket_name):
    return s3_client.create_bucket(Bucket=bucket_name)
```

⬆️ **Run → PASS** ✅

***

### Step 3 — REFACTOR (Clean without breaking)

No big change needed; the function is already minimal.
**End of first TDD cycle.**

***

## **TDD Cycle 2 — RED → GREEN → REFACTOR for test 2**

### Step 1 — RED (Add failing test for invalid bucket name case)

```python
# test_s3.py (continuing same class)
    def test_create_bucket_raises_for_invalid_name(self):
        mock_s3_client = MagicMock()
        mock_s3_client.create_bucket.side_effect = Exception("Invalid bucket name")
        
        with self.assertRaises(Exception) as context:
            create_s3_bucket(mock_s3_client, "Invalid_Bucket_Name!")
        
        self.assertIn("Invalid bucket name", str(context.exception))
        mock_s3_client.create_bucket.assert_called_once_with(Bucket="Invalid_Bucket_Name!")
```

⬆️ **Run now → FAIL**?
Actually, with the current function, this will **already pass** because the exception is propagated. But in TDD, it's okay if it passes immediately — that means the code already meets the new requirement.

***

### Step 2 — GREEN (Already Green)

No code changes needed; test passes.

***

### Step 3 — REFACTOR

We might decide to wrap errors for better DevOps logging later,
but for now, no change → tests still pass.

***

## **TDD Cycle 3 — RED → GREEN → REFACTOR for test 3**

### Step 1 — RED (Multiple bucket creation call verification)

```python
# test_s3.py
    def test_create_bucket_called_multiple_times(self):
        mock_s3_client = MagicMock()
        mock_s3_client.create_bucket.return_value = {"ResponseMetadata": {"HTTPStatusCode": 200}}
        
        buckets = ["bucket-one", "bucket-two", "bucket-three"]
        for b in buckets:
            create_s3_bucket(mock_s3_client, b)
        
        self.assertEqual(mock_s3_client.create_bucket.call_count, 3)
        mock_s3_client.create_bucket.assert_any_call(Bucket="bucket-one")
        mock_s3_client.create_bucket.assert_any_call(Bucket="bucket-two")
        mock_s3_client.create_bucket.assert_any_call(Bucket="bucket-three")
```

⬆️ **Run now → PASS immediately** (because function already loops correctly — well, our loop is in the test).

***

### Step 2 — GREEN

Nothing extra; already passing.

***

### Step 3 — REFACTOR

Maybe extract bucket creation into a helper if logic grows later — but for this case, code is small enough to keep.

***

## **Final Code after all 3 TDD Cycles**

### devops_script.py

```python
def create_s3_bucket(s3_client, bucket_name):
    """Creates an S3 bucket using provided boto3 S3 client."""
    return s3_client.create_bucket(Bucket=bucket_name)
```


### test_s3.py

```python
import unittest
from unittest.mock import MagicMock
from app.devops_script import create_s3_bucket

class TestS3BucketCreation(unittest.TestCase):

    def test_create_bucket_calls_once_with_correct_args(self):
        mock_s3_client = MagicMock()
        mock_s3_client.create_bucket.return_value = {"ResponseMetadata": {"HTTPStatusCode": 200}}
        bucket_name = "my-test-bucket"

        response = create_s3_bucket(mock_s3_client, bucket_name)

        mock_s3_client.create_bucket.assert_called_once_with(Bucket=bucket_name)
        self.assertEqual(response["ResponseMetadata"]["HTTPStatusCode"], 200)

    def test_create_bucket_raises_for_invalid_name(self):
        mock_s3_client = MagicMock()
        mock_s3_client.create_bucket.side_effect = Exception("Invalid bucket name")

        with self.assertRaises(Exception) as context:
            create_s3_bucket(mock_s3_client, "Invalid_Bucket_Name!")

        self.assertIn("Invalid bucket name", str(context.exception))
        mock_s3_client.create_bucket.assert_called_once_with(Bucket="Invalid_Bucket_Name!")

    def test_create_bucket_called_multiple_times(self):
        mock_s3_client = MagicMock()
        mock_s3_client.create_bucket.return_value = {"ResponseMetadata": {"HTTPStatusCode": 200}}

        buckets = ["bucket-one", "bucket-two", "bucket-three"]
        for b in buckets:
            create_s3_bucket(mock_s3_client, b)

        self.assertEqual(mock_s3_client.create_bucket.call_count, 3)
        mock_s3_client.create_bucket.assert_any_call(Bucket="bucket-one")
        mock_s3_client.create_bucket.assert_any_call(Bucket="bucket-two")
        mock_s3_client.create_bucket.assert_any_call(Bucket="bucket-three")

if __name__ == "__main__":
    unittest.main()
```


***