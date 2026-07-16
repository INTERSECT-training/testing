---
title: "Brittle Tests and Bugs"
teaching: 20
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions 

- How should you respond to bugs?
- What does it mean if you have to change a lot of tests while adding features?
- What are the advantages of testing an interface?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Learn how to use TDD when making large changes to code

::::::::::::::::::::::::::::::::::::::::::::::::


## What to Do When You Find a Bug

No matter how disciplined you are with Test-Driven Development, bugs will eventually slip into your production code. When this happens, your instinct might be to open the file and fix the code immediately. 

However, professional software engineering dictates a strict, non-negotiable rule: **never touch the implementation code first**.

### The Bug Adage

> **The Rule of Bug Fixing:**
> The moment you encounter a bug, the very first thing you must do is write a test that reproduces the bug. Watch that test fail (turn Red), and *then* modify your code to make it pass (turn Green).

Why do we force this workflow?
1. **It proves you understand the problem:** If you cannot write a test that reliably fails because of the bug, you do not actually understand what is causing the bug.
2. **It creates a historical record:** The test is committed to Git alongside the fix, documenting exactly when and why the issue was discovered.
3. **It prevents regressions:** Once that test is added to your test suite, it acts as a permanent shield, ensuring that exact bug can never slip back into your codebase unnoticed.

---

## Example: The Touching Border Bug

Imagine a researcher running our rectangle overlap code finds a bug: when two rectangles share an edge (for example, Rectangle A's right border is exactly at $x=5$, and Rectangle B's left border starts exactly at $x=5$), our code returns `True` (they overlap). 

In their physics simulation, sharing an edge does *not* constitute a physical overlap. 

Following our rule, we do not edit `overlap.py` yet. Instead, we write a reproducing test case first:

```python
# Inside test_overlap.py

def test_touching_borders_returns_false():
    # Arrange: Rectangles share a border at x=5, but do not overlap
    rect_a = (0, 0, 5, 5)
    rect_b = (5, 0, 10, 5)
    
    # Act & Assert (This will fail if our math treats touching as overlapping)
    assert check_overlap(rect_a, rect_b) is False
```

Run `pytest`. Once you verify that this new test fails, open `overlap.py`, adjust your inequality operators (e.g., changing `<=` to `<`), and run your test suite again to verify the fix works and hasn't broken any other scenarios.

---

## Testing Interfaces, Not Implementations (Brittle Tests)

A common trap for developers is writing **brittle tests**—tests that break when you make minor changes to your code, even though the overall behavior is still perfectly correct.

Brittle tests usually happen because you tested *how* the code does something (implementation details) rather than *what* the code is contracted to do (the interface).

### The Scenario
Suppose our legacy pipeline from Episode 6 outputs a matrix file of 1s and 0s. 

* **Brittle Approach:** We write unit tests that assert the exact variable names, loop counters, and temporary file paths used inside `legacy_detector.py` to build that matrix.
* **Robust Approach:** We only test the input file parsing and the output matrix file correctness (the public contract).

If our requirements change tomorrow, and we decide we want to output a text report containing the actual overlap percentage area instead of a binary matrix, the brittle tests will completely break and require a massive rewrite. The robust tests, however, will allow us to completely swap out the internal formulas or data structures while remaining green.

::: rationale

## The Design Rule
Always design your tests to assert against the **public interface** of your modules (the inputs and expected outputs) rather than the internal, private helper mechanisms. 

Testing boundaries allows you to completely refactor your math, change internal library dependencies, or optimize execution speeds without having to constantly rewrite your test suite.
:::

---

## Preventing Feature Creep

Why didn't we design our overlap detector to handle 3D spheres, 1D lines, or rotated polygons right from the beginning?

Test-Driven Development acts as a powerful psychological shield against **feature creep** (also known as YAGNI: *"You Aren't Gonna Need It"*). 

When you write implementation code without tests, it is incredibly easy to get distracted by "what-ifs." You start writing complex helper classes and configuration options for scenarios that might never happen. This introduces unnecessary complexity, increases maintenance overhead, and creates more surface area for bugs to hide.

By forcing yourself to write the test first, you force yourself to answer: **"What is the exact requirement I need to solve right this second?"** If there is no test asking for a feature, you do not write the code for it. This simple constraint keeps your scientific codebase lean, clean, and strictly aligned with your actual research objectives.

::::::::::::::::::::::::::::::::::::: keypoints 

- Changing a lot of test code for minor features can indicate your tests are not DRY and heavily coupled.
- Do NOT invent a Swiss-army knife! TDD helps keep you focused on iterative, essential development.
- Testing a module's interface focuses tests on what a user would typically observe.  You don't have to change as many tests when internal change.

::::::::::::::::::::::::::::::::::::::::::::::::
