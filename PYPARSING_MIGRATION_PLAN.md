# Pyparsing 3.x Migration Plan

## Executive Summary

This document outlines the plan to migrate pyhocon from pyparsing 2.x to the latest pyparsing 3.x release. The primary changes involve updating deprecated camelCase method and parameter names to PEP8-compliant snake_case equivalents.

**Current Version:** `pyparsing>=2,<4` (supports both 2.x and 3.x)
**Target Version:** `pyparsing>=3.0.0` (3.1+ recommended for latest features)
**Python Support:** Drop Python 2.7, require Python 3.6.8+ (aligned with pyparsing 3.x)

## Impact Analysis

### Files Affected
1. `pyhocon/config_parser.py` - **HIGH IMPACT** (primary parser implementation)
2. `pyhocon/period_parser.py` - **MEDIUM IMPACT** (period/duration parsing)
3. `pyhocon/config_tree.py` - **LOW IMPACT** (minimal pyparsing usage)
4. `tests/test_config_parser.py` - **LOW IMPACT** (exception handling)
5. `setup.py` - **MEDIUM IMPACT** (version constraints)
6. `tox.ini` - **MEDIUM IMPACT** (remove Python 2.7 support)

### Breaking Changes Required
- Update pyparsing version constraint in `setup.py`
- Remove Python 2.7 from classifiers and tox.ini
- Update all deprecated method calls
- Update all deprecated parameter names
- Remove Python 3.8+ deepcopy workaround (may no longer be needed in pyparsing 3.1+)

## Detailed Migration Steps

### Phase 1: Update Dependencies and Python Version Support

#### 1.1 Update `setup.py`

**Current:**
```python
install_requires=[
    'pyparsing~=2.0;python_version<"3.0"',
    'pyparsing>=2,<4;python_version>="3.0"',
],
classifiers=[
    ...
    'Programming Language :: Python :: 2',
    'Programming Language :: Python :: 2.7',
    'Programming Language :: Python :: 3',
    'Programming Language :: Python :: 3.4',
    'Programming Language :: Python :: 3.5',
    ...
]
```

**Updated:**
```python
install_requires=[
    'pyparsing>=3.0.0',
],
classifiers=[
    ...
    'Programming Language :: Python :: 3',
    'Programming Language :: Python :: 3.7',
    'Programming Language :: Python :: 3.8',
    'Programming Language :: Python :: 3.9',
    'Programming Language :: Python :: 3.10',
    'Programming Language :: Python :: 3.11',
    'Programming Language :: Python :: 3.12',
]
```

**Note:** Remove Python 3.4-3.6 as they are EOL. Pyparsing 3.0+ requires Python 3.6.8+.

#### 1.2 Update `tox.ini`

**Current:**
```ini
envlist = flake8, py{27,38,39,310,311,312}
```

**Updated:**
```ini
envlist = flake8, py{37,38,39,310,311,312,313}
```

#### 1.3 Remove Python 2.x Compatibility Code

**Files to update:**
- `pyhocon/config_parser.py`: Remove `basestring` and `unicode` compatibility checks
- `pyhocon/config_tree.py`: Remove `basestring` compatibility
- `pyhocon/converter.py`: Remove `basestring` compatibility

**Pattern to remove:**
```python
try:
    basestring
except NameError:
    basestring = str
    unicode = str
```

**Note:** These can be removed entirely since Python 3 is required.

### Phase 2: Update Method Names in `pyhocon/config_parser.py`

#### 2.1 Import Statement Updates

**Current:**
```python
from pyparsing import (Forward, Group, Keyword, Literal, Optional,
                       ParserElement, ParseSyntaxException, QuotedString,
                       Regex, SkipTo, StringEnd, Suppress, TokenConverter,
                       Word, ZeroOrMore, alphanums, alphas8bit, col, lineno,
                       replaceWith)
```

**Updated:**
```python
from pyparsing import (Forward, Group, Keyword, Literal, Optional,
                       ParserElement, ParseSyntaxException, QuotedString,
                       Regex, SkipTo, StringEnd, Suppress, TokenConverter,
                       Word, ZeroOrMore, alphanums, alphas8bit, col, lineno,
                       replace_with)
```

**Change:** `replaceWith` → `replace_with`

#### 2.2 Method Call Updates

**Lines to update (approximate line numbers from analysis):**

| Line | Current Code | Updated Code |
|------|--------------|--------------|
| 379 | `ParserElement.setDefaultWhitespaceChars(' \t')` | `ParserElement.set_default_whitespace_chars(' \t')` |
| 381 | `ParserElement.setDefaultWhitespaceChars(default)` | `ParserElement.set_default_whitespace_chars(default)` |
| 385 | `Keyword("true", caseless=True).setParseAction(replaceWith(True))` | `Keyword("true", case_insensitive=True).set_parse_action(replace_with(True))` |
| 386 | `Keyword("false", caseless=True).setParseAction(replaceWith(False))` | `Keyword("false", case_insensitive=True).set_parse_action(replace_with(False))` |
| 387 | `Keyword("null", caseless=True).setParseAction(replaceWith(NoneValue()))` | `Keyword("null", case_insensitive=True).set_parse_action(replace_with(NoneValue()))` |
| 388 | `QuotedString('"""', escChar='\\', unquoteResults=False)` | `QuotedString('"""', esc_char='\\', unquote_results=False)` |
| 389 | `QuotedString('"', escChar='\\', unquoteResults=False)` | `QuotedString('"', esc_char='\\', unquote_results=False)` |
| 397 | `Regex(...).setParseAction(convert_number)` | `Regex(...).set_parse_action(convert_number)` |
| 400 | `Regex(...).setParseAction(parse_multi_string)` | `Regex(...).set_parse_action(parse_multi_string)` |
| 402 | `Regex(...).setParseAction(create_quoted_string)` | `Regex(...).set_parse_action(create_quoted_string)` |
| 408 | `Regex(...).setParseAction(unescape_string)` | `Regex(...).set_parse_action(unescape_string)` |
| 410 | `Regex(...).setParseAction(create_substitution)` | `Regex(...).set_parse_action(create_substitution)` |
| 420 | `Keyword("include", caseless=True)` | `Keyword("include", case_insensitive=True)` |
| 425 | `).setParseAction(include_config)` | `).set_parse_action(include_config)` |
| 454 | `config_expr.parseString(content, parseAll=True)` | `config_expr.parse_string(content, parse_all=True)` |

#### 2.3 Constant Name Updates

**Current (around line 378):**
```python
default = ParserElement.DEFAULT_WHITE_CHARS
```

**Check if this constant name has changed.** It may now be `DEFAULT_WHITESPACE_CHARS` or `default_whitespace_chars`. Need to verify in latest pyparsing docs.

### Phase 3: Update `pyhocon/period_parser.py`

#### 3.1 Method Call Updates

**Line 67:**
```python
# Current
).setParseAction(convert_period)

# Updated
).set_parse_action(convert_period)
```

**Line 71:**
```python
# Current
return get_period_expr().parseString(content, parseAll=True)[0]

# Updated
return get_period_expr().parse_string(content, parse_all=True)[0]
```

### Phase 4: Update `tests/test_config_parser.py`

#### 4.1 Exception Import Updates

**Line 19:**
```python
# Current
from pyparsing import ParseBaseException, ParseException, ParseSyntaxException

# Check if these exception names have changed in pyparsing 3.x
# Most likely they remain the same, but verify
```

**Note:** Exception names typically don't follow PEP8 conventions as they are class names (PascalCase is standard for classes).

### Phase 5: Remove Python 3.8+ Workaround (if applicable)

#### 5.1 Check if Deepcopy Issue is Resolved

**Current code (lines 19-30 in config_parser.py):**
```python
# Fix deepcopy issue with pyparsing
if sys.version_info >= (3, 8):
    def fixed_get_attr(self, item):
        if item == '__deepcopy__':
            raise AttributeError(item)
        try:
            return self[item]
        except KeyError:
            return ""

    pyparsing.ParseResults.__getattr__ = fixed_get_attr
```

**Action:** Test if this workaround is still needed with pyparsing 3.1+. If the issue is resolved upstream, remove this code block entirely.

### Phase 6: Update Other Potential Usages

#### 6.1 Search for Additional Deprecated Patterns

Run these searches to find any missed occurrences:

```bash
# Search for camelCase method calls
grep -rn "\.setParseAction\|\.parseString\|\.setDefaultWhitespaceChars\|\.scanString\|\.searchString" pyhocon/
grep -rn "\.setResultsName\|\.setDebug\|\.setName\|\.parseFile" pyhocon/

# Search for deprecated parameters
grep -rn "parseAll\|escChar\|unquoteResults\|caseless" pyhocon/

# Search for deprecated helper functions
grep -rn "replaceWith\|upcaseTokens\|downcaseTokens" pyhocon/
```

## Testing Strategy

### Phase 1: Setup Test Environment
1. Create a new virtual environment
2. Install pyparsing 3.x: `pip install 'pyparsing>=3.0.0'`
3. Install pyhocon in development mode: `pip install -e .`

### Phase 2: Run Existing Tests
```bash
# Run all tests
pytest tests/

# Run with coverage
coverage run --source=pyhocon setup.py test
coverage report -m
```

### Phase 3: Test Each Migration Incrementally
1. Make changes to one file at a time
2. Run tests after each file update
3. Fix any breakage immediately before proceeding

### Phase 4: Regression Testing
1. Test with sample HOCON files from `samples/` directory
2. Test CLI tool: `pyhocon -i samples/database.conf -f json`
3. Test all conversion formats: JSON, YAML, properties, HOCON
4. Test include functionality
5. Test substitution functionality
6. Test period/duration parsing

### Phase 5: Compatibility Testing
Test against multiple Python versions using tox:
```bash
tox -e py37,py38,py39,py310,py311,py312
```

## Rollback Strategy

1. Create a new branch for migration: `git checkout -b pyparsing-3-migration`
2. Commit each phase separately with clear commit messages
3. If issues arise, can revert specific commits
4. Keep original version constraints as fallback
5. Tag the last working 2.x-compatible version before merging

## Risk Assessment

### Low Risk
- Method and parameter renames (backward compatible in pyparsing 3.0-3.2)
- Import statement updates

### Medium Risk
- Removing Python 2.7 compatibility code (thorough testing required)
- Updating version constraints (users must upgrade)

### High Risk
- Behavioral changes in pyparsing 3.x (e.g., ZeroOrMore, counted_array)
- Removing Python 3.8+ workaround (may break on older pyparsing versions)

## Migration Timeline (Estimated)

1. **Phase 1** (Dependencies): 30 minutes
2. **Phase 2** (config_parser.py): 1-2 hours
3. **Phase 3** (period_parser.py): 15 minutes
4. **Phase 4** (tests): 30 minutes
5. **Phase 5** (workaround removal): 30 minutes + testing
6. **Phase 6** (cleanup): 30 minutes
7. **Testing**: 2-3 hours
8. **Documentation**: 30 minutes

**Total Estimated Time:** 6-9 hours

## Post-Migration Tasks

1. Update CHANGELOG.md with migration notes
2. Update README.md if needed (installation instructions)
3. Add migration notes for users in release notes
4. Consider updating CLAUDE.md with new pyparsing information
5. Update CI/CD pipelines if they reference Python 2.7
6. Create GitHub issue/discussion announcing the change

## Success Criteria

- [ ] All tests pass with pyparsing 3.x
- [ ] No deprecation warnings when running with pyparsing 3.3+
- [ ] All CLI tool functions work correctly
- [ ] Tox tests pass for all supported Python versions
- [ ] Code follows PEP8 naming conventions
- [ ] Documentation is updated
- [ ] Migration is backward compatible with recent pyparsing 3.x versions

## References

- [Pyparsing 3.0.0 What's New](https://pyparsing-docs.readthedocs.io/en/latest/whats_new_in_3_0_0.html)
- [Pyparsing API Documentation](https://pyparsing-docs.readthedocs.io/en/latest/pyparsing.html)
- [Pyparsing PEP-8 Planning](https://github.com/pyparsing/pyparsing/wiki/PEP-8-planning)
- [Pyparsing Releases](https://github.com/pyparsing/pyparsing/releases)
