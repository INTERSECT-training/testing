---
title: "Property-Based Testing"
teaching: 20
exercises: 15
---

:::::::::::::::::::::::::::::::::::::: questions 

- What is property-based testing (PBT)?
- How to write property-based tests?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- See how property-based testing works with Hypothesis.

::::::::::::::::::::::::::::::::::::::::::::::::


## The Limits of Hand-Written Examples

So far, we have tested our overlap function using hand-written coordinate examples. While this approach is great for establishing initial confidence and designing the function interface, it has a significant limitation: **you can only test for the bugs you can anticipate**.

If we measure our test suite's code coverage, it might show 100%. However, 100% coverage only means that every line of code was executed at least once during our tests. It does *not* prove that the code is correct under all possible input combinations. For example, what happens if we input negative numbers, extremely large integers, or coordinates where the rectangle has zero width or height?

Instead of manually inventing more specific coordinate pairs to test, we can use **Property-Based Testing** to describe the underlying mathematical rules (properties) that our program must always obey, and let the computer generate hundreds of random scenarios to try and break those rules.

---

## Introduction to Hypothesis

In Python, we use a library called `Hypothesis` to perform property-based testing.

```bash
pip install hypothesis
```

Rather than writing tests that assert specific outputs for specific inputs (like "Rectangle A at (0,0) and Rectangle B at (5,5) returns False"), a property-based test asserts that a general rule always holds true.

### Finding Mathematical Properties in Space

When dealing with spatial overlaps, what properties are always true, regardless of what coordinates we choose?

1. **Commutativity (Symmetry):** If Rectangle $A$ overlaps Rectangle $B$, then Rectangle $B$ *must* overlap Rectangle $A$. The order of arguments should not change the result: 
   $$\text{overlap}(A, B) \equiv \text{overlap}(B, A)$$
2. **Idempotency (Self-Overlap):** Any valid rectangle with non-zero area *must* overlap with itself:
   $$\text{overlap}(A, A) \equiv \text{True}$$

---

## Writing a Property Test

To write these tests, we need to tell Hypothesis how to generate valid rectangles. In our design, a rectangle is defined by four coordinates: `(xmin, ymin, xmax, ymax)`. To make a rectangle physically valid, we must ensure that $xmin \le xmax$ and $ymin \le ymax$. 

We can define this generation rule using a custom strategy in Hypothesis with the `@st.composite` decorator.

::: challenge ## Terminology: "Drawing" in Hypothesis

In the strategy code below, you will see a parameter called `draw`. 

* **What it does NOT mean:** It does not mean "drawing a shape," rendering a line, or sketching a picture on your screen.
* **What it DOES mean:** Think of it like **"drawing a card from a deck of possibilities."** You are telling the testing engine: *"Draw a random integer from the deck of integers between -100 and 100, and hand it to me so I can construct my coordinates."*
:::

Create a new test file named `test_properties.py`:

```python
from hypothesis import given, strategies as st
from overlap import check_overlap

# Define a custom data strategy to generate valid, physical rectangles
@st.composite
def rectangles(draw):
    # Draw four coordinates from our integer deck
    x1 = draw(st.integers(min_value=-100, max_value=100))
    x2 = draw(st.integers(min_value=-100, max_value=100))
    y1 = draw(st.integers(min_value=-100, max_value=100))
    y2 = draw(st.integers(min_value=-100, max_value=100))
    
    # Ensure min coordinate is always less than or equal to max coordinate
    return (min(x1, x2), min(y1, y2), max(x1, x2), max(y1, y2))

# Test Property 1: Commutativity (Symmetry)
@given(rectangles(), rectangles())
def test_overlap_is_commutative(rect_a, rect_b):
    result_ab = check_overlap(rect_a, rect_b)
    result_ba = check_overlap(rect_b, rect_a)
    assert result_ab == result_ba

# Test Property 2: Self-Overlap (Idempotency)
@given(rectangles())
def test_self_overlap(rect):
    # Only check rectangles that have actual non-zero area (width and height > 0)
    if rect[2] > rect[0] and rect[3] > rect[1]:
        assert check_overlap(rect, rect) is True
```

Run this file using pytest:

```bash
pytest test_properties.py
```

---

## How Hypothesis Finds Bugs (Shrinking)

If there is a flaw in your overlap math, Hypothesis will not just report a failure; it will systematically attempt to simplify the failing inputs. This process is called **shrinking**. 

If Hypothesis breaks your code with coordinates like `(12, -87, 43, 99)`, it will try smaller values, eventually printing the absolute simplest, most minimal set of numbers that causes your test to fail (often involving simple coordinates like `0`, `1`, or identical overlapping edges).

::: challenge ## Exercise: Unleash the Fuzzer

1. Create the `test_properties.py` file shown above.
2. Run `pytest test_properties.py` to see if your `overlap.py` implementation satisfies these mathematical properties over 100 randomly generated scenarios.
3. **Intentionally introduce a bug:** Open `overlap.py` and temporarily change one of your inequality comparisons (for example, change a `<` to a `<=`). 
4. Run `pytest test_properties.py` again. Look at the terminal output. Notice how Hypothesis identifies the failure and prints the exact "Falsifying example" coordinates.
5. Revert your code back to its passing state once you have observed the shrinking behavior.
:::

::::::::::::::::::::::::::::::::::::: keypoints 

- PBT emphasizes writing conditions that test examples should satisfy
- Actual test cases are auto-generated.
- "Shrinking" zeroes in on the minimal examples that trigger failure.

::::::::::::::::::::::::::::::::::::::::::::::::
