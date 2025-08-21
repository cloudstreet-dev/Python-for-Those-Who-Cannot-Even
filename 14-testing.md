# Chapter 14: Testing (Because Your Code Definitely Has Bugs)

*"Testing is like going to the gym: everyone knows they should do it, most people don't, and those who do won't shut up about it."* - Developer with 15% code coverage

Python has two testing frameworks: unittest (which feels like Java had a baby with XML) and pytest (which everyone actually uses). Let's explore both, but mostly pytest, because life is too short for self.assertEqual.

## unittest: Java Programmers Will Feel at Home

```python
import unittest

# The Java-inspired way
class TestStringMethods(unittest.TestCase):
    
    def setUp(self):
        """Runs before each test method"""
        self.test_string = "hello world"
    
    def tearDown(self):
        """Runs after each test method"""
        # Clean up resources
        pass
    
    def test_upper(self):
        self.assertEqual(self.test_string.upper(), "HELLO WORLD")
    
    def test_split(self):
        s = "hello world"
        self.assertEqual(s.split(), ['hello', 'world'])
        # Check that split fails with bad separator
        with self.assertRaises(TypeError):
            s.split(2)
    
    def test_isupper(self):
        self.assertTrue('HELLO'.isupper())
        self.assertFalse('Hello'.isupper())

# Running tests
if __name__ == '__main__':
    unittest.main()

# Or from command line:
# python -m unittest test_module.py
# python -m unittest discover  # Find all tests

# The assertions nobody remembers
class TestAssertions(unittest.TestCase):
    def test_all_assertions(self):
        # Equality
        self.assertEqual(1, 1)
        self.assertNotEqual(1, 2)
        
        # Truth
        self.assertTrue(True)
        self.assertFalse(False)
        
        # Identity
        self.assertIs(None, None)
        self.assertIsNot([], [])
        
        # Membership
        self.assertIn(1, [1, 2, 3])
        self.assertNotIn(4, [1, 2, 3])
        
        # Comparison
        self.assertGreater(2, 1)
        self.assertLess(1, 2)
        self.assertGreaterEqual(2, 2)
        self.assertLessEqual(2, 2)
        
        # Types
        self.assertIsInstance("hello", str)
        self.assertNotIsInstance("hello", int)
        
        # Regex
        self.assertRegex("hello", r"h.llo")
        
        # Containers
        self.assertCountEqual([1, 2, 2], [2, 1, 2])  # Same elements
        self.assertListEqual([1, 2], [1, 2])
        self.assertDictEqual({"a": 1}, {"a": 1})
        self.assertSetEqual({1, 2}, {2, 1})
        
        # Almost equal (for floats)
        self.assertAlmostEqual(1.0000001, 1.0000002, places=5)
        
        # Why are there so many?!
```

## pytest: The Actually Good Testing Framework

```python
# pytest: Testing for humans
import pytest

# No classes needed!
def test_upper():
    assert "hello".upper() == "HELLO"

def test_split():
    assert "hello world".split() == ['hello', 'world']

def test_isupper():
    assert 'HELLO'.isupper()
    assert not 'Hello'.isupper()

# That's it. Just assert.
# pytest introspects assertions and provides detailed failure messages

# Run with:
# pytest
# pytest test_file.py
# pytest test_file.py::test_function
# pytest -v  # Verbose
# pytest -s  # Show print statements
# pytest -x  # Stop on first failure
# pytest --pdb  # Drop into debugger on failure

# Fixtures: pytest's killer feature
@pytest.fixture
def sample_data():
    """This runs before tests that use it"""
    return {"name": "Alice", "age": 30}

def test_with_fixture(sample_data):
    assert sample_data["name"] == "Alice"

# Fixture scope
@pytest.fixture(scope="module")  # Run once per module
def database_connection():
    conn = create_connection()
    yield conn  # This is what the test gets
    conn.close()  # Cleanup after test

# Parametrized tests (test multiple inputs)
@pytest.mark.parametrize("input,expected", [
    ("hello", "HELLO"),
    ("world", "WORLD"),
    ("Python", "PYTHON"),
])
def test_upper_parametrized(input, expected):
    assert input.upper() == expected

# Marking tests
@pytest.mark.slow
def test_slow_operation():
    time.sleep(10)
    assert True

@pytest.mark.skip(reason="Not implemented yet")
def test_future_feature():
    assert False

@pytest.mark.skipif(sys.version_info < (3, 10), reason="Requires Python 3.10+")
def test_new_feature():
    assert True

@pytest.mark.xfail(reason="Known bug")
def test_broken_feature():
    assert False  # This "passes" as expected failure

# Run marked tests:
# pytest -m slow  # Run only slow tests
# pytest -m "not slow"  # Skip slow tests
```

## Mocking: Pretending Your Code Works

```python
from unittest.mock import Mock, MagicMock, patch, call

# Basic mock
mock = Mock()
mock.return_value = 42
result = mock()  # Returns 42

# Checking calls
mock.assert_called()
mock.assert_called_once()
mock.assert_called_with()  # Any arguments
mock.assert_called_once_with(specific_arg=True)

# MagicMock: Mock with magic methods
magic = MagicMock()
magic.__str__.return_value = "mocked string"
str(magic)  # "mocked string"

# Mock attributes
mock.attribute = "value"
mock.method.return_value = "result"

# Side effects
mock = Mock(side_effect=[1, 2, 3])
mock()  # 1
mock()  # 2
mock()  # 3
mock()  # StopIteration

# Exception side effect
mock = Mock(side_effect=ValueError("Oh no!"))
# mock()  # Raises ValueError

# Patching (the real power)
class EmailSender:
    def send(self, to, subject, body):
        # Actually sends email
        pass

def notify_user(user_email):
    sender = EmailSender()
    sender.send(user_email, "Notification", "Hello!")
    return True

# Test without sending real email
@patch('module.EmailSender')
def test_notify_user(mock_email_sender):
    # mock_email_sender is the class
    # mock_email_sender.return_value is the instance
    instance = mock_email_sender.return_value
    
    result = notify_user("test@example.com")
    
    assert result is True
    instance.send.assert_called_once_with(
        "test@example.com", "Notification", "Hello!"
    )

# Patch as context manager
def test_with_context_manager():
    with patch('module.dangerous_function') as mock_func:
        mock_func.return_value = "safe value"
        result = function_that_calls_dangerous()
        assert result == "safe value"

# Patch decorator
@patch('os.remove')
@patch('os.path.exists')
def test_multiple_patches(mock_exists, mock_remove):
    mock_exists.return_value = True
    delete_file("test.txt")
    mock_remove.assert_called_once_with("test.txt")

# Patch object attributes
with patch.object(SomeClass, 'method', return_value=42):
    obj = SomeClass()
    assert obj.method() == 42

# Mock spec (restrict to real methods)
from my_module import RealClass
mock = Mock(spec=RealClass)
# mock.nonexistent_method()  # AttributeError!
```

## Test Organization and Best Practices

```python
# Project structure
my_project/
├── src/
│   └── my_package/
│       ├── __init__.py
│       ├── module1.py
│       └── module2.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py  # Shared fixtures
│   ├── test_module1.py
│   └── test_module2.py
└── pytest.ini  # pytest configuration

# conftest.py: Shared fixtures
import pytest

@pytest.fixture
def client():
    """Test client for API testing"""
    from app import create_app
    app = create_app(testing=True)
    with app.test_client() as client:
        yield client

@pytest.fixture
def database():
    """Test database"""
    db = create_test_db()
    yield db
    db.cleanup()

# pytest.ini: Configuration
[pytest]
testpaths = tests
python_files = test_*.py
python_classes = Test*
python_functions = test_*
addopts = -v --tb=short --strict-markers
markers =
    slow: marks tests as slow
    integration: marks tests as integration tests
    unit: marks tests as unit tests

# Test file organization
class TestUserAuthentication:
    """Group related tests"""
    
    def test_valid_login(self, client):
        response = client.post('/login', data={
            'username': 'test',
            'password': 'secret'
        })
        assert response.status_code == 200
    
    def test_invalid_login(self, client):
        response = client.post('/login', data={
            'username': 'test',
            'password': 'wrong'
        })
        assert response.status_code == 401

# Testing exceptions
def test_division_by_zero():
    with pytest.raises(ZeroDivisionError):
        1 / 0
    
    # Check exception message
    with pytest.raises(ValueError, match=r".*invalid.*"):
        raise ValueError("invalid input")

# Testing warnings
def test_deprecation_warning():
    with pytest.warns(DeprecationWarning):
        old_function()

# Approximate comparisons (for floats)
def test_float_comparison():
    assert 0.1 + 0.2 == pytest.approx(0.3)
    assert [0.1, 0.2] == pytest.approx([0.1, 0.2])
```

## Coverage: Measuring How Much You're Testing

```bash
# Install coverage
pip install pytest-cov

# Run with coverage
pytest --cov=my_package
pytest --cov=my_package --cov-report=html  # HTML report
pytest --cov=my_package --cov-report=term-missing  # Show missing lines

# Coverage configuration (.coveragerc)
[run]
source = src/
omit = 
    */tests/*
    */venv/*
    */__pycache__/*

[report]
exclude_lines =
    pragma: no cover
    def __repr__
    raise AssertionError
    raise NotImplementedError
    if __name__ == .__main__.:

# Coverage comments in code
def my_function():
    if complex_condition():  # pragma: no cover
        # This line won't count against coverage
        handle_rare_case()
```

## Test Doubles: Stubs, Mocks, Fakes, and Spies

```python
# Stub: Provides canned answers
class StubDatabase:
    def get_user(self, id):
        return {"id": id, "name": "Test User"}

# Mock: Records calls and can assert expectations
mock_db = Mock()
mock_db.get_user.return_value = {"id": 1, "name": "Test"}
user = mock_db.get_user(1)
mock_db.get_user.assert_called_with(1)

# Fake: Working implementation but simplified
class FakeDatabase:
    def __init__(self):
        self.users = {}
    
    def add_user(self, user):
        self.users[user['id']] = user
    
    def get_user(self, id):
        return self.users.get(id)

# Spy: Real object with recording
class SpyDatabase:
    def __init__(self, real_db):
        self.real_db = real_db
        self.calls = []
    
    def get_user(self, id):
        self.calls.append(('get_user', id))
        return self.real_db.get_user(id)

# Dummy: Passed around but never used
dummy_logger = Mock()  # Never actually called
```

## Property-Based Testing with Hypothesis

```python
from hypothesis import given, strategies as st

# Traditional test
def test_reverse_traditional():
    assert reverse([1, 2, 3]) == [3, 2, 1]
    assert reverse([]) == []
    assert reverse([1]) == [1]

# Property-based test
@given(st.lists(st.integers()))
def test_reverse_properties(lst):
    reversed_list = reverse(lst)
    
    # Properties that should always be true
    assert len(reversed_list) == len(lst)
    assert reverse(reversed_list) == lst  # Reversing twice gives original
    
    if lst:
        assert reversed_list[0] == lst[-1]
        assert reversed_list[-1] == lst[0]

# Hypothesis generates random test cases
# Including edge cases you didn't think of:
# - Empty lists
# - Single elements
# - Duplicate elements
# - Very large lists
# - Lists with None
# - Lists with negative numbers

@given(st.text())
def test_string_properties(s):
    assert s.upper().lower() == s.lower()
    assert len(s.strip()) <= len(s)

# Custom strategies
user_strategy = st.builds(
    dict,
    name=st.text(min_size=1),
    age=st.integers(min_value=0, max_value=150),
    email=st.emails()
)

@given(user_strategy)
def test_user_validation(user):
    validate_user(user)  # Should not raise
```

## Integration and End-to-End Testing

```python
# Integration test example
class TestAPIIntegration:
    @pytest.fixture(autouse=True)
    def setup(self, database):
        """Setup test data"""
        database.add_user({"id": 1, "name": "Alice"})
        database.add_post({"id": 1, "user_id": 1, "title": "Test"})
    
    def test_get_user_with_posts(self, client, database):
        response = client.get('/users/1')
        assert response.status_code == 200
        
        data = response.json()
        assert data['name'] == 'Alice'
        assert len(data['posts']) == 1
        assert data['posts'][0]['title'] == 'Test'

# End-to-end test with Selenium
from selenium import webdriver
from selenium.webdriver.common.by import By

def test_user_registration_flow():
    driver = webdriver.Chrome()
    try:
        # Navigate to registration page
        driver.get("http://localhost:5000/register")
        
        # Fill out form
        driver.find_element(By.NAME, "username").send_keys("newuser")
        driver.find_element(By.NAME, "email").send_keys("new@example.com")
        driver.find_element(By.NAME, "password").send_keys("secret123")
        
        # Submit
        driver.find_element(By.ID, "submit").click()
        
        # Verify redirect to dashboard
        assert "/dashboard" in driver.current_url
        
        # Verify welcome message
        welcome = driver.find_element(By.CLASS_NAME, "welcome")
        assert "Welcome, newuser" in welcome.text
    finally:
        driver.quit()

# Testing async code
@pytest.mark.asyncio
async def test_async_function():
    result = await async_operation()
    assert result == expected

# Testing with real database (slow but thorough)
@pytest.mark.integration
def test_database_transaction(real_database):
    with real_database.transaction() as tx:
        tx.execute("INSERT INTO users VALUES (?, ?)", (1, "Alice"))
        tx.execute("INSERT INTO posts VALUES (?, ?, ?)", (1, 1, "Title"))
    
    user = real_database.query("SELECT * FROM users WHERE id = ?", (1,))
    assert user['name'] == 'Alice'
```

## Testing Anti-Patterns (What Not to Do)

```python
# Anti-pattern 1: Testing implementation, not behavior
def test_implementation_bad():
    calc = Calculator()
    # Bad: Testing private method
    assert calc._internal_calculate(2, 3) == 5

def test_behavior_good():
    calc = Calculator()
    # Good: Testing public interface
    assert calc.add(2, 3) == 5

# Anti-pattern 2: One test does everything
def test_everything_bad():
    user = create_user("Alice")
    assert user.name == "Alice"
    
    user.age = 30
    assert user.age == 30
    
    user.delete()
    assert user.deleted
    # Too many things in one test!

# Anti-pattern 3: Brittle tests
def test_brittle():
    html = get_page()
    # Bad: Depends on exact HTML structure
    assert '<div class="header"><h1>Title</h1></div>' in html

def test_robust():
    html = get_page()
    # Good: Tests what matters
    assert 'Title' in html

# Anti-pattern 4: Not cleaning up
def test_no_cleanup_bad():
    create_temp_file("test.txt")
    # Oops, file stays after test!

@pytest.fixture
def temp_file():
    path = "test.txt"
    create_temp_file(path)
    yield path
    os.remove(path)  # Cleanup happens automatically

# Anti-pattern 5: Time-dependent tests
def test_time_bad():
    obj = create_object()
    assert obj.created_at == datetime.now()  # Might fail!

def test_time_good():
    with freeze_time("2023-01-01"):
        obj = create_object()
        assert obj.created_at == datetime(2023, 1, 1)
```

## Exercises: Try Not to Cry

1. **Test Refactoring**: Take a test file written with unittest and convert it to pytest. Notice how much cleaner it becomes.

2. **Mock Mastery**: Write tests for a function that sends emails, makes API calls, and writes to a database. Mock all external dependencies.

3. **Coverage Challenge**: Write a function with complex branching logic. Achieve 100% test coverage. Then introduce a bug that passes all tests.

4. **Property Testing**: Use Hypothesis to test a sorting function. Let it find edge cases you didn't think of.

5. **Fixture Factory**: Create a fixture that generates test data with different characteristics based on parameters.

6. **Integration Test**: Write an integration test for a web API that creates a user, logs in, performs an action, and verifies the result.

## Summary: What We've Learned

1. **unittest is verbose** (Java programmers feel at home)
2. **pytest is better** (just use assert)
3. **Fixtures are powerful** (dependency injection for tests)
4. **Mocking prevents side effects** (don't send real emails)
5. **Coverage doesn't mean correctness** (100% coverage, 100% bugs)
6. **Property-based testing finds edge cases** (computers are better at this)
7. **Integration tests are slow but valuable** (test the real thing)
8. **Test behavior, not implementation** (public API only)
9. **Clean up after your tests** (use fixtures)
10. **Testing is hard** (but not testing is harder)

Next chapter: The Conclusion, where we'll reflect on our journey through Python's quirks and question our life choices.

---

*"Writing tests is like flossing: everyone agrees it's important, most people lie about doing it, and when you finally do it, you find problems you wish you hadn't."* - Developer with 47% code coverage