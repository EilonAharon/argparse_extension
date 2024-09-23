# DependentArgument Class for argparse

## Overview

The `DependentArgument` class is a custom `argparse.Action` that allows you to create command-line arguments that depend on the presence of one or more other arguments. This is useful when you want to enforce logical dependencies between different options in your command-line interface, regardless of the order in which the arguments are provided.

## Features

- Create arguments that are only valid when certain other arguments are present
- Works with both optional and positional arguments
- Order-independent: dependent arguments can be specified before or after their required arguments
- Clear warning messages when dependent arguments are ignored
- Easy integration with existing argparse-based CLI applications

## Installation

The `DependentArgument` class is not part of a package, so you'll need to include it directly in your project. Copy the `DependentArgument` class into your Python script or module.

## Usage

### Basic Setup

1. Import the necessary modules:

```python
import argparse
import warnings
from dependent_argument import DependentArgument
```

2. Create your argument parser and add arguments as usual:

```python
parser = argparse.ArgumentParser(description="Example script with dependent arguments.")
parser.add_argument('--primary1', action='store_true', help='First primary argument')
parser.add_argument('--primary2', action='store_true', help='Second primary argument')
```

3. Add your dependent argument using the `DependentArgument` action:

```python
parser.add_argument('--dependent', action=DependentArgument, required_args=['primary1', 'primary2'], 
                    help='This argument is only used if either --primary1 or --primary2 is set')
```

4. Parse arguments and process dependencies:

```python
args = parser.parse_args()
DependentArgument.process_dependencies(args)
```

5. Use the arguments in your script:

```python
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

### Complete Example

Here's a complete example of how to use the `DependentArgument` class:

```python
import argparse
import warnings
from dependent_argument import DependentArgument

def create_parser():
    parser = argparse.ArgumentParser(description="Example script with dependent arguments.")
    parser.add_argument('--primary1', action='store_true', help='First primary argument')
    parser.add_argument('--primary2', action='store_true', help='Second primary argument')
    parser.add_argument('--dependent', action=DependentArgument, required_args=['primary1', 'primary2'], 
                        help='This argument is only used if either --primary1 or --primary2 is set')
    return parser

def main():
    parser = create_parser()
    args = parser.parse_args()
    DependentArgument.process_dependencies(args)

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

## Example Use Cases

1. Both primary and dependent arguments provided:
   ```
   $ python script.py --primary1 --dependent value
   At least one primary argument is set.
   Dependent argument is: value
   ```

2. Only primary argument provided:
   ```
   $ python script.py --primary1
   At least one primary argument is set.
   Dependent argument was not provided.
   ```

3. Only dependent argument provided:
   ```
   $ python script.py --dependent value
   No primary arguments are set.
   Dependent argument will be ignored even if provided.
   Warning: Ignoring --dependent because none of --primary1, --primary2 are set.
   ```

4. Arguments provided in different order:
   ```
   $ python script.py --dependent value --primary2
   At least one primary argument is set.
   Dependent argument is: value
   ```

## Notes

- The `DependentArgument` class uses warnings to notify about ignored dependent arguments. You may want to configure warning handling in your application.
- Make sure to call `DependentArgument.process_dependencies(args)` after `parse_args()` to ensure proper handling of dependent arguments.
- The class supports both optional and positional arguments as dependencies.

## Contributing

This is a standalone class, but feel free to modify and extend it to suit your specific needs. If you make improvements, consider sharing them with the community.

## License

This code is provided under the MIT License. Feel free to use, modify, and distribute it as needed.

## Contact

If you have any questions, issues, or suggestions, please open an issue in the GitHub repository or contact the maintainer.
