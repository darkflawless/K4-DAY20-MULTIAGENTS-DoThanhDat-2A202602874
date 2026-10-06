### Human
The test suite of the `inventory` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Assistant
I'll start by reading the required skills, then explore the workspace.

### Tool call: read_file
{"file_path": "/skills/enumerate-required-artifacts-before-coding/SKILL.md", "limit": 1000}

### Tool call: read_file
{"file_path": "/skills/satisfy-house-rules-not-just-tests/SKILL.md", "limit": 1000}

### Tool result
@@ lines 1-30 of 30 @@
---
name: enumerate-required-artifacts-before-coding
description: Use at the start of any task that produces files or code changes, to list every required deliverable and house rule before writing any code.
---
Before writing or editing anything, build an explicit checklist of every required output and rule. Do not start coding until this list exists.

## Steps
1. Read the task instruction and any `README.md` in the workspace end to end.
2. If a `check.py` / checker / test harness exists in the task directory, read it — it encodes the exact expected schema, field names, sorting, and conventions. Prefer it over guessing.
3. Extract and write down, as a literal checklist:
   - Every file that must exist (exact path and name).
   - Every required field / column / key and its exact type and format.
   - Every house rule stated (naming, sorting, units, headers, changelog, tests).
   - Any "do not modify" constraints (e.g. existing tests).
4. Keep the checklist visible and tick items off as you go.

## Common required artifacts to look for
- A specific output file (e.g. `answer.json`, `errors.json`, `clean.csv`) — not just "an answer".
- A `meta` / metadata block with source file name, row counts, etc.
- Regression tests in a specific file, one test per fix.
- A `CHANGELOG.md` entry under a specific heading with a specific bullet format.
- Type annotations on all public functions.

## Anti-patterns to avoid
- Assuming "tests pass" means the task is done.
- In

### Tool result
@@ lines 1-24 of 24 @@
---
name: satisfy-house-rules-not-just-tests
description: Use on code-fix tasks where passing tests is necessary but not sufficient, to ensure changelog, regression tests, type hints, and no-modify rules are all met.
---
Passing the existing test suite is the floor, not the goal. House rules are graded separately and fail independently.

## Rules to satisfy on every code-fix task
1. Do not modify existing tests. New test files are allowed; editing files under `tests/` that already exist is not.
2. Add regression tests: create the required file (e.g. `tests/test_regressions.py`) with one test function per bug fixed, meeting the minimum count. Run it and confirm it passes.
3. Add type annotations to every public function (name not starting with `_`): all parameters and the return value.
4. Update the changelog: add one bullet per fix under the required heading (e.g. `## Unreleased`) in the exact format (e.g. `- fix(<function name>): <short description>`), meeting the minimum bullet count.

## Workflow
1. Fix the source so behavior matches docstrings/spec.
2. Write the regression tests and run them.
3. Add type hints and re-run the full suite.
4. Update the changelog.
5. Run the task's own checker if present; otherwise re-read the instruction and tick each rule.

## Anti-patterns
- Verifying only with a throwaway `_check.py` and the pre-existing suite.
- Fixing behavior but leaving functions untyped.
- Forgetting the changelog or writing bullets in the wro

### Tool call: ls
{"path": "/workspace"}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/inventory/', '/workspace/tests/']

### Tool call: read_file
{"file_path": "/workspace/README.md"}

### Tool call: read_file
{"file_path": "/workspace/CHANGELOG.md"}

### Tool call: ls
{"path": "/workspace/inventory"}

### Tool call: ls
{"path": "/workspace/tests"}

### Tool result
@@ lines 1-5 of 5 @@
# inventory

Small package. Run the tests from the `workspace/` folder:

    cd workspace && python -m pytest tests -q

### Tool result
@@ lines 1-6 of 6 @@
# Changelog

## Unreleased

## 0.3.1
- initial import

### Tool result
['/workspace/inventory/__init__.py', '/workspace/inventory/__pycache__/', '/workspace/inventory/export.py', '/workspace/inventory/pricing.py', '/workspace/inventory/report.py']

### Tool result
['/workspace/tests/__pycache__/', '/workspace/tests/test_report.py']

### Tool call: read_file
{"file_path": "/workspace/inventory/__init__.py"}

### Tool call: read_file
{"file_path": "/workspace/inventory/export.py"}

### Tool call: read_file
{"file_path": "/workspace/inventory/pricing.py"}

### Tool call: read_file
{"file_path": "/workspace/inventory/report.py"}

### Tool call: read_file
{"file_path": "/workspace/tests/test_report.py"}

### Tool result
@@ lines 1-1 @@
System reminder: File exists but has empty contents

### Tool result
@@ lines 1-12 of 12 @@
"""Export helpers."""
from .pricing import parse_price


def to_csv_row(item):
    """Format one item as a CSV row: name,price,qty (price with 2 decimals).

    A name that contains a comma or a double quote is wrapped in double quotes,
    and every double quote inside it is doubled (RFC 4180).
    """
    price = parse_price(item["price"])
    return f'{item["name"]},{price:.2f},{item["qty"]}'

### Tool result
@@ lines 1-25 of 25 @@
"""Price utilities for the inventory package."""
from decimal import Decimal


def parse_price(text):
    """Convert a price string to a Decimal.

    Supported formats:
      "$1,299.50" -> Decimal("1299.50")
      "  12 "     -> Decimal("12")
      "1299.5"    -> Decimal("1299.5")
      "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
    """
    cleaned = text.strip().replace("$", "")
    return Decimal(cleaned)


def apply_discount(price, percent):
    """Return `price` after a percentage discount.

    The result is a Decimal rounded to 2 decimal places using the
    commercial rule "round half up" (0.005 -> 0.01).
    """
    discounted = price * (100 - percent) / 100
    return discounted.quantize(Decimal("0.01"))

### Tool result
@@ lines 1-25 of 25 @@
"""Inventory reports."""
from decimal import Decimal

from .pricing import apply_discount, parse_price


def total_value(items):
    """Total value of `items` after per-item discounts.

    Each item is a dict: {"name": str, "price": str, "qty": int, "discount": percent (optional)}.
    """
    total = Decimal("0")
    for item in items:
        price = parse_price(item["price"])
        price = apply_discount(price, item.get("discount", 0))
        total += price * item["qty"]
    return total


def low_stock(items, threshold=5):
    """Names of the items whose quantity is strictly below `threshold`.

    The names are sorted alphabetically, ignoring case.
    """
    return [item["name"] for item in items if item["qty"] <= threshold]

### Tool result
@@ lines 1-32 of 32 @@
from decimal import Decimal

from inventory.export import to_csv_row
from inventory.pricing import apply_discount
from inventory.report import low_stock, total_value


def test_total_value_plain():
    items = [{"name": "pen", "price": "$2.50", "qty": 4}]
    assert total_value(items) == Decimal("10.00")


def test_total_value_thousands_separator():
    items = [{"name": "laptop", "price": "$1,299.50", "qty": 2}]
    assert total_value(items) == Decimal("2599.00")


def test_discount_rounding_half_up():
    assert apply_discount(Decimal("10.05"), 10) == Decimal("9.05")


def test_total_value_with_discount():
    items = [{"name": "desk", "price": "100.00", "qty": 1, "discount": 15}]
    assert total_value(items) == Decimal("85.00")


def test_low_stock_basic():
    assert low_stock([{"name": "pen", "qty": 2}, {"name": "ink", "qty": 9}]) == ["pen"]


def test_csv_row_plain_name():
    assert to_csv_row({"name": "pen", "price": "2.5", "qty": 4}) == "pen,2.50,4"

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q 2>&1 | tail -40"}

### Tool result
    self.pluginmanager.load_setuptools_entrypoints("pytest11")
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\pluggy\_manager.py", line 416, in load_setuptools_entrypoints
    plugin = ep.load()
             ^^^^^^^^^
  File "~\AppData\Local\Programs\Python\Python312\Lib\importlib\metadata\__init__.py", line 205, in load
    module = import_module(match.group('module'))
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "~\AppData\Local\Programs\Python\Python312\Lib\importlib\__init__.py", line 90, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "<frozen importlib._bootstrap>", line 1387, in _gcd_import
  File "<frozen importlib._bootstrap>", line 1360, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1331, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 935, in _load_unlocked
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\_pytest\assertion\rewrite.py", line 188, in exec_module
    exec(co, module.__dict__)
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\anyio\pytest_plugin.py", line 305, in <module>
    @pytest.fixture(scope="module", params=get_available_backends())
                                           ^^^^^^^^^^^^^^^^^^^^^^^^
  File "~\Desktop\06-10 AI

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q -p no:anyio 2>&1 | tail -40"}

### Tool result
  File "<frozen importlib._bootstrap>", line 1360, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1331, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 935, in _load_unlocked
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\_pytest\assertion\rewrite.py", line 188, in exec_module
    exec(co, module.__dict__)
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\langsmith\_openapi_client\types\__init__.py", line 5, in <module>
    from .run import Run as Run
  File "<frozen importlib._bootstrap>", line 1360, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1331, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 935, in _load_unlocked
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\_pytest\assertion\rewrite.py", line 188, in exec_module
    exec(co, module.__dict__)
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\langsmith\_openapi_client\types\run.py", line 7, in <module>
    from .._models import BaseModel
  File "<frozen importlib._bootstrap>", line 1360, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1331, in _find_and_load_unlocked
  File "<frozen importlib._bootstrap>", line 935, in _load_unlocked
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q -p no:cacheprovider -p no:langsmith 2>&1 | tail -20; echo \"---\"; python -m pytest tests -q --co 2>&1 | tail -5"}

### Tool result
tail: option used in invalid context -- 2

[Command succeeded with exit code 0]

### Tool call: execute
{"command": "cd workspace && python -m pytest tests -q -p no:cacheprovider 2>&1 | head -60"}

### Tool result
Traceback (most recent call last):
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\anyio\_core\_eventloop.py", line 204, in get_async_backend
    return loaded_backends[asynclib_name]
           ~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^
KeyError: 'asyncio'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "<frozen runpy>", line 198, in _run_module_as_main
  File "<frozen runpy>", line 88, in _run_code
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\pytest\__main__.py", line 9, in <module>
    raise SystemExit(_console_main())
                     ^^^^^^^^^^^^^^^
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\_pytest\config\__init__.py", line 253, in _console_main
    code = _main(prog=_get_prog_name(sys.argv))
           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\_pytest\config\__init__.py", line 223, in _main
    config = _prepareconfig(new_args, plugins, prog=prog)
             ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
  File "~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\.venv\Lib\site-packages\_pytest\config\__init__.py", line 410, in _prepareconfig
    config: Config = pluginmanager.hook.pytest_cmdline_parse(
                

### Tool call: execute
{"command": "cd workspace && PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests -q 2>&1 | head -60"}

### Tool result
<no output>

Exit code: 255
[Command failed with exit code 255]

### Tool call: execute
{"command": "cd workspace && PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests -q; echo \"EXIT=$?\""}

### Tool result
[stderr] 'PYTEST_DISABLE_PLUGIN_AUTOLOAD' is not recognized as an internal or external command,
[stderr] operable program or batch file.

Exit code: 1
[Command failed with exit code 1]

### Tool call: execute
{"command": "cd workspace && set PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 && python -m pytest tests -q & echo EXIT=%ERRORLEVEL%"}

### Tool result
.FF...                                                                   [100%]
================================== FAILURES ===================================
____________________ test_total_value_thousands_separator _____________________

    def test_total_value_thousands_separator():
        items = [{"name": "laptop", "price": "$1,299.50", "qty": 2}]
>       assert total_value(items) == Decimal("2599.00")
               ^^^^^^^^^^^^^^^^^^

~\Desktop\06-10 AI in action\K4-DAY20-MULTIAGENTS-DoThanhDat-2A202602874\tasks\code-learn\workspace\tests\test_report.py:15: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
inventory\report.py:14: in total_value
    price = parse_price(item["price"])
            ^^^^^^^^^^^^^^^^^^^^^^^^^^
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _

text = '$1,299.50'

    def parse_price(text):
        """Convert a price string to a Decimal.
    
        Supported formats:
          "$1,299.50" -> Decimal("1299.50")
          "  12 "     -> Decimal("12")
          "1299.5"    -> Decimal("1299.5")
          "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
        """
        cleaned = text.strip().replace("$", "")
>       return Decimal(cleaned)
               ^^^^^^^^^^^^^^^^
E       decimal.InvalidOperation: [<class 'decimal.ConversionSyntax'>]

inventory\pricing.py:15: InvalidOperation
_______________________ test_discount_rounding_half_up __________

### Tool call: execute
{"command": "cd workspace && python -c \"import sys; print(sys.version)\" && python -c \"import pytest; print(pytest.__version__)\""}

### Tool result
3.12.10 (tags/v3.12.10:0cc8128, Apr  8 2025, 12:21:36) [MSC v.1943 64 bit (AMD64)]
9.1.1

[Command succeeded with exit code 0]

### Tool call: execute
{"command": "cd workspace && python -c \"\nfrom decimal import Decimal\nprint(Decimal('10.05')*(100-10)/100)\nprint(Decimal('10.05')*Decimal(90)/Decimal(100))\nprint((Decimal('10.05')*Decimal(90)/Decimal(100)).quantize(Decimal('0.01')))\n\""}

### Tool result
<no output>
[Command succeeded with exit code 0]

### Tool call: execute
{"command": "cd workspace && python -c \"from decimal import Decimal; print(Decimal('10.05')*(100-10)/100); print(Decimal('10.05')*Decimal(90)/Decimal(100))\""}

### Tool result
9.045
9.045

[Command succeeded with exit code 0]

### Tool call: execute
{"command": "cd workspace && python -c \"from decimal import Decimal; print(Decimal('9.045').quantize(Decimal('0.01'))); print(Decimal('9.045').quantize(Decimal('0.01'), rounding='ROUND_HALF_UP'))\""}

### Tool result
9.04
9.05

[Command succeeded with exit code 0]

### Assistant
Now let me write the fixes.

### Tool call: write_file
{"file_path": "/workspace/inventory/pricing.py", "content": "\"\"\"Price utilities for the inventory package.\"\"\"\nfrom decimal import ROUND_HALF_UP, Decimal\n\n_CENTS = Decimal(\"0.01\")\n\n\ndef parse_price(text: str) -> Decimal:\n    \"\"\"Convert a price string to a Decimal.\n\n    Supported formats:\n      \"$1,299.50\" -> Decimal(\"1299.50\")\n      \"  12 \"     -> Decimal(\"12\")\n      \"1299.5\"    -> Decimal(\"1299.5\")\n      \"(12.00)\"   -> Decimal(\"-12.00\")   # accounting style: parentheses mean negative\n    \"\"\"\n    cleaned = text.strip().replace(\"$\", \"\").replace(\",\", \"\")\n    negative = cleaned.startswith(\"(\") and cleaned.endswith(\")\")\n    if negative:\n        cleaned = cleaned[1:-1].strip()\n    value = Decimal(cleaned)\n    return -value if negative else value\n\n\ndef apply_discount(price: Decimal, percent: float) -> Decimal:\n    \"\"\"Return `price` after a percentage discount.\n\n    The result is a Decimal rounded to 2 decimal places using the\n    commercial rule \"round half up\" (0.005 -> 0.01).\n    \"\"\"\n    discounted = price * (Decimal(100) - Decimal(str(percent))) / Decimal(100)\n    return discounted.quantize(_CENTS, rounding=ROUND_HALF_UP)\n"}

### Tool result
Updated file /workspace/inventory/pricing.py

### Tool call: write_file
{"content": "\"\"\"Inventory reports.\"\"\"\nfrom decimal import Decimal\nfrom typing import Any, Iterable, Mapping\n\nfrom .pricing import apply_discount, parse_price\n\n\ndef total_value(items: Iterable[Mapping[str, Any]]) -> Decimal:\n    \"\"\"Total value of `items` after per-item discounts.\n\n    Each item is a dict: {\"name\": str, \"price\": str, \"qty\": int, \"discount\": percent (optional)}.\n    \"\"\"\n    total = Decimal(\"0\")\n    for item in items:\n        price = parse_price(item[\"price\"])\n        price = apply_discount(price, item.get(\"discount\", 0))\n        total += price * item[\"qty\"]\n    return total\n\n\ndef low_stock(items: Iterable[Mapping[str, Any]], threshold: int = 5) -> list[str]:\n    \"\"\"Names of the items whose quantity is strictly below `threshold`.\n\n    The names are sorted alphabetically, ignoring case.\n    \"\"\"\n    names = [item[\"name\"] for item in items if item[\"qty\"] < threshold]\n    return sorted(names, key=str.lower)\n", "file_path": "/workspace/inventory/report.py"}

### Tool result
Updated file /workspace/inventory/report.py

### Tool call: write_file
{"content": "\"\"\"Export helpers.\"\"\"\nfrom typing import Any, Mapping\n\nfrom .pricing import parse_price\n\n\ndef to_csv_row(item: Mapping[str, Any]) -> str:\n    \"\"\"Format one item as a CSV row: name,price,qty (price with 2 decimals).\n\n    A name that contains a comma or a double quote is wrapped in double quotes,\n    and every double quote inside it is doubled (RFC 4180).\n    \"\"\"\n    price = parse_price(item[\"price\"])\n    name = item[\"name\"]\n    if \",\" in name or '\"' in name:\n        name = '\"' + name.replace('\"', '\"\"') + '\"'\n    return f\"{name},{price:.2f},{item['qty']}\"\n", "file_path": "/workspace/inventory/export.py"}

### Tool result
Updated file /workspace/inventory/export.py

### Tool call: execute
{"command": "cd workspace && set PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 && python -m pytest tests -q"}

### Tool result
......                                                                   [100%]
6 passed in 0.02s

[Command succeeded with exit code 0]

### Assistant
Now the regression tests:

### Tool call: write_file
{"content": "\"\"\"Regression tests for the bugs fixed in the inventory package.\"\"\"\nfrom decimal import Decimal\n\nfrom inventory.export import to_csv_row\nfrom inventory.pricing import apply_discount, parse_price\nfrom inventory.report import low_stock, total_value\n\n\ndef test_parse_price_strips_thousands_separator():\n    assert parse_price(\"$1,299.50\") == Decimal(\"1299.50\")\n\n\ndef test_parse_price_accounting_parentheses_are_negative():\n    assert parse_price(\"(12.00)\") == Decimal(\"-12.00\")\n\n\ndef test_parse_price_plain_and_padded_values():\n    assert parse_price(\"  12 \") == Decimal(\"12\")\n    assert parse_price(\"1299.5\") == Decimal(\"1299.5\")\n\n\ndef test_apply_discount_rounds_half_up():\n    assert apply_discount(Decimal(\"10.05\"), 10) == Decimal(\"9.05\")\n\n\ndef test_apply_discount_rounds_half_up_on_exact_half():\n    assert apply_discount(Decimal(\"0.05\"), 50) == Decimal(\"0.03\")\n\n\ndef test_total_value_with_thousands_separator():\n    items = [{\"name\": \"laptop\", \"price\": \"$1,299.50\", \"qty\": 2}]\n    assert total_value(items) == Decimal(\"2599.00\")\n\n\ndef test_low_stock_is_strictly_below_threshold():\n    items = [{\"name\": \"pen\", \"qty\": 5}, {\"name\": \"ink\", \"qty\": 4}]\n    assert low_stock(items) == [\"ink\"]\n\n\ndef test_low_stock_sorted_case_insensitively():\n    items = [{\"name\": \"Zebra\", \"qty\": 1}, {\"name\": \"apple\", \"qty\": 2}]\n    assert low_stock(items) == [\"apple\", \"Zebra\"]\n\n\ndef test_

### Tool result
Updated file /workspace/tests/test_regressions.py

### Tool call: execute
{"command": "cd workspace && set PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 && python -m pytest tests -q"}

### Tool result
................                                                         [100%]
16 passed in 0.10s

[Command succeeded with exit code 0]

### Tool call: edit_file
{"file_path": "/workspace/CHANGELOG.md", "old_string": "## Unreleased\n", "new_string": "## Unreleased\n- fix(parse_price): strip thousands separators and treat parentheses as negative\n- fix(apply_discount): round half up instead of banker's rounding\n- fix(low_stock): use a strict \"below threshold\" comparison and sort case-insensitively\n- fix(to_csv_row): quote names containing commas or double quotes per RFC 4180\n"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/CHANGELOG.md'

### Tool call: execute
{"command": "cd workspace && set PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 && python -m pytest tests -q && python -c \"import inventory.export, inventory.pricing, inventory.report; print('ok')\""}

### Tool result
................                                                         [100%]
16 passed in 0.14s
ok

[Command succeeded with exit code 0]

### Tool call: read_file
{"file_path": "/workspace/CHANGELOG.md"}

### Tool result
@@ lines 1-10 of 10 @@
# Changelog

## Unreleased
- fix(parse_price): strip thousands separators and treat parentheses as negative
- fix(apply_discount): round half up instead of banker's rounding
- fix(low_stock): use a strict "below threshold" comparison and sort case-insensitively
- fix(to_csv_row): quote names containing commas or double quotes per RFC 4180

## 0.3.1
- initial import

### Assistant
All done. Checklist verified: source fixed to match docstrings, existing tests untouched, new regression tests added, type hints on all public functions, changelog updated.

## Summary

**Bugs fixed (source only, `tests/test_report.py` untouched):**

1. **`inventory/pricing.py` → `parse_price`** — did not strip thousands separators (`"$1,299.50"` raised `InvalidOperation`) and ignored accounting-style parentheses. Now removes `,` and maps `"(12.00)"` → `Decimal("-12.00")`.
2. **`inventory/pricing.py` → `apply_discount`** — used the default `ROUND_HALF_EVEN` context, so `10.05` at 10% gave `9.04`. Now quantizes with `ROUND_HALF_UP` (and converts `percent` via `Decimal(str(...))` to avoid float artifacts).
3. **`inventory/report.py` → `low_stock`** — used `<=` (docstring says *strictly below*) and returned items in input order. Now uses `<` and sorts case-insensitively.
4. **`inventory/export.py` → `to_csv_row`** — never quoted names. Now wraps names containing a comma or double quote in quotes and doubles inner quotes (RFC 4180).

**House rules:**
- Added type annotations to every public function (`parse_price`, `apply_discount`, `total_value`, `low_stock`, `to_csv_row`).
- Added `tests/test_regressions.py` with 10 regression tests (one or more per fix); full suite: **16 passed**.
- Added four `- fix(<function>): <description>` bullets under `## Unreleased` in `CHANGELOG.md`.

Note: the sandbox's pytest auto-loads a broken `anyio` plugin, so I ran the suite with `PYTEST_DISABL