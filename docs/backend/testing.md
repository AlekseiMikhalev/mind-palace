# Python Testing

Sources:

- Books:
    - [Python Testing with pytest, Second Edition](https://pragprog.com/titles/bopytest2/python-testing-with-pytest-second-edition/)

- Documentation:
    - [pytest](https://docs.pytest.org/en/7.2.x/)


## Main Concepts of Testing

### Test Strategy

#### Determing Test Scope

We will almost always want to test the behavior of the user visible functionality. However, there are quite a few other questions we need to consider when determining how much testing we need to do:

- Is security a concern?
- Performance? Do interactions need to be fast? How fast?
- Loading? Can you handle lots of people with lots of requests? Are you expecting to need to? If so, you should test for that.
- Input validation? For really any system that accepts input from users, we should validate the data before acting on it.

Decide what features to test?

#### Considering Software Architecture

- how your project’s software is organized
- what APIs are available
- what the interfaces are
- where code complexity lives
- modularity
- etc.

Another main factors that affect testing strategy:

- At what level should we be testing? The top user interface? Something lower? Subsystem? All levels?
- How easy is it to test at different levels?
- Who is responsible for the different levels and the testing of each? If you are supplying a subsystem, are you only responsible for that subsystem? Is someone else doing the system testing? If so, it’s an easy choice: test your own subsystem.

#### Evaluating the features to test

Prioritize features to test based on the following factors:

- *Recent* — New features, new areas of code, new functionality that has been recently repaired, refactored, or otherwise modified
- *Core* — Your product’s unique selling propositions (USPs). The essential functions that must continue to work in order for the product to be useful
- *Risk* — Areas of the application that pose more risk, such as areas important to customers but not used regularly by the development team or parts that use third-party code you don’t quite trust
- *Problematic* — Functionality that frequently breaks or often gets defect reports against it
- *Expertise* — Features or algorithms understood by a limited subset of people

#### Creating test cases

- Start with a non-trivial, “happy path” test case.
- Then look at test cases that represent

    - interesting sets of input,
    - interesting starting states,
    - interesting end states, or
    - all possible error states.

### Coverage

Code coverage tool cannot tell you if your test suite is good; it can only tell you how much of the application code is getting hit by your test suite. But that in itself is useful information. 

```console
    $ pip install coverage
    $ pip install pytest-cov
```

### Commands

```shell
    pytest -v test_two.py #(1)

    pytest --tb=no #(2)

    pytest --tb=no test_one.py test_two.py #(3)
```

1. `-v` adds information if test fails and shows exactly the reason of failure
2. use `--tb=no` flag to rutn off tracebacks
3. indicate specific tests to run

### Test naming conventions

- Test files should be named `test_<something>.py` or `<something>_test.py`.
- Test methods and functions should be named `test_<something>`.
- Test classes should be named `Test<Something>`

### Test exceptions

If we need to test expected exceptions we can use `pytest.raises()`:

```python
    import pytest
    import cards
    
    def test_no_path_raises():
        with pytest.raises(TypeError): #(1)
            cards.CardsDB()

    def test_raises_with_info(): #(2)
        match_regex = "missing 1 .* positional argument"
        with pytest.raises(TypeError, match=match_regex):
            cards.CardsDB()
    
    def test_raises_with_info_alt():
        with pytest.raises(TypeError) as exc_info:
            cards.CardsDB()
        expected = "missing 1 required positional argument"
        assert expected in str(exc_info.value)
```

1. the `with pytest.raises(TypeError):` statement says that whatever is in the next block of code should raise a TypeError exception. If no exception is raised, the test fails. If the test raises a different exception, it fails.

2. check to make sure the message is correct, or any other aspect of the exception, like additional parameters

### Structuring test functions

```python
    def test_to_dict():
        # GIVEN a Card object with known contents
        c1 = Card("something", "brian", "todo", 123) #(1)

        # WHEN we call to_dict() on the object
        c2 = c1.to_dict() #(2)

        # THEN the result will be a dictionary with known content
        c2_expected = { 
            "summary": "something",
            "owner": "brian",
            "state": "todo",
            "id": 123,
        }
        assert c2 == c2_expected #(3)
```

1. A starting state. This is where you set up data or the environment to get ready for the action.

2. Some action is performed. This is the focus of the test—the
behavior we are trying to make sure is working right.

3. Some expected result or end state should happen. At the end of the test, we make sure the action resulted in the expected behavior.

Structure tests in classes:

```python
    class TestEquality:
        def test_equality(self):
            c1 = Card("something", "brian", "todo", 123)
            c2 = Card("something", "brian", "todo", 123)
            assert c1 == c2

        def test_equality_with_diff_ids(self):
            c1 = Card("something", "brian", "todo", 123)
            c2 = Card("something", "brian", "todo", 4567)
            assert c1 == c2

        def test_inequality(self):
            c1 = Card("something", "brian", "todo", 123)
            c2 = Card("completely different", "okken", "done", 123)
            assert c1 != c2
```

Running a single test method, test class, or module:

```console
    $ pytest ch2/test_classes.py::TestEquality::test_equality
    $ pytest ch2/test_classes.py::TestEquality
    $ pytest ch2/test_classes.py
```

Running a single test function or module:

```console
    $ pytest ch2/test_card.py::test_defaults
    $ pytest ch2/test_card.py
```

Running the whole directory:

```console
    $ pytest ch2
```

### Pytest Fixtures

Pytest treats exceptions differently during fixtures compared to during a test function. An exception (or assert failure or call to `pytest.fail()`) that happens during the test code proper results in a “Fail” result. However, during a fixture, the test function is reported as “Error.” This distinction is helpful when debugging why a test didn’t pass. If a test results in “Fail,” the failure is somewhere in the test function (or something the function called). If a test results in “Error,” the failure is somewhere in a fixture.

```python
    import pytest

    @pytest.fixture()
    def cards_db(): #(1)
        with TemporaryDirectory() as db_dir:
            db_path = Path(db_dir)
            db = cards.CardsDB(db_path)
            yield db #(2)
            db.close()

    def test_empty(cards_db):
        assert cards_db.count() == 0

    def test_two(cards_db):
        cards_db.add_card(cards.Card("first"))
        cards_db.add_card(cards.Card("second"))
        assert cards_db.count() == 2
```

1. `cards_db` fixture is “setting up” for the test by getting the database ready. It is like a dependency, which is injected into the test function.

2. Fixture functions run before the tests that use them. If there is a `yield` in the function, it stops there, passes control to the tests, and picks up on the next line after the tests are done.

The following command-line flag `--setup-show` shows us the order of operations of tests and fixtures:

```console
    $ pytest --setup-show test_count.py
    ======================== test session starts =========================
    collected 2 items

    test_count.py
        SETUP
        F cards_db
        ch3/test_count.py::test_empty (fixtures used: cards_db).
        TEARDOWN F cards_db
        SETUP
        F cards_db
        ch3/test_count.py::test_two (fixtures used: cards_db).
        TEARDOWN F cards_db
    ========================= 2 passed in 0.02s ==========================
```

Fixture scope:

```python
    @pytest.fixture(scope="module") #(1)
    def cards_db():
        with TemporaryDirectory() as db_dir:
            db_path = Path(db_dir)
            db = cards.CardsDB(db_path)
            yield db
            db.close()
```

1. `scope="module"` allows us to open database only once and use the result in the whole module

Using `scope` allows us to run slow part of the test once for multiple tests:

```console
    $ pytest --setup-show test_mod_scope.py
    ========================== test session starts ==========================
    collected 2 items

    test_mod_scope.py
        SETUP
        M cards_db
        ch3/test_mod_scope.py::test_empty (fixtures used: cards_db).
        ch3/test_mod_scope.py::test_two (fixtures used: cards_db).
        TEARDOWN M cards_db
    =========================== 2 passed in 0.03s ===========================
```

`scope` options:

- `scope='function'` - Run once per test function. The setup portion is run before each test using the fixture.

- `scope='class'` - Run once per test class, regardless of how many test methods are in the class.

- `scope='module'` - Run once per module, regardless of how many test functions or methods or other fixtures in the module use it.

- `scope='package'` - Run once per package, or test directory, regardless of how many test functions or methods or other fixtures in the package use it.

- `scope='session'` - Run once per session. All test methods and functions using a fixture of session scope share one setup and teardown call.

The `conftest.py` file is considered by pytest as a “local plugin” and can contain hook functions and fixtures. Thus we can share fixtures among other tests. 

Tests can use any fixture that is in the same test module as a test function, or in a `conftest.py` file in the same directory, or in any level of parent directory up to the root of the tests.

```console
    $ pytest --fixtures -v
    ...
    -------------------- fixtures defined from conftest ---------------------
    cards_db [session scope] -- conftest.py:7
    CardsDB object connected to a temporary database
    ...
```

#### Builtin Fixtures

The `tmp_path` and `tmp_path_factory` fixtures are used to create temporary directories. 

The `tmp_path` function-scope fixture returns a `pathlib.Path` instance that points to a temporary directory that sticks around during your test and a bit longer. 

The `tmp_path_factory` session-scope fixture returns a `TempPathFactory` object. This object has a `mktemp()` function that returns `Path` objects. We can use `mktemp() `to create multiple temporary directories.

Example:

```python 
    def test_tmp_path(tmp_path):
        file = tmp_path / "file.txt"
        file.write_text("Hello")
        assert file.read_text() == "Hello"
    
    def test_tmp_path_factory(tmp_path_factory):
        path = tmp_path_factory.mktemp("sub")
        file = path / "file.txt"
        file.write_text("Hello")
        assert file.read_text() == "Hello"
```

#### Parametrization

The `@pytest.mark.parametrize()` decorator is used to define the sets of arguments to pass to the test:

```python
    import pytest
    from cards import Card
    
    @pytest.mark.parametrize(
        "start_summary, start_state", #(1)
        [
            ("write a book", "done"), #(2)
            ("second edition", "in prog"),
            ("create a course", "todo"),
        ],
    )

    def test_finish(cards_db, start_summary, start_state):
        initial_card = Card(summary=start_summary, state=start_state)
        index = cards_db.add_card(initial_card)

        cards_db.finish(index)

        card = cards_db.get_card(index)
        assert card.state == "done"
```

1. list of names of the parameters. It can be list of strings, comma-separated strings.

2. our list of test cases. Each element in the list is a test case represented by a tuple or list that has one element for each argument that gets sent to the test function.

Result:

```console
    $ pytest -v test_func_param.py::test_finish
    ========================= test session starts ==========================
    collected 3 items

    test_func_param.py::test_finish[write a book-done] PASSED [ 33%]
    test_func_param.py::test_finish[second edition-in prog] PASSED [ 66%]
    test_func_param.py::test_finish[create a course-todo] PASSED [100%]
    ========================== 3 passed in 0.05s ===========================
```

Fixture paramentization:

```python
    @pytest.fixture(params=["done", "in prog", "todo"]) #(1)
    def start_state(request):
        return request.param
    
    def test_finish(cards_db, start_state):
        c = Card("write a book", state=start_state)
        index = cards_db.add_card(c)
        cards_db.finish(index)
        card = cards_db.get_card(index)
        assert card.state == "done"
```

1. pytest will then call the fixture once each for every set of values we provide. Then downstream, every test function that depends on the fixture will be called, once each for every fixture value.

### Mocking

The `mock` package is used to swap out pieces of the system to isolate bits of our application code from the rest of the system.

