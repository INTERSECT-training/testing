---
title: "Mocking and Isolation"
teaching: 15
exercises: 5
---

## The Problem of External Dependencies

Unit tests are supposed to be fast, deterministic, and isolated. But real-world scientific software frequently has to interact with things outside of its direct control, such as:
* Reading or writing files on a local hard drive.
* Querying an external web API or database.
* Running computationally expensive simulations that take hours.

If a unit test tries to interact with these external systems directly, it is no longer a unit test—it becomes a slow, brittle integration test. If the external weather database goes offline, your test suite fails, even though your local calculations are mathematically flawless.

To solve this, we use **Mocks** (also known as test doubles). A mock is a fake object that mimics the behavior of a real dependency, allowing you to isolate the specific unit of code you want to test.

---

## Example: Mocking a Web API

Consider a function that fetches weather data from an external API to calculate local atmospheric density. We want to test the density math without actually hitting the internet every time we run `pytest`.

Instead of making a real network request, we can use Pytest's built-in `monkeypatch` fixture to intercept the external call and return a pre-configured dummy response.

```python
import requests
from density_calculator import calculate_density_from_api

# 1. The code we want to test
def calculate_density_from_api(city):
    # This calls a slow, external network API
    response = requests.get(f"[https://api.weather-science.org/](https://api.weather-science.org/){city}")
    data = response.json()
    
    # Core mathematical logic we want to verify
    temp = data["temperature"]
    pressure = data["pressure"]
    return (pressure) / (287.05 * (temp + 273.15))


# 2. Our isolated unit test
def test_density_calculation(monkeypatch):
    # We define a dummy response class
    class MockResponse:
        def json(self):
            return {"temperature": 15.0, "pressure": 101325.0}

    # Intercept 'requests.get' and replace it with our dummy response
    monkeypatch.setattr(requests, "get", lambda url: MockResponse())

    # Act
    result = calculate_density_from_api("Geneva")

    # Assert: We can now safely verify our mathematical logic in-memory!
    assert round(result, 4) == 1.2250
```

Because of the mock, this test runs in under a millisecond, requires no internet connection, and will never fail due to API downtime.

---

## When to Mock: Helpful Tool vs. Code Smell

Mocking is a powerful safety valve, but it can easily be overused. Writing highly complicated mocks to test basic logic is a major source of technical debt. 

Use these heuristics to guide your design:

### Scenario A: Good Mocking Targets (The Boundary)
Mocking is highly effective when applied to the **boundaries** of your software system:
* **The Network:** Mocking API requests, database queries, or web sockets.
* **The Filesystem:** Mocking file creation, read/write loops, or system hardware status.
* **Non-Deterministic Inputs:** Mocking things that change every time you run them, such as system time (`datetime.now()`) or random number generators.

---

### Scenario B: Bad Mocking Targets (The Code Smell)
If you find yourself mocking your own helper functions, mathematical subroutines, or internal class structures, you are likely using mocks to mask bad software design. 

::: warning ## Code Smell: "Over-Mocking"
If you have to mock out 5 layers of your own internal code just to write a unit test for a single function, **your codebase is tightly coupled**. 

Your test is now directly bound to the internal implementation details of your code. The moment you change a variable name, split a function, or optimize your logic, all your tests will break—even if the overall calculation is still correct.
:::

### The Solution: Decouple Side Effects from Logic
Instead of writing increasingly complex mocks, the best approach is to refactor your code to separate **I/O / side effects** from **pure calculation logic**. 

1. Write a small, dirty function that performs the I/O (reads the database, makes the API call) and returns raw data.
2. Write a pure, deterministic mathematical function that takes that raw data and does the calculations. 
3. Write clean, mock-free unit tests for your calculation logic. Test the I/O function sparingly using a dedicated integration test.

::: rationale ## Discussion
Look at a recent script or program you wrote for your research. 

* Where does it read files, query databases, or call external libraries? 
* Are those actions mixed in directly with your calculations, or are they isolated? 
* If you wanted to test your core algorithm, how much of your own script would you have to mock out?
:::
