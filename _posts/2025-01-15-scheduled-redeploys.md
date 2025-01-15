---
layout: post
title: "How to Enable Scheduled Redeploys in Github Actions"
date: "2025-01-15 12:00:00 +0000"
tags: [github]
excerpt: "This post explains how to to setup github actions so that your jekyll static website is rebuild daily, irrespective of push redeploys, enduring that pushed future posts are added to the live site on the right day automatically."
---

I host this blog on github, which is useful because when I add a post scheduled for release in the past (specified in the frontmatter of the .md post file in `date: 2025-01-01 08:00:00 +0000` for example) and then test the site locally with `bundle exec jekyll serve` I can see exactly what the site will look like when I push the changes to github and trigger a rebuild/redeploy of the website. When I write a post scheduled for a time in the future I can also see what the site will look like online with `bundle exec jekyll serve --future` which builds all posts irrespective of the frontmatter `date` tag. Without amending the github actions this causes a small problem. By default github actions only rebuilds/redeploys the site when changes are pushed to github, so if there are several posts with frontmatter `date` in the future, these posts will not be build until a new push ofter their release date. To automatically rebuild/redeploy the site daily we need to add a scheduled redeployments. Luckily, this is straightforward.

Step 1. Create a `schedule-redeploy.yml` configuration file in `.github/workflows`:

```bash
mkdir -p .github/workflows
touch .github/workflows/schedule-redeploy.yml
```

Step 2. Edit the `schedule-redeploy.yml`:

```yml
# name of the workflow, visible on the github Actions tab
name: Scheduled Redeployment

# define the trigger for the workflow
on:
  schedule:
    # cron is a Unix-based utility for scheduling tasks ("cron jobs")
    # Runs every day at 06:00 AM UTC
    - cron: "0 6 * * *"

# define the job and specify the environment (ubuntu-latest) on which the job will run
jobs:
  rebuild:
    runs-on: ubuntu-latest

    # check out the repo code into the workflow environment
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

        # configure git with username and email for committing changes (required by git)
      - name: Configure Git
        run: |
          git config --global user.name "GitHub Actions"
          git config --global user.email "actions@github.com"

        # use the GITHUB_TOKEN to grant permission in the repo settings
        # creates an empty commit (--allow-empty) to avoid modifying the repo's content
        # pushes the empty commit to the repo, triggering a github pages rebuild
      - name: Commit and Push Empty Change
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          git commit --allow-empty -m "Trigger Pages Build"
          git push
```

Step 3. Commit the new `schedule-redeploy.yml` file and push to github:

```bash
git add .github/worklows/schedule-redeploy.yml
git commit -m "added schedule-redeploy.yml for scheduled redeploy"
git push
```

Step 4. If you leave it here, the scheduled redeploy action will fail because by default the workflow settings only allows actions read permissions. You need to set read and write permissions to workflow in the repo Settings->Actions->General section and save the change:

<div style="text-align: center;">
   <img src="{{ site.baseurl }}/assets/posts/github-workflow-permissions.png" style="width: 100%;">
</div>

This will allow the scheduled redeploy to work. You can test this by setting the `schedule-redeploy.yml` `cron` to a few minutes in the future and checking the github Actions tab to check if the action ran successfully. Once this works you can change the `cron` time to whenever you want to action to be performed daily (or monthly or yearly).

<!-- I have had to use unit tests before: I made changes to a codebase, pushed to github, and the tests (written by someone else) failed, so I re-wrote the code until they passed. I didn't understand how unit tests actually worked (I still don't fully understand). This post introduces to basics of using the Python standard library module `unittest`. It is heavily influenced by [Corey Shafer's](https://www.youtube.com/@coreyms) great introduction video on the topic [Python Tutorial: Unit Testing Your Code with the unittest Module](https://www.youtube.com/watch?v=6tNS--WetLI). It won't go into details of how to integrate tests into github actions, where it becomes really powerful and basically essential for collaborating on large codebases uisng continuous integration (CI) principles.

To get started, we need a module to import our custom functions we want to test. I have created a module called `calculator.py` with a single class `Calculator` with `add`, `subtract`, `multiply` and `divide` methods:

```python
# calculator.py
class Calculator:
    """Simple calculator class for basic arithmetic operations."""
    def add(self, a, b):
        return a + b

    def subtract(self, a, b):
        return a - b

    def multiply(self, a, b):
        return a * b

    def divide(self, a, b):
        if b == 0:
            raise ValueError("Cannot divide by zero.")
        return a / b
```

We then need to import `unittest` and the `Calculator` class from this module into out `test_calculator.py`. `unittest` test files must begin with `test_` to be recognised by the `unittest` framework. `test_calculator.py` has the `TestCalculator` class which inherits from the `unittest.TestCase` class. In that class there are:

- two classmethods: `setUpClass(cls)`, for setting up the class at the start of the tests, which inherits from the cls itself, and `tearDownClass(cls)` for performing actions at the end the tests, e.g. deleting test generated files or closing open connections
- two methods that are run at the start and end of every individual test: `setUp(self)` (which defines some instance variables and initializes an instance of the `Calculator()` class, and `tearDown(self)` that performs actions at the end of each test
- four individual tests, each of which tests an individual `Calculator` method. It is important that each test is isolated and only tests a single thing, so we don't want a test that test `calc.add` and `calc.substract` together, but each test can test a single method in different ways. Each test is a method that must begin with `test_`.

The heart of unittest is `self.assertEqual()` which does what it says on the tin, the test is successful when each `self.assertEqual()` in that test passes. Here are the most commonly used assert methods (from the `unittest` [documentation](https://docs.python.org/3/library/unittest)).

| Method                                                                                                               | Checks that            |
| -------------------------------------------------------------------------------------------------------------------- | ---------------------- |
| [`assertEqual(a, b)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertEqual)                 | `a == b`               |
| [`assertNotEqual(a, b)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertNotEqual)           | `a != b`               |
| [`assertTrue(x)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertTrue)                      | `bool(x) is True`      |
| [`assertFalse(x)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertFalse)                    | `bool(x) is False`     |
| [`assertIs(a, b)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertIs)                       | `a is b`               |
| [`assertIsNot(a, b)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertIsNot)                 | `a is not b`           |
| [`assertIsNone(x)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertIsNone)                  | `x is None`            |
| [`assertIsNotNone(x)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertIsNotNone)            | `x is not None`        |
| [`assertIn(a, b)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertIn)                       | `a in b`               |
| [`assertNotIn(a, b)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertNotIn)                 | `a not in b`           |
| [`assertIsInstance(a, b)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertIsInstance)       | `isinstance(a, b)`     |
| [`assertNotIsInstance(a, b)`](https://docs.python.org/3/library/unittest.html#unittest.TestCase.assertNotIsInstance) | `not isinstance(a, b)` |

Here is the unittest code:

```python
# test_calulator.py
import unittest
from calculator import Calculator

class TestCalculator(unittest.TestCase):

    # @classmethod decorator defines a method bound to the class, not an instance of the class
    # # useful when you want to perform a costly function once (e.g. set up a database) and then run all the tests against that, instead of setting it up each time in setUp
    @classmethod
    def setUpClass(cls):
        print('set up class\n')

    @classmethod
    def tearDownClass(cls):
        print('tear down class')

    # runs this code before every single test
    def setUp(self):
        print('set up')
        self.a = 5
        self.b = 10
        # initialize our Calculator class for each test
        self.calc = Calculator()

    # runs this code after every single test
    # # useful if functions create files which you want to delete after each test
    def tearDown(self):
        print('tear down\n')

    def test_add(self):
        print('test_add')
        result = self.calc.add(6, 8)
        self.assertEqual(result, 14)

        # use the variables defined in the test setUp(self)
        self.assertEqual(self.calc.add(self.a, self.b), 15)

    def test_subtract(self):
        print('test_subtract')
        self.assertEqual(self.calc.subtract(6, 8), -2)

    def test_multiply(self):
        print('test_multiply')
        self.assertEqual(self.calc.multiply(5, 6), 30)
        self.assertEqual(self.calc.multiply(0, 0), 0)
        self.assertEqual(self.calc.multiply(-1, 0), 0)

    def test_divide(self):
        print('test_divide')
        self.assertEqual(self.calc.divide(6, 2), 3)

        # two ways to use assertRaises
        # 1. without the context manager
        self.assertRaises(ValueError, self.calc.divide, 10, 0)
        # 2. with the context manager
        # # preferred way to use assertRaises, using the context manager
        with self.assertRaises(ValueError):
            self.calc.divide(5, 0)

# comment out the following lines if you want to use unittest from the command line with:
# # python -m unittest test_calculator.py
if __name__ == '__main__':
    unittest.main()
```

This can be run as a python command in the terminal, which will run the tests from `unittest.main()`, or if you want to run the tests directly from the terminal, you can comment out the `if __name__ == 'main': unittest.main()` section and run `python -m unittest test_calculator.py`. This should print all the print statements each time ther are run (they are not needed, but they show when the class is set up and torn down, and when each individual test is set up and torn down for demonstrative purposes). If you make an edit to the calculator module to introduce a bug, e.g:

```python
def add(self, a, b):
    return a + b + 10 # deliberately introduce bug
```

this will cause one of the tests to fail, and a traceback will be shown to show exactly where the failure occurred. Here is an example:

```bash
$ python -m unittest test_calculator.py
set up class

set up
test_add
Ftear down

set up
test_divide
tear down

.set up
test_multiply
tear down

.set up
test_subtract
tear down

.tear down class

======================================================================
FAIL: test_add (test_calculator.TestCalculator.test_add)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "[PATH_TO_FILE]/test_calculator.py", line 33, in test_add
    self.assertEqual(result, 14)
AssertionError: 24 != 14

----------------------------------------------------------------------
Ran 4 tests in 0.000s

FAILED (failures=1)
```

### Some Best Practices for Writing Unit Tests

1. Isolation of tests: Each test should be independent and pass or fail on its own, regardless of the outcome of other tests. This means avoiding persistent state between tests that could contaminate each other.
2. Test-Driven Development (TDD): Some developers operate the "TDD" framework, where the idea is to design the functionality you want, write the tests first to check that functionality, and then write the code to pass the tests. This helps clarify requirements and encourages clean, testable code.
3. Start simple: Any testing is better than no testing. Begin with basic assertions like `assertEqual` or `assertTrue`. Simple tests can still catch silly but significant bugs and help build confidence when learning/developing.

Happy testing! -->
