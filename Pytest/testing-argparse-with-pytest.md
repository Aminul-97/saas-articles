# How to Test Argparse CLI arguments with Pytest (CLIENT PROJECT)

Have you ever struggled with testing the Command-Line Interface (CLI) layer of your applications or scripts?

Perhaps you've build a robust application, with a database and REST API and interfaced via a CLI. 

You've [tested the database](https://pytest-with-eric.com/database-testing/pytest-sql-database-testing/) and [REST API](https://pytest-with-eric.com/pytest-advanced/pytest-fastapi-testing/), but what about the CLI?

How do you test that your code correctly handles missing CLI arguments, wrong data types, or invalid strings?

Command Line Arguments are a prime error candidate for errors, given their immense interaction with the end user. Hence, its crucial to ensure your application correctly processes user inputs and handles errors gracefully.

How do you do this without redefining all arguments in your tests? How do you abstract that layer?

<!-- more -->

The good news is Pytest has you covered. 

Whether you're using Python libraries like [Argparse](https://docs.python.org/3/library/argparse.html) or [Typer](https://typer.tiangolo.com/), Pytest provides a variety of methods to test CLI arguments.

Options include passing a list of command-line values, [parametrization](https://pytest-with-eric.com/introduction/pytest-parameterized-tests/), and [addoption](https://pytest-with-eric.com/pytest-advanced/pytest-addoption/).

This article will take you through different ways to test your CLI application using a real example to help you understand and apply the core concept.

We'll also discuss not just the HOW but also WHAT to test for CLI applications and share some best practices when testing command-line based applications.

So, let's begin.

[Example Code](https://github.com/Pytest-with-Eric/pytest-argument-testing-example.git)

## What You’ll Learn
This article will teach you to:
- Use Python command-line libraries like Typer and Argparse
- What to test for a CLI application.
- How to test CLI applications with Pytest.
- Automatically and manually pass arguments for tests.
- Abstract the CLI layer for testing.
- Essential best practices in testing CLI applications.
  

## Understanding Argparse
Before delving into the core topic, let's do a brief overview of the popular Python library - [Argparse](ttps://docs.python.org/3/library/argparse.html).

Argparse is a standard Python library dedicated to parsing command-line arguments. With Argparse, crafting robust and intuitive command-line interfaces is simplified.

This utility enables the parsing of command-line arguments, options and documentation effortlessly. 

Argparse helps you include several command-line parameters within a single option including subcommands.

However, it's important to note that Argparse requires explicit type conversion and validation for CLI arguments. 

While it does support the specification of argument types (e.g., `type=int`), users are required to manually manage type conversions and validation procedures.

Let's quickly look at a basic argparse CLI example,

`src/script_argparse.py`
```python
import argparse

def main():
    # Create an ArgumentParser object
    parser = argparse.ArgumentParser(description="A simple greeting application")

    # Add arguments
    parser.add_argument("--name", help="The name of the person")
    parser.add_argument("--age", type=int, help="The age of the person")

    # Parse the arguments from the command line
    args = parser.parse_args()


    # Access the argument and print the greeting
    if args.age:
        print(f"Hello, {args.name}! You are {args.age} years old.")
    else:
        print(f"Hello, {args.name}!")

if __name__ == "__main__":
    main()
```

Here we define a `main` function where we initiate an `ArgumentParser` object named `parser`. Using the `add_argument()` method, we define two arguments: `--name` and `--age`.

Upon invocation of the `parse_args()` method, the arguments `--name` and `--age` are parsed, and their respective values can be accessed using `args.name` and `args.age`.

Now, if you run the code following the below command:

```shell
python src/script_argparse.py --name=John --age=30
```

You'll have something like this,

![pytest-CLI-argument-test-example](/uploads/pytest-cli-argument-test-argparse-code.JPG "pytest-CLI-argument-test-example")

If the user needs help with arguments, they can easily generate a quick guide using the `--help` flag,

```shell
python src/script_argparse.py --help
```

You'll have the following output:

![pytest-CLI-argument-test-example](/uploads/pytest-cli-argument-test-argparse-help.JPG "pytest-CLI-argument-test-example")

However, there are few issues with Argparse.

Argparse lacks support for critical features to build robust CLI applications like automated type conversion, interactive prompts, colorful output, and an automatic help generator.

That's where a new library [Typer](https://typer.tiangolo.com/) comes in, containing all the features and functionalities required to create a highly interactive and user-friendly CLI application.

Typer was developed by the popular [Sebastián Ramírez](https://github.com/tiangolo), who's also the creator of [FastAPI](https://github.com/tiangolo/fastapi).

## Typer: An Argparse Alternative
[`typer`](https://typer.tiangolo.com/) is an advanced Python library tailored for crafting CLI applications. Renowned for its simplicity, developer and user-friendliness compared to Argparse, it emerges as the optimal modern choice for building robust CLI solutions.

The primary objective is to speed up CLI development, capitalizing on Python's type hints to enhance clarity and ease of use.

Key features of `typer` includes:

- **Decorator-Based API:** With `typer`, defining CLI commands is a breeze. By employing the `@typer.command()` decorator, any normal Python function effortlessly transforms into a CLI command.

- **Automatic Type Conversion:** `typer` excels in automatically converting arguments into specified Python types, leveraging Python's type hint system.

- **Interactive Prompts:** Enhancing user interaction, `typer` facilitates interactive prompts through functionalities like `typer.prompt()` and `typer.confirm()`, thereby elevating the user experience of the CLI application.

- **Colorful Output:** Leveraging vibrant colors and formatted output, `typer` elevates the visual appeal of CLI applications, ensuring a pleasant user experience.

- **Automatic Help Generation:** `typer` streamlines documentation efforts by automatically generating help messages and usage information based on function signatures and docstrings, reducing manual intervention and enhancing developer productivity.

If we develop a similar greeting program that we just created using Argparse, it will look like this:

```python
import typer

app = typer.Typer()

@app.command()
def greet(name: str, age: int = 0):
    """Greet someone."""
    if age:
        typer.echo(f"Hello, {name}! You are {age} years old.")
    else:
        typer.echo(f"Hello, {name}!")

if __name__ == "__main__":
    app()
```

As yo can see we've converted the method `greet()` to a CLI command through the docorator `@app.command()`.

Now, if you run the code following the below command:

```shell
python src/script_typer.py John --age=30 
```

You'll have something like this,

![pytest-CLI-argument-test-example](/uploads/pytest-cli-argument-test-typer-code.JPG "pytest-CLI-argument-test-example")

If the user needs help with arguments, they can easily generate a quick guide using the following command,

```shell
python src/script_typer.py --help
```

You'll have the following output:

![pytest-CLI-argument-test-example](/uploads/pytest-cli-argument-test-typer-help.JPG "pytest-CLI-argument-test-example")

Here is a brief note on arguments in general:

**Required Arguments:**
These are arguments for which no default value is specified. 

They must be provided by the user when invoking the CLI command. 

In the example provided, `name` is a required argument. 

This is because the function greet is defined with `name: str` without a default value, indicating that the CLI command must be called with a `name` value for it to execute properly.

**Optional Arguments:**
Optional arguments have a default value defined in the function signature. 

These arguments do not need to be explicitly provided by the user, as the default value will be used if the argument is omitted. 

In the code example, `age` is an optional argument because it is defined with a default value (age: int = 0). 

This means the user can omit the `age` argument when calling the command, and 0 will be used as the default age.

**Positional Arguments:**

In the context of Typer and CLI applications, arguments are considered positional based on the order they appear in the function definition. 

Typer expects these arguments in the order they are defined. 

Both `name` and `age` in the example are positional arguments from the CLI's perspective. 

**Overall**

In the provided example, `name` is a required argument because it lacks a default value. 

`age` is an optional argument because it has a default value (0). 

The user must provide `name` when invoking the greet command, but can choose to provide age either by its position or by using the `--age` flag.

## Argparse vs Typer
Argparse and Typer are both libraries designed for crafting command-line interfaces (CLIs), each with its own unique characteristics. Let's see which one is ideal for creating a CLI application.

1. **Defining CLI Arguments:** In Argparse, defining CLI arguments involves calling specific functions, whereas Typer simplifies this process by enabling the conversion of functions into CLI arguments using the `@typer.command()` decorator.
   
2. **Ease of Use:** Argparse has a more verbose syntax. In contrast, Typer has a concise and intuitive syntax, helping you create complex CLI applications with ease.

3. **Interactivity:** While Argparse primarily focuses on parsing command-line arguments and generating help messages, Typer stands out by supporting interactive prompts through functionalities such as `typer.prompt()` and `typer.confirm()`, helping you develop more interactive CLI applications.

4. **Colorful Output:** Argparse lacks support for colorful or formatted output, whereas Typer enhances the visual appeal of CLI applications by enabling colorful and formatted output to the terminal.

5. **Argument Type Conversion:** In Argparse, manual conversion of argument data types is required, whereas Typer automates this process, enabling automatic type conversion for arguments, thereby streamlining development tasks.

## Practical Example
Let's get started with a practical example.

### Prerequisites
Some basics of Python and Pytest would be helpful:
- Python (3.11+)
- Pytest

### Getting Started
Our example repo looks like this:

```shell
.  
├── .gitignore
├── README.md
├── pytest.ini
├── requirements.txt
├── tests
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_argparse_parametrization.py
│   ├── test_typer_parametrization.py
│   ├── test_yaml_reader_custom_cli.py
│   └── test_yaml_reader_list.py
└── src
    ├── yaml_configs
    │   └── config.yml
    ├── argparse_yaml_reader.py
    ├── script_argparse.py
    ├── script_typer.py
    └── typer_yaml_reader.py
```

We have a `src` folder containing the main code and a `tests` folder containing the test code. 

Our example code is a simple YAML reader that accepts a path to a YAML file as a command-line argument and reads the file.

To get started. clone the Github Repo [here](https://github.com/Pytest-with-Eric/pytest-argument-testing-example.git), or you can create your own repo by creating a folder and running `git init` to initialize it.

Create a virtual environment and install the required packages using the following command:

```shell
pip install -r requirements.txt
```

Feel free to use any package manager you wish.

## Example Code
Our example code is a basic YAML reader developed using the [Argparse](https://docs.python.org/3/library/argparse.html) and [Typer](https://typer.tiangolo.com/) libraries.

There are 2 functions - one to read the command-line argument (path to config file) and another to read the config file itself. 

Here's the YAML config file,

`src/yaml_configs/config.yml`
```yml
rest:
  url: "https://example.com/"
  port: 3001

dev:
  url: "https://dev.com/"
  port: 3010

prod:
  url: "https://prod.com/"
  port: 2007
```

The YAML file includes several API configurations.

### Argparse Example
First, we'll look at the YAML reader with the Argparse library.

`src/argparse_yaml_reader.py`
```python
import os
import yaml
from typing import Dict
import argparse
from argparse import ArgumentParser, ArgumentDefaultsHelpFormatter


def parse_args(args=None) -> ArgumentParser.parse_args:
    """
    Function to parse command line arguments

    Args:
    args: list of strings to parse

    Returns:
    parsed_args: parsed arguments
    """
    argument_parser = ArgumentParser(
        description="Command line arguments for reading a configuration file",
        formatter_class=ArgumentDefaultsHelpFormatter,
    )
    argument_parser.add_argument(
        "--configpath", type=str, help="Configuration file path", required=True)
    argument_parser.add_argument(
        "--env", type=str,default="rest", help="Select environment")
    return argument_parser.parse_args(args)


def yaml_reader(path: str, env:str) -> Dict:
    """
    Function to read YAML config file

    Args:
    path: path to the YAML file

    Returns:
    data: dictionary of data from the YAML file
    """
    try:
        with open(path, "r") as yamlfile:
            data = yaml.load(yamlfile, Loader=yaml.FullLoader)
            return data[env]
    except Exception as e:
        print(f"Error reading YAML file: {e}")


def main(args=None) -> None:
    """
    Main function to read YAML file
    """
    args = parse_args(args)
    configpath = args.configpath
    env = args.env

    if len(configpath) == 0:
        print("No path provided")
    else:
        if configpath and os.path.isfile(configpath):
            print(yaml_reader(path=configpath, env=env))
        else:
            print(
                f"`configpath` must be a valid file path. Provided path: `{configpath}` does not exist."
            )

if __name__ == "__main__":
    main()
```

The docstrings shares brief about the example code. We've bundled the core operations into the `main()` function.

### Typer Example
Let's write the same operation using Typer.

`src/typer_yaml_reader.py`
```python
import yaml
import typer
import os
app = typer.Typer()

@app.command()
def main(configpath: str, env:str = 'rest') -> None:
    """
    Main function to read YAML file

    Args:
    configpath: path to the YAML file

    Returns:
    None
    """
    if configpath and os.path.isfile(configpath):
        print(yaml_reader(configpath, env))
    else:
        print(
            f"`configpath` must be a valid file path. Provided path: `{configpath}` does not exist."
        )

def yaml_reader(path: str, env:str) -> None:
    """
    Function to read YAML config file
    """
    try:
        with open(path, "r") as yamlfile:
            data = yaml.load(yamlfile, Loader=yaml.FullLoader)
            return data[env]
    except Exception as e:
        print(f"Error reading YAML file: {e}")

if __name__ == "__main__":
    app()
```

You can straightaway see how much cleaner Typer is. 

Here, `app()` serves as the main function within the `typer` object.

## 3 Ways to Test Command-Line Arguments

We'll explore 3 different ways to test command-line arguments using Pytest. 

The first being a simple list-based testing, followed by automated parameterized testing, and finally manual testing with Pytest Addoption.

### List-Based Testing
This is the most basic way to test command-line arguments. 

You can pass arguments as a list to the main function. 

Here's what our test looks like

`tests/test_yaml_reader_list.py`
```python
from src.argparse_yaml_reader import main
from typer.testing import CliRunner
from src.typer_yaml_reader import app


def test_argparse_yaml_with_list(capsys):
    test_args = ['--configpath', 'src/yaml_configs/config.yml', '--env', 'rest'] 
    print(test_args)
    expected_output = "{'url': 'https://example.com/', 'port': 3001}"
    main(test_args)
    output = capsys.readouterr().out.rstrip()
    assert expected_output in output
```
Here, the test function tests the YAML reader implemented with Argparse.

The `capsys` fixture captures `stdout` and `stderr` output during the execution of test functions allowing you to access it and perform assertions against expected outputs.

If you're unfamiliar with `capsys` and it's different modes, we have you covered [here](https://pytest-with-eric.com/configuration/pytest-stdout/).

Now run the test:

```shell
pytest -v tests/test_yaml_reader_list.py
```

You'll have the following result:

![pytest-CLI-argument-test-example-result](/uploads/pytest-cli-argument-test-example-list-test-result-argparse.JPG "pytest-CLI-argument-test-example-result")

Now, let's test the Typer YAML reader using the same method.

`tests/test_yaml_reader_list.py`
```python
from typer.testing import CliRunner
from src.typer_yaml_reader import app

def test_typer_yaml_with_list():
    runner = CliRunner()
    test_args = ['src/yaml_configs/config.yml', '--env', 'rest'] 
    result = runner.invoke(app, test_args)
    assert result.exit_code == 0
    # Use result.stdout to access the command's output
    output = result.stdout.rstrip()
    expected_output = "{'url': 'https://example.com/', 'port': 3001}"
    assert expected_output in output
```

Typer provides a `CliRunner()` object to invoke the command and capture the output. 

When running the test:

```shell
pytest -v tests/test_yaml_reader_list.py
```

You'll have the following result:

![pytest-CLI-argument-test-example-result](/uploads/pytest-cli-argument-test-example-list-test-result-typer.JPG "pytest-CLI-argument-test-example-result")

### Parametrized Testing
Pytest parametrization allows you to run tests efficiently with multiple input data sets, eliminating the need for redundant test code. 

Using the decorator `@pytest.mark.parameterize`, you can specify the inputs and expected outputs for tests. 

You can use any type of value, such as numbers, strings, lists, or dictionaries.

For a deep exploration of Pytest Parameterization, check out this [comprehensiveguide](https://pytest-with-eric.com/introduction/pytest-parameterized-tests/).

Now, for our example code, we can parameterize tests as follows,

`tests/test_argparse_parametrization.py`
```python
from src.argparse_yaml_reader import main
import pytest
import shlex

test_cases = [
    (
        "--configpath='src/yaml_configs/config.yml'",  # Valid path without optional args
        "{'url': 'https://example.com/', 'port': 3001}",
    ),
    (
        "--configpath 'src/yaml_configs/config.yml' --env='dev'",  # Valid path with optional args
        "{'url': 'https://dev.com/', 'port': 3010}",
    ),
    (
        "--env='prod' --configpath 'src/yaml_configs/config.yml'",  # Different order
        "{'url': 'https://prod.com/', 'port': 2007}",
    ),
    (
        "--configpath 'src/config.yml' --env='dev'",  # Path doesn't exist 
        "`configpath` must be a valid file path. Provided path: `src/config.yml` does not exist.",
    ),
    (
        "--configpath ''",  # Null or None value passed 
        "No path provided",
    ),
    (
        "--configpath 'src/yaml_configs==config.yml'",  # Invalid path
        "`configpath` must be a valid file path. Provided path: `src/yaml_configs==config.yml` does not exist.",
    ),
    (
        "--configpath 'path/to/nonexistent/file.yml'",  # Nonexistent file
        "`configpath` must be a valid file path. Provided path: `path/to/nonexistent/file.yml` does not exist.",
    ),
]


@pytest.mark.parametrize("command, expected_output", test_cases)
def test_argparse_yaml_reader(capsys, command, expected_output):
    main(shlex.split(command))
    captured = capsys.readouterr()
    output = captured.out + captured.err
    assert expected_output in output

# Test cases
test_cases_sys_exit = [
    (
        "",  # No argument passed
        "the following arguments are required: --configpath",
    ),
    (
        "-configpath 'src/yaml_configs/config.yml' --env 'dev'",  # Wrong flag passed 
        "the following arguments are required: --configpath",
    ),
    (
        "configpath 'src/yaml_configs/config.yml' --env 'dev'",  # No flag passed 
        "the following arguments are required: --configpath",
    ),
    (
        "-+configpath 'src/yaml_configs/config.yml' --env 'dev'",  # Wrong Type of flag passed 
        "the following arguments are required: --configpath",
    ),
    (
        "--wrong_argument 'src/yaml_configs/config.yml' --env 'dev'",  # Wrong argument name
        "the following arguments are required: --configpath",
    ),
]


@pytest.mark.parametrize("command, expected_output", test_cases_sys_exit)
def test_argparse_yaml_reader_sys_exit(capsys, command, expected_output):
    with pytest.raises(SystemExit):  # Expecting SystemExit due to argparse error
        main(shlex.split(command))
    captured = capsys.readouterr()  # Capture both stdout and stderr
    output = captured.out + captured.err  # Combine stdout and stderr
    assert expected_output in output
```

Here, the list `test_cases` and `test_cases_sys_exit` contains a set of test cases for the Argparse YAML reader. We can perform the same thing using the Typer YAML reader as follows:

`tests/test_typer_parametrization.py`
```python
from typer.testing import CliRunner
from src.typer_yaml_reader import app
import pytest
import shlex

runner = CliRunner()

# Test cases with location and expected result
test_cases = [
    (
        "src/yaml_configs/config.yml",  # Valid path without optional args
        "{'url': 'https://example.com/', 'port': 3001}",
    ),
    (
        "src/yaml_configs/config.yml --env 'dev'",  # Valid path witho optional args
        "{'url': 'https://dev.com/', 'port': 3010}",
    ),
    (
        "--env 'prod' 'src/yaml_configs/config.yml'",  # Different order
        "{'url': 'https://prod.com/', 'port': 2007}",
    ),
    (
        "src/config.yml --env 'prod'",  # Path not exist 
        "`configpath` must be a valid file path. Provided path: `src/config.yml` does not exist.",
    ),
    (
        " ",  # Null or None value passed 
        "Missing argument",
    ),
    (
        "",  # No argument passed
        "Missing argument",
    ),
    (
        "'src/yaml_configs/config.yml' -env 'dev'",  # Invalid flag
        "No such option",
    ),
    (
        "src/yaml_configs==config.yml --env 'dev'",  # Invalid ascii character passsed
        "`configpath` must be a valid file path. Provided path: `src/yaml_configs==config.yml` does not exist.",
    ),
    (
        "path/to/nonexistent/file.yml --env 'dev'",  # Nonexistent file
        "`configpath` must be a valid file path. Provided path: `path/to/nonexistent/file.yml` does not exist.",
    ),
]

# Testing typer_yaml_reader()
@pytest.mark.parametrize("command, expected_output", test_cases)
def test_typer_yaml_reader(command, expected_output):
    result = runner.invoke(app, shlex.split(command))
    assert expected_output in result.stdout
```
Same as before, the variable `test_cases` stores a list of tests with expected output. Also the variable `runner` is the object of `CliRunner()`. 

The library method `shlex.split()` breaks the test case text based on spaces. 

When running the test:

```shell
pytest -v tests/test_argparse_parametrization.py tests/test_typer_parametrization.py
```

You'll have the following result:

![pytest-CLI-argument-test-example-result](/uploads/pytest-cli-argument-test-example-param-test-result.JPG "pytest-CLI-argument-test-example-result")

### Manual Testing with Pytest Addoption
Pytest Addoption allows you to define custom CLI arguments for tests. These arguments can be used to modify the behavior of your tests or pass configuration parameters to your test function or fixtures.

You can read more about [pytest addoption here](https://pytest-with-eric.com/pytest-advanced/pytest-addoption/).

The only downside of this strategy is that you have to redefine the arguments in your test suite, which ideally, is a layer you'd like to abstract.

In case of testing CLI applications, we will use this feature to pass arguments and expected outputs to our tests.

`tests/conftest.py`
```python
def pytest_addoption(parser):
    parser.addoption("--configpath", action="store", help="Location to YAML file")
    parser.addoption("--env", action="store", help="Environment to read from YAML file")
```

The `conftest.py` file contains the test arguments which are then processed by a fixture as follows,

`tests/test_yaml_reader_custom_cli.py`
```python
import pytest
import shlex
from src.argparse_yaml_reader import main, yaml_reader
from typer.testing import CliRunner
from src.typer_yaml_reader import app


# Fixture to get user inputs and find expected outputs
@pytest.fixture
def get_user_input(request):
    configpath = str(request.config.getoption("--configpath"))
    env = str(request.config.getoption("--env"))
    return configpath, env


# Testing argparse_yaml_reader()
def test_argparse_yaml_reader(capsys, get_user_input):
    configpath, env = get_user_input
    expected_output = str(yaml_reader(configpath, env))
    main(shlex.split("--configpath " + configpath + " --env " + env))
    output = capsys.readouterr().out.rstrip()
    assert expected_output in output


# Testing typer_yaml_reader()
def test_typer_yaml_reader(get_user_input):
    configpath, env = get_user_input
    expected_output = str(yaml_reader(configpath, env))
    runner = CliRunner()
    result = runner.invoke(app, shlex.split(configpath + " --env " + env))
    assert expected_output in result.stdout
```

Here, the `get_user_input()` fixture processes CLI arguments from `conftest.py` and passes them to tests `test_argparse_yaml_reader()` and `test_typer_yaml_reader()`.

When running the test you can provide arguments with the necessary value:

```shell
pytest -v tests/test_yaml_reader_custom_cli.py --yaml_location="src/yaml_configs/config.yml" --env="dev"
```

You can also include these options with `adopts` in the [`pytest.ini` file](https://pytest-with-eric.com/pytest-best-practices/pytest-ini/). This is an excellent approach when you have a lot of tests to run manually:

```ini
[pytest]
addopts = 
    --configpath="src/yaml_configs/config.yml" 
    --env="prod"
```

Then run it using:

```shell
pytest -v tests/test_yaml_reader_custom_cli.py
```

You'll have the following result:

![pytest-CLI-argument-test-example-result](/uploads/pytest-cli-argument-test-example-manual-test-result.JPG "pytest-CLI-argument-test-example-result")

## Running the test
We've already run tests separately for each testing method.

You can run all the test using this simple command:

```shell
pytest -v
```

After executing the command, you may have the following output:

![pytest-CLI-argument-test-example-result](/uploads/pytest-cli-argument-test-example-result.JPG "pytest-CLI-argument-test-example-result")

## What To Test - Strategy
Testing isn't just about perfect functionality, it's also about ensuring correct user interaction. Let's explore some essential strategies for effective application testing:

- **Test Incorrect Arguments:** Test the features of your application that handle incorrect arguments. We can create test cases to check invalid argument behavior as follows,
  
    ```python
    test_incorrect_args = ["--configpath", "/loc/config.py"]
    ```
    The above test case passes an invalid location to `--configpath`. Similarly you can also test for wrong CLI argument names.

- **Test Missing Arguments:** Test the feature of your application that handles missing arguments. The following test case demonstrates missing arguments,
  
    ```python
    test_missing_args = [" "]
    ```

- **Test Datatypes:** Check if your application validates the argument datatypes correctly. The following test case illustrates the wrong data types for the test,
  
    ```python
    test_incorrect_args = ["--configpath", 10]
    ```
    Here, the argument `--configpath` requires a string but provides an integer (e.g. 10).

- **Test Null Values:** Test if your application can handle null values or empty strings. You can create a test case for null values like this.
    
    ```python
    test_incorrect_args = ["--configpath", ""]
    ```

- **Test Typing Mistakes:** Sometimes users can make typing mistakes (e.g., Typing `--config` instead of `--configpath`). Make sure the application can handle this kind of mistake. 

    ```python
    test_args = ["--config", "src/yaml_configs/config.yml"]
    ```

- **Test Command-Line Interface (CLI) Visibility:** Test the CLI's visible arguments and features to ensure smooth user interaction. Verify that users can easily understand and interact with the CLI commands and correct documentation is displayed when `--help` is invoked.

- **Create Effective Test Cases:** To fully assess the application's functionality and robustness, create test cases that include a variety of inputs, starting states, end states, and error conditions.

## Best Practices for Testing CLI Arguments
Let’s unwrap some tips and tricks to follow while testing CLI arguments,

**Parametrized Tests:**
Employ parameterized tests to cover various input scenarios with different command-line argument combinations. This approach aids in uncovering unexpected behavior or edge cases that your CLI application may encounter. If you can, use [property-based testing tools like Hypothesis](https://pytest-with-eric.com/pytest-advanced/hypothesis-testing-python/).

**Error Handling:**
Thoroughly test error handling and exception scenarios to validate the handling of invalid CLI arguments. Ensure error messages, handlers, and exit codes align with expectations.

**Use configuration files:**
If you attempt to run a test manually, use configuration files like `pytest.ini`, `tox.ini`, `setup.cfg`, and `pyproject.toml`. If you're unfamiliar with them, go through this [comprehensive guide](https://pytest-with-eric.com/configuration/pytest-config-file/). This approach reduces the need to pass CLI commands manually on each run eliminating possibility for errors. 

**User Focused Test Cases:**
The test cases should be from the user's perspective. Design test cases that demonstrate common mistakes and errors like typos, empty arguments, wrong data types, and more. This practice ensures that the application behaves as expected when critical errors occur. 

**Prioritize Feature Testing:** 
Develop a strategic approach to prioritize feature tests based on their significance, such as new features, core functionalities, security measures, reporting capabilities, and other advanced features. This approach ensures efficient resource allocation and focuses on critical aspects.

## Wrapping Up
That's all about testing CLI applications.

This article provides quick insights into testing CLI applications built with popular Python libraries such as Argparse and Typer with Pytest.

Illustrated through a practical example, you explored the practical aspects of testing CLI applications using diverse methods like lists, parametrization, and adoption.

You also learned about the best practices for testing CLI applications, including error handling, user-focused test cases, and feature prioritization.

My recommendation - depends on your use case but I like to use a combination of list based arguments with parameterization.

For manual testing with different arguments, Pytest addoption is a nice choice.


Till the next time… Cheers!

## Additional Readings
[Example Code](https://github.com/Pytest-with-Eric/pytest-argument-testing-example.git)  
[The Ultimate Guide To Capturing Stdout/Stderr Output In Pytest](https://pytest-with-eric.com/configuration/pytest-stdout/)  
[How to Effortlessly Generate Unit Test Cases with Pytest Parameterized Tests](https://pytest-with-eric.com/introduction/pytest-parameterized-tests/)  
[How To Use Pytest With Command Line Options (Easy To Follow Guide)](https://pytest-with-eric.com/pytest-advanced/pytest-addoption/)  
[What Is `pytest.ini` And How To Save Time Using Pytest Config](https://pytest-with-eric.com/pytest-best-practices/pytest-ini/)  
[Pytest Config Files - A Practical Guide To Good Config Management](https://pytest-with-eric.com/configuration/pytest-config-file/)  
[Typer](https://typer.tiangolo.com/)  
[Testing argparse Applications](https://pythontest.com/testing-argparse-apps/)  
[Build Command-Line Interfaces With Python's argparse](https://realpython.com/command-line-interfaces-python-argparse/#:~:text=This%20module%20was%20released%20as,parameters%20in%20a%20single%20option)  
[Python: Better CLIs with Typer](https://pravash-techie.medium.com/python-better-clis-with-typer-a8783fafec6c#:~:text=Rather%20than%20specifying%20valid%20CLI,based%20on%20python%20type%20hints.)  