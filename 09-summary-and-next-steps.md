---
title: "Summary and Next Steps"
teaching: 10
exercises: 0
---

## Key Takeaways

Throughout this workshop, we transitioned from writing reactive, ad-hoc scripts to designing robust, self-documenting computational workflows. Here is the core checklist to carry back to your scientific projects:

* **Test First, Code Second:** Writing tests before implementation forces you to define your mathematical specifications and boundary conditions clearly before you write a single line of production code.
* **The Red-Green-Refactor Loop:** Write a test that fails (Red), write the absolute minimum implementation to make it pass (Green), and then clean up your design (Refactor) with your test suite as a safety net.
* **Parametrize to Kill Boilerplate:** Avoid duplicating test code. Use `@pytest.mark.parametrize` to feed dozens of coordinate combinations through a single test interface.
* **Let the Machine Find the Bugs:** Use property-based testing with `Hypothesis` to test underlying mathematical invariants (like symmetry, idempotency, or commutativity) rather than relying solely on hand-written coordinate examples.
* **Mock Only the Boundaries:** Use mocks to isolate your code from slow, unstable, or non-deterministic external dependencies (the network, the database, or the filesystem). Over-mocking your own internal helper functions is a code smell indicating tightly coupled design.
* **Dismantle Legacy Code Safely:** When refactoring a legacy monolith, freeze the existing behavior by writing a high-level "characterization test" (integration test) first. Only then should you begin extracting modules and writing fast, targeted unit tests.
* **Fix Bugs by Writing Tests:** Never fix a production bug in the implementation code first. Write a reproducing test case, watch it fail, and then modify your math to fix it. This creates a permanent regression guard.
* **Co-Author with AI Safely:** Leverage LLMs to write boilerplate, generate parametric test cases, and draft property-based strategies. However, never let an LLM write post-hoc tests for messy, untested code, as this bypasses critical design feedback and fails to build genuine trust.

---

## Where to Go From Here: Topics for Further Study

Now that you have mastered the fundamentals of Test-Driven Development, here are the key concepts and tools to explore next as you scale your research pipelines:

### 1. Continuous Integration (CI) with GitHub Actions
Do not rely on running `pytest` manually on your laptop. Set up a Continuous Integration workflow to automatically execute your test suite every single time you or a collaborator pushes code to GitHub. This ensures your main branch remains clean and mathematically valid.

### 2. Mutation Testing (Using Mutmut)
How do you test your tests? Mutation testing tools automatically inject tiny, silent bugs into your production code (like swapping a `>` to a `<` or changing a `+` to a `-`) and run your test suite. If your tests still pass despite the mutation, your test suite is missing critical assertions.

### 3. Pytest Fixtures for Integration Testing
Learn how to write robust integration tests using Pytest "fixtures." Fixtures allow you to safely spin up temporary, isolated test filesystems or dummy databases, configure them before your tests run, and automatically tear them down afterward.

### 4. Behavior-Driven Development (BDD)
For larger, cross-disciplinary collaborations, frameworks like `Behave` allow you to write executable specifications in plain English (using the "Given-When-Then" syntax). This allows domain scientists, stakeholders, and programmers to collaborate on a single, shared source of truth.

