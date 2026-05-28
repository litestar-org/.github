# Contribution guide

> This applies to all Litestar repositories, unless otherwise noted in a specific repository.
> 
> This is a very generalized contribution guide. Information below may not apply to all repositories.

## Setting up the environment

1. Install [`poetry`](https://python-poetry.org/)
2. Run `poetry install` to create a [`virtual environment`](https://docs.python.org/3/tutorial/venv.html) and install
   the dependencies
3. Install [`pre-commit`](https://pre-commit.com/)
4. Run `pre-commit install` to install pre-commit hooks

## Code contributions

### Workflow
1. `Fork` the repository you are wanting to contribute to 
2. Clone your fork locally with git
3. [Set up the environment](#setting-up-the-environment)
4. Make your changes
5. **(Optional)** Run `pre-commit run --all-files` to run linters and formatters. This step is optional and will be executed
   automatically by git before you make a commit, but you may want to run it manually in order to apply fixes
6. Commit your changes to git
7. Push the changes to your fork
8. Open a [`pull request`](https://docs.github.com/en/pull-requests). Give the pull request a descriptive title
   indicating what it changes. If it has a corresponding open issue, the issue number should be included in the title as
   well. For example a pull request that fixes issue `Bug: Increased stack size making it impossible to find needle #100`
   could be titled `Fix #100 - Make needles easier to find by applying fire to haystack`
9. Add yourself as a contributor to the relevant area using the [`all-contributors bot`](https://allcontributors.org/docs/en/bot/usage)


### Guidelines for writing code

- Code should be [`Pythonic and zen`](https://peps.python.org/pep-0020/)
- All code should be fully [`typed`](https://peps.python.org/pep-0484/) . This is enforced via
  [`mypy`](https://mypy.readthedocs.io/en/stable/) and [`pyright`](https://github.com/microsoft/pyright)

  * When requiring complex types, use a [`type alias`](https://docs.python.org/3/library/typing.html#type-aliases).
  * If something cannot be typed correctly due to a limitation of the type checkers, you may use
    [`typing.cast`](https://docs.python.org/3/library/typing.html#typing.cast) to rectify the situation. However, you
    should only use as a last resort if you've exhausted all other options of
    [`type narrowing`](https://mypy.readthedocs.io/en/stable/type_narrowing.html), such as
    [`isinstance()`](https://docs.python.org/3/library/functions.html#isinstance) checks and
    [`type guards`](https://docs.python.org/3/library/typing.html#typing.TypeGuard)
  * You may use `type: ignore` if you ensured that a line is correct, but [`mypy`](https://mypy.readthedocs.io/en/stable/) / [`pyright`](https://github.com/microsoft/pyright) has issues with it. Don't use
    blank `type: ignore` though, and instead supply the specific error code, e.g. `type: ignore[attr-defined]`

- If you are adding or modifying existing code, ensure that it's fully tested. 100% test coverage is mandatory, and will
  be checked on the PR using [`SonarCloud`](https://www.sonarsource.com/products/sonarcloud/)
- All functions, methods, classes and attributes should be documented with a docstring. We use the
  [`Google docstring style`](https://sphinxcontrib-napoleon.readthedocs.io/en/latest/example_google.html). If you come
  across a function or method that doesn't conform to this standard, please update it as you go

### Writing and running tests

Tests should be contained within the `tests` directory, and follow the same directory structure as the main package.
If you are adding a test case, it should be located within the correct submodule of `tests`. (e.g., tests for
`litestar/utils/sync.py` should go in in `tests/utils/test_sync.py`.)

If not, you are able to manually run `poetry run pytest` or `pytest tests/`

More information on `pytest` is available [here](https://docs.pytest.org/en/7.1.x/how-to/usage.html).

## Creating a new release
> This section applies to repository maintainers only.

1. Increment the version in `pyproject.toml` according to the
   [`versioning scheme`](https://litestar.dev/about/litestar-releases.html#version-numbering)
2. Draft a new release on GitHub in the repository

   * Use `vMAJOR.MINOR.PATCH` (e.g. `v1.2.3`) as both the tag and release title
   * Fill in the release description. You can use the "Generate release notes" function to get a draft for this

3. Update `CHANGELOG.rst` by adding a new section, with the version number as a heading. Include the contents of the
   release notes as they relate to changes in code
4. Commit your changes and push to `main`
5. Publish the release
6. Check that the "publish" action (Actions tab in the repository) has run successfully
