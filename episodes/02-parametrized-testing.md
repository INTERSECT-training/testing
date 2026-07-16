---
title: "Parameterized Testing"
teaching: 15
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions 

- What is a test framework?
- How does a test framework like pytest make writing test cases easier?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Start writing cleaner tests with pytest.

::::::::::::::::::::::::::::::::::::::::::::::::


## The Problem of Test Duplication

Now that you have a passing test suite, look closely at your `test_overlap.py` file. If you wrote three or four different test cases, your file probably looks something like this:

```python
from overlap import check_overlap

def test_overlapping_rectangles():
    rect_a = (0, 0, 2, 2)
    rect_b = (1, 1, 3, 3)
    assert check_overlap(rect_a, rect_b) is True

def test_separated_rectangles():
    rect_a = (0, 0, 2, 2)
    rect_b = (3, 3, 5, 5)
    assert check_overlap(rect_a, rect_b) is False

def test_subset_rectangles():
    rect_a = (0, 0, 4, 4)
    rect_b = (1, 1, 2, 2)
    assert check_overlap(rect_a, rect_b) is True
```

This works, but it is highly repetitive. Every single one of these functions does the exact same thing: it sets up two coordinate tuples, feeds them into `check_overlap`, and asserts a boolean result. 

If you want to test 50 different coordinate combinations, you do not want to copy-paste this boilerplate 50 times. If you change your function's signature later, you will have to rewrite 50 different functions!

---

## The Pytest Way: `@pytest.mark.parametrize`

Pytest solves this problem using a decorator called `parametrize`. This allows you to write the test logic **exactly once** and pass in a list of different inputs and expected outputs.

Here is how it works:

```python
import pytest
from overlap import check_overlap

@pytest.mark.parametrize(
    "rect_a, rect_b, expected",
    [
        ((0, 0, 2, 2), (1, 1, 3, 3), True),   # Overlapping
        ((0, 0, 2, 2), (3, 3, 5, 5), False),  # Separated
    ]
)
def test_overlap_scenarios(rect_a, rect_b, expected):
    assert check_overlap(rect_a, rect_b) is expected
```

### What is Happening Here?
1. **The Decorator:** `@pytest.mark.parametrize` tells pytest that this function is a template.
2. **The Argument Names:** The first string `"rect_a, rect_b, expected"` defines the variable names that will be passed into our test function.
3. **The Data Matrix:** The list of tuples contains our actual test cases. Each tuple in this list represents a single run of the test.
4. **The Test Runner:** When you run `pytest`, it treats each item in the list as an entirely separate test case. If the second case fails, the first one still passes!

---

::: challenge

## Exercise: Parameterize Your Overlap Tests (10 mins)

1. Open `test_overlap.py`.
2. Refactor your individual test functions into a single, clean parameterized test function called `test_overlap_scenarios`.
3. Add at least **five different scenarios** to your parametrization list (including at least one edge case, like completely identical rectangles or one rectangle entirely inside another).
4. Run `pytest -v` (the `-v` flag stands for "verbose") in your terminal. Observe how pytest dynamically generates names for each of your parameterized runs!
:::

::: solution

## Solution

Here is a clean, robust way to parameterize your overlap tests using coordinate tuples:

```python
import pytest
from overlap import check_overlap

@pytest.mark.parametrize(
    "rect_a, rect_b, expected",
    [
        ((0, 0, 2, 2), (1, 1, 3, 3), True),    # Scenario 1: Partial overlap
        ((0, 0, 2, 2), (3, 3, 5, 5), False),   # Scenario 2: Distant separation
        ((0, 0, 4, 4), (1, 1, 2, 2), True),    # Scenario 3: B inside A (nested)
        ((0, 0, 2, 2), (0, 0, 2, 2), True),    # Scenario 4: Identical coordinates
        ((0, 0, 2, 2), (2, 0, 4, 2), False),   # Scenario 5: Shared boundary (touching edge)
    ]
)
def test_overlap_scenarios(rect_a, rect_b, expected):
    assert check_overlap(rect_a, rect_b) is expected
```

If you run this with `pytest -v`, your terminal output will look like this, showing each run as its own independent check:

```bash
test_overlap.py::test_overlap_scenarios[rect_a0-rect_b0-True] PASSED
test_overlap.py::test_overlap_scenarios[rect_a1-rect_b1-False] PASSED
test_overlap.py::test_overlap_scenarios[rect_a2-rect_b2-True] PASSED
test_overlap.py::test_overlap_scenarios[rect_a3-rect_b3-True] PASSED
test_overlap.py::test_overlap_scenarios[rect_a4-rect_b4-False] PASSED
```
:::

::::::::::::::::::::::::::::::::::::: keypoints 

- A test framework simplifies adding tests to your project.
- Choose a framework that doesn't get in your way and makes testing fun.
- Coverage tools are useful to identify which parts of the code are executed with tests

::::::::::::::::::::::::::::::::::::::::::::::::
