# DependentArgument Class

## Overview

`DependentArgument` is a custom `argparse.Action` class that allows you to create command-line arguments that depend on the presence of one or more other arguments. This is useful when you want to enforce logical dependencies between different options in your command-line interface.

## Installation

This class is not part of a package, so you'll need to include it directly in your project. Copy the `DependentArgument` class into your Python script or module.

## Usage

To use the `DependentArgument` class in your argparse-based CLI:

1. Import the necessary modules:
   ```python
   import argparse
   import warnings
   ```

2. Copy the `DependentArgument` class into your script.

3. When adding arguments to your `ArgumentParser`, use `DependentArgument` as the `action` for dependent arguments.

## Class Details

### DependentArgument

```python
class DependentArgument(argparse.Action):
    def __init__(self, option_strings, dest=None, required_args=None, **kwargs):
        # ...

    def __call__(self, parser, namespace, values, option_string=None):
        # ...
```

#### Parameters

- `option_strings`: List of option strings for the argument (e.g., `-d`, `--dependent`).
- `dest`: The name of the attribute to be added to the object returned by `parse_args()`.
- `required_args`: A list of argument names that this argument depends on. The dependent argument will be valid if any of these arguments are set.
- `**kwargs`: Additional keyword arguments passed to the `Action` constructor.

#### Behavior

- If any of the required arguments are set, the dependent argument's value is stored normally.
- If none of the required arguments are set, a warning is issued, and the dependent argument is ignored.

## Updated Implementation

Here's the updated `DependentArgument` class that supports multiple primary arguments:

```python
import argparse
import warnings

class DependentArgument(argparse.Action):
    """
    Custom argparse action to handle dependent arguments.

    This action allows an argument to be dependent on one or more other arguments.
    If any of the required arguments are set, this action will store the dependent argument's value.
    If none of the required arguments are set, it will ignore the dependent argument and issue a warning.

    Attributes:
        required_args (list): The names of the arguments that this argument depends on.
    """

    def __init__(self, option_strings, dest=None, required_args=None, **kwargs):
        """
        Initialize the DependentArgument action.

        Args:
            option_strings (list): The option strings for the argument.
            dest (str): The name of the attribute to be added to the object returned by parse_args().
            required_args (list): The names of the arguments that this argument depends on.
            **kwargs: Additional keyword arguments passed to the Action constructor.

        Raises:
            ValueError: If required_args is not provided or is not a list of strings.
        """
        if required_args is None or not isinstance(required_args, list) or not all(isinstance(arg, str) for arg in required_args):
            raise ValueError("required_args must be a list of strings")
        
        self.required_args = required_args
        super().__init__(option_strings, dest, **kwargs)

    def __call__(self, parser, namespace, values, option_string=None):
        """
        Called when the argument is encountered during argument parsing.

        If any of the required arguments are set, this method sets the value for the dependent argument.
        If none of the required arguments are set, it issues a warning and does not set the value.

        Args:
            parser (ArgumentParser): The ArgumentParser object.
            namespace (Namespace): The Namespace object that will be returned by parse_args().
            values: The value of the argument.
            option_string (str): The option string used to invoke this action.

        Raises:
            AttributeError: If any of the required_args are not valid argument names.
        """
        for arg in self.required_args:
            if not hasattr(namespace, arg):
                raise AttributeError(f"'{arg}' is not a valid argument name")
        
        if any(getattr(namespace, arg) for arg in self.required_args):
            setattr(namespace, self.dest, values)
        else:
            warnings.warn(f"Ignoring {option_string} because none of {', '.join(self.required_args)} are set.")

def create_parser():
    """
    Create and configure the argument parser.

    Returns:
        argparse.ArgumentParser: The configured argument parser.
    """
    parser = argparse.ArgumentParser(description="Example script with dependent arguments.")
    parser.add_argument('--primary1', action='store_true', help='First primary argument')
    parser.add_argument('--primary2', action='store_true', help='Second primary argument')
    parser.add_argument('--dependent', action=DependentArgument, required_args=['primary1', 'primary2'], 
                        help='This argument is only used if either --primary1 or --primary2 is set')
    return parser

def main():
    """
    Main function to demonstrate the use of dependent arguments.

    This function parses the command-line arguments and prints the results.
    """
    parser = create_parser()
    args = parser.parse_args()

    if args.primary1 or args.primary2:
        print("At least one primary argument is set.")
        if hasattr(args, 'dependent'):
            print(f"Dependent argument is: {args.dependent}")
        else:
            print("Dependent argument was not provided.")
    else:
        print("No primary arguments are set.")
        print("Dependent argument will be ignored even if provided.")

if __name__ == "__main__":
    main()
```

## Examples

### Basic Usage

```python
parser = argparse.ArgumentParser(description="Example with dependent arguments")
parser.add_argument('--primary1', action='store_true', help='First primary argument')
parser.add_argument('--primary2', action='store_true', help='Second primary argument')
parser.add_argument('--dependent', action=DependentArgument, required_args=['primary1', 'primary2'], 
                    help='This argument is only used if either --primary1 or --primary2 is set')

args = parser.parse_args()

if args.primary1 or args.primary2:
    print("At least one primary argument is set.")
    if hasattr(args, 'dependent'):
        print(f"Dependent argument is: {args.dependent}")
    else:
        print("Dependent argument was not provided.")
else:
    print("No primary arguments are set.")
    print("Dependent argument will be ignored even if provided.")
```

### Example Cases

1. Dependent argument with first primary argument:
   ```
   $ python script.py --primary1 --dependent value
   At least one primary argument is set.
   Dependent argument is: value
   ```

2. Dependent argument with second primary argument:
   ```
   $ python script.py --primary2 --dependent value
   At least one primary argument is set.
   Dependent argument is: value
   ```

3. Dependent argument with both primary arguments:
   ```
   $ python script.py --primary1 --primary2 --dependent value
   At least one primary argument is set.
   Dependent argument is: value
   ```

4. Only primary arguments provided:
   ```
   $ python script.py --primary1 --primary2
   At least one primary argument is set.
   Dependent argument was not provided.
   ```

5. Only dependent argument provided:
   ```
   $ python script.py --dependent value
   No primary arguments are set.
   Dependent argument will be ignored even if provided.
   Warning: Ignoring --dependent because none of primary1, primary2 are set.
   ```

6. No arguments provided:
   ```
   $ python script.py
   No primary arguments are set.
   Dependent argument will be ignored even if provided.
   ```

## Error Handling

- If `required_args` is not provided or is not a list of strings when initializing `DependentArgument`, a `ValueError` is raised.
- If any of the `required_args` do not correspond to valid argument names, an `AttributeError` is raised when the argument is processed.

## Notes

- This class uses warnings to notify about ignored dependent arguments. You may want to configure warning handling in your application.
- Make sure to handle the case where the dependent argument might not be set, even when one or more primary arguments are provided.

## Contributing

This is a standalone class, but feel free to modify and extend it to suit your specific needs. If you make improvements, consider sharing them with the community.

## License

This code is provided under the MIT License. Feel free to use, modify, and distribute it as needed.
