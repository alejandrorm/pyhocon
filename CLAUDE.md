# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pyhocon is a Python parser for HOCON (Human-Optimized Config Object Notation), the configuration format used by Typesafe Config. It provides parsing capabilities and conversion tools to transform HOCON files into JSON, YAML, and .properties formats.

## Development Commands

### Testing

```bash
# Run all tests with coverage
python setup.py test

# Run tests with pytest directly
pytest

# Run tests with coverage reporting
coverage run --source=pyhocon setup.py test
coverage report -m

# Run tests across multiple Python versions using tox
tox

# Run specific test file
pytest tests/test_config_parser.py

# Run linting with flake8
flake8 pyhocon tests setup.py
```

### Building and Installation

```bash
# Install in development mode
pip install -e .

# Install with duration support (includes python-dateutil)
pip install -e .[Duration]

# Build package
python setup.py sdist bdist_wheel
```

### Using the CLI Tool

```bash
# Convert HOCON to JSON
pyhocon -i input.conf -f json -o output.json

# Convert HOCON to YAML
pyhocon -i input.conf -f yaml

# Convert from stdin to stdout
cat input.conf | pyhocon -f json

# Use compact format for nested dictionaries
pyhocon -i input.conf -f hocon -c
```

## Architecture

### Core Components

1. **ConfigParser** (`pyhocon/config_parser.py`): The main parser that uses pyparsing to parse HOCON syntax into internal data structures. Contains the `ConfigFactory` class which provides static methods like `parse_file()`, `parse_string()`, and `parse_URL()`.

2. **ConfigTree** (`pyhocon/config_tree.py`): An OrderedDict subclass that represents the parsed configuration as a nested dictionary. Provides path-based access (e.g., `config['a.b.c']`) and type-safe getter methods (`get_string()`, `get_int()`, `get_bool()`, etc.).

3. **HOCONConverter** (`pyhocon/converter.py`): Handles conversion from ConfigTree to various output formats (JSON, YAML, .properties, HOCON). Each format has dedicated conversion methods (`to_json()`, `to_yaml()`, `to_properties()`, `to_hocon()`).

4. **Period Parser** (`pyhocon/period_parser.py`): Parses duration/period strings (e.g., "10 seconds", "5 days") into timedelta or relativedelta objects.

### Key Classes and Data Structures

- **ConfigTree**: The main configuration container supporting nested dictionary access and substitution resolution
- **ConfigValues**: Represents a value that may contain multiple tokens including literals and substitutions
- **ConfigSubstitution**: Represents a variable substitution like `${foo.bar}` or `${?OPTIONAL_VAR}`
- **ConfigInclude**: Represents an include directive for importing other configuration files
- **ConfigUnquotedString/ConfigQuotedString**: String value representations
- **ConfigList**: List values in the configuration

### Substitution System

The parser supports variable substitution with two types:
- **Required substitutions**: `${path}` - throws exception if not found
- **Optional substitutions**: `${?path}` - silently ignored if not found

Substitutions are resolved after parsing and can reference:
- Other config values within the same document
- Environment variables
- Values from included files

### Include System

Supports multiple include formats:
- Relative paths: `include "test.conf"`
- HTTP/HTTPS URLs: `include "https://example.com/config.conf"`
- File URLs: `include "file:///path/to/config.conf"`
- Package resources: `include package("package:assets/test.conf")`
- Required includes: `include required(file("test.conf"))`

Includes are processed during parsing, with relative paths resolved against the including file's directory.

## Testing Structure

Tests are organized by component:
- `test_config_parser.py`: Parser functionality, includes, substitutions
- `test_config_tree.py`: ConfigTree operations and merging
- `test_converter.py`: Format conversion tests
- `test_periods.py`: Duration/period parsing
- `test_tool.py`: CLI tool functionality

## Code Style

- Flake8 is used for linting with max line length of 160
- The project supports Python 2.7 and Python 3.4+
- Uses pyparsing for grammar definition and parsing
- Python 3.8+ compatibility requires special handling for pyparsing's `__deepcopy__`

## Important Notes

- The parser uses pyparsing which requires careful handling of parser element composition
- ConfigTree maintains insertion order (via OrderedDict) to preserve configuration key ordering
- The root ConfigTree maintains a history of value assignments for debugging purposes
- Nanosecond durations are converted to microseconds with reduced precision (divided by 1000)
- Month/year durations require python-dateutil to be installed
