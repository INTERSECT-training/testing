---
title: "Refactoring Legacy Code"
teaching: 20
exercises: 15
---
:::::::::::::::::::::::::::::::::::::: questions 

- Why are long methods hard to test?


::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Learn some key refactoring methods to change code safely.
- Modify the overlap script to support some new features or design goals.

::::::::::::::::::::::::::::::::::::::::::::::::


## Confronting the Legacy Monolith

In scientific research, you will rarely start projects from a completely blank slate. More often, you will inherit an existing codebase—frequently referred to as "legacy code." 

Legacy code is often defined simply as **code that doesn't have tests**. It might be a single, massive 1,000-line Python script that mixes file parsing, mathematical modeling, simulation execution, and plotting into one giant monolithic file.

Without tests, changing even a single line of this monolith feels incredibly risky. How do we clean up, modularize, and improve this code without breaking its existing functionality?

---

## Step 1: Establish a Safety Net (Characterization Testing)

We cannot use Test-Driven Development on legacy code because the code already exists. Instead, our first step is to write a **Characterization Test** (also known as a Golden Master or End-to-End test).

A characterization test does not check if the code is elegantly written; it simply documents what the code *actually does* right now. 

### The Legacy Pipeline
Imagine we have inherited a legacy script named `legacy_detector.py`. This script reads a text file containing coordinates of multiple rectangles, parses them, runs nested loops to detect overlaps, and writes a raw matrix file output (`matrix.txt`) where $i,j = 1$ if rectangle $i$ and $j$ overlap, and $0$ otherwise.

To test this safely, we will write a high-level integration test that runs the script as an external process, feeds it a sample input file, and checks that the generated output matches a verified "gold standard" reference file.

```python
# test_legacy.py
import subprocess
import os
import shutil

def test_legacy_integration():
    # Arrange: Set up our input and output file paths
    input_file = "test_data/sample_rectangles.txt"
    output_file = "matrix.txt"
    reference_file = "test_data/reference_matrix.txt"
    
    # Ensure any old output files are cleared
    if os.path.exists(output_file):
        os.remove(output_file)
        
    # Act: Run the legacy script as a subprocess
    result = subprocess.run(
        ["python", "legacy_detector.py", input_file],
        capture_output=True,
        text=True
    )
    
    # Assert: Verify the script ran successfully and generated the correct output
    assert result.returncode == 0
    assert os.path.exists(output_file)
    
    # Compare the output file byte-for-byte with our Golden Master reference
    with open(output_file, "r") as f_out, open(reference_file, "r") as f_ref:
        assert f_out.read() == f_ref.read()
```

::: challenge

## Exercise: Run the Integration Test

1. Create a dummy legacy script structure or use the provided legacy code in your environment.
2. Run `pytest test_legacy.py`. 
3. Verify that your high-level integration test passes (turns green). 

This is now your permanent safety net. If you break *anything* during the refactoring process, this test will catch it.
:::

---

## Step 2: Systematically Dismantle the Monolith

With our safety net in place, we can begin extracting modules from the legacy file.

### Task A: Swap the Core Math
Find the nested coordinate loop calculations inside `legacy_detector.py`. This is where the legacy code does raw, hard-to-read inequality checks. 

Since we built and fully tested a clean `overlap.py` module in Episode 1 and 2, we can swap out the legacy math with our tested module:

```python
# Inside legacy_detector.py

# BEFORE:
# if not (r1[2] < r2[0] or r2[2] < r1[0] or r1[3] < r2[1] or r2[3] < r1[1]):
#     matrix[i][j] = 1

# AFTER:
from overlap import check_overlap
if check_overlap(r1, r2):
    matrix[i][j] = 1
```

Once you make this change, run your integration test:

```bash
pytest test_legacy.py
```

If it passes, you have successfully refactored a piece of the legacy logic while maintaining absolute system stability.

---

### Task B: Extract File Parsing
Next, locate the section of `legacy_detector.py` that reads the input file and splits strings into numeric coordinate tuples. Extract this raw I/O code into its own clean, isolated helper function:

```python
def parse_input_file(filepath):
    rectangles = []
    with open(filepath, "r") as f:
        for line in f:
            parts = line.strip().split(",")
            rectangles.append(tuple(map(float, parts)))
    return rectangles
```

Run `pytest test_legacy.py` again. 

Because we decoupled file parsing from the execution loop, we can now write small, rapid, mock-free **unit tests** specifically for `parse_input_file()`, verifying how it handles empty lines, whitespace, or bad input characters in memory.

---

## The Refactoring Rule of Thumb

When modernizing legacy systems, never attempt to rewrite the entire codebase from scratch in one go. Instead, use a branch-and-extract workflow:

1. Write a high-level **characterization test** to freeze existing behavior.
2. Identify a single responsibility (like file parsing or coordinate math).
3. Extract that responsibility into an isolated, pure function.
4. Write fast, specific unit tests for your new function.
5. Replace the legacy code blocks with calls to your new function.
6. Run your high-level characterization test to verify nothing broke.
7. Repeat.

::::::::::::::::::::::::::::::::::::: keypoints 

- Testing long methods is difficult since you can't pinpoint a few lines of logic.
- Testable code is also good code!
- Changing code without tests can be dangerous.  Work slowly and carefully making only the simplest changes first.
- Write tests with an adversarial viewpoint.
- Keep tests DRY, fixtures and parameter can help.

::::::::::::::::::::::::::::::::::::::::::::::::
