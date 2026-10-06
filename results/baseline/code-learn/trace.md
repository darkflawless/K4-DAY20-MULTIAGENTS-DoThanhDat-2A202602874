### Human
The test suite of the `inventory` package in the folder `workspace/` is failing.

Fix the source code so that the whole test suite passes and every function does what its docstring says.
The visible tests do not cover everything: the docstrings are the specification.
Do not modify the existing files in `tests/`.
Your changes are also checked by Acme's review bot against the Acme Python team conventions.

When you are done, reply with a short summary of what you changed and why.


### Assistant
I'll start by exploring the workspace structure.

### Tool call: ls
{"path": "workspace"}

### Tool call: glob
{"pattern": "workspace/**/*.py"}

### Tool result
['/workspace/CHANGELOG.md', '/workspace/README.md', '/workspace/inventory/', '/workspace/tests/']

### Tool result
['/workspace/inventory/__init__.py', '/workspace/inventory/export.py', '/workspace/inventory/pricing.py', '/workspace/inventory/report.py', '/workspace/tests/test_report.py']

### Tool call: read_file
{"file_path": "workspace/README.md"}

### Tool call: read_file
{"file_path": "workspace/CHANGELOG.md"}

### Tool call: read_file
{"file_path": "workspace/inventory/__init__.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/export.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/pricing.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/report.py"}

### Tool call: read_file
{"file_path": "workspace/tests/test_report.py"}

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
{"command": "cd workspace && python -m pytest tests -q -p no:cacheprovider -p no:langsmith 2>&1 | tail -20; echo \"---\"; python -c \"import sys; print(sys.version)\""}

### Tool result
[stderr] tail: option used in invalid context -- 2

Exit code: 1
[Command failed with exit code 1]

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

### Assistant
Now I can run the tests. Let me analyze the bugs against the docstrings:

1. **`parse_price`**: doesn't handle thousands separators (`$1,299.50`) or accounting-style parentheses (`(12.00)` → negative).
2. **`apply_discount`**: uses default `ROUND_HALF_EVEN` instead of `ROUND_HALF_UP`.
3. **`low_stock`**: uses `<=` instead of strictly `<`, and doesn't sort case-insensitively.
4. **`to_csv_row`**: doesn't quote names containing commas/quotes per RFC 4180.

Let me fix them.

### Tool call: edit_file
{"file_path": "workspace/inventory/pricing.py", "old_string": "\"\"\"Price utilities for the inventory package.\"\"\"\nfrom decimal import Decimal\n\n\ndef parse_price(text):\n    \"\"\"Convert a price string to a Decimal.\n\n    Supported formats:\n      \"$1,299.50\" -> Decimal(\"1299.50\")\n      \"  12 \"     -> Decimal(\"12\")\n      \"1299.5\"    -> Decimal(\"1299.5\")\n      \"(12.00)\"   -> Decimal(\"-12.00\")   # accounting style: parentheses mean negative\n    \"\"\"\n    cleaned = text.strip().replace(\"$\", \"\")\n    return Decimal(cleaned)\n\n\ndef apply_discount(price, percent):\n    \"\"\"Return `price` after a percentage discount.\n\n    The result is a Decimal rounded to 2 decimal places using the\n    commercial rule \"round half up\" (0.005 -> 0.01).\n    \"\"\"\n    discounted = price * (100 - percent) / 100\n    return discounted.quantize(Decimal(\"0.01\"))", "new_string": "\"\"\"Price utilities for the inventory package.\"\"\"\nfrom decimal import ROUND_HALF_UP, Decimal\n\n_CENTS = Decimal(\"0.01\")\n\n\ndef parse_price(text):\n    \"\"\"Convert a price string to a Decimal.\n\n    Supported formats:\n      \"$1,299.50\" -> Decimal(\"1299.50\")\n      \"  12 \"     -> Decimal(\"12\")\n      \"1299.5\"    -> Decimal(\"1299.5\")\n      \"(12.00)\"   -> Decimal(\"-12.00\")   # accounting style: parentheses mean negative\n    \"\"\"\n    cleaned = text.strip().replace(\"$\", \"\").replace(\",\", \"\")\n    negative = cleaned.startswith(\"(\") and cleaned.end

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/inventory/pricing.py'

### Tool call: edit_file
{"file_path": "workspace/inventory/report.py", "new_string": "def low_stock(items, threshold=5):\n    \"\"\"Names of the items whose quantity is strictly below `threshold`.\n\n    The names are sorted alphabetically, ignoring case.\n    \"\"\"\n    names = [item[\"name\"] for item in items if item[\"qty\"] < threshold]\n    return sorted(names, key=str.lower)", "old_string": "def low_stock(items, threshold=5):\n    \"\"\"Names of the items whose quantity is strictly below `threshold`.\n\n    The names are sorted alphabetically, ignoring case.\n    \"\"\"\n    return [item[\"name\"] for item in items if item[\"qty\"] <= threshold]"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/inventory/report.py'

### Tool call: edit_file
{"file_path": "workspace/inventory/export.py", "new_string": "def to_csv_row(item):\n    \"\"\"Format one item as a CSV row: name,price,qty (price with 2 decimals).\n\n    A name that contains a comma or a double quote is wrapped in double quotes,\n    and every double quote inside it is doubled (RFC 4180).\n    \"\"\"\n    price = parse_price(item[\"price\"])\n    name = item[\"name\"]\n    if \",\" in name or '\"' in name:\n        name = '\"{}\"'.format(name.replace('\"', '\"\"'))\n    return f\"{name},{price:.2f},{item['qty']}\"", "old_string": "def to_csv_row(item):\n    \"\"\"Format one item as a CSV row: name,price,qty (price with 2 decimals).\n\n    A name that contains a comma or a double quote is wrapped in double quotes,\n    and every double quote inside it is doubled (RFC 4180).\n    \"\"\"\n    price = parse_price(item[\"price\"])\n    return f'{item[\"name\"]},{price:.2f},{item[\"qty\"]}'"}

### Tool result
Successfully replaced 1 instance(s) of the string in '/workspace/inventory/export.py'

### Assistant
Now let me verify with the test suite plus extra checks against the docstrings.

### Tool call: execute
{"command": "cd workspace && set PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 && python -m pytest tests -q & echo EXIT=%ERRORLEVEL%"}

### Tool result
......                                                                   [100%]
6 passed in 0.01s
EXIT=0

[Command succeeded with exit code 0]

### Tool call: execute
{"command": "cd workspace && python -c \"\nfrom decimal import Decimal\nfrom inventory.pricing import parse_price, apply_discount\nfrom inventory.report import low_stock, total_value\nfrom inventory.export import to_csv_row\n\nassert parse_price('\\$1,299.50') == Decimal('1299.50')\nassert parse_price('  12 ') == Decimal('12')\nassert parse_price('1299.5') == Decimal('1299.5')\nassert parse_price('(12.00)') == Decimal('-12.00')\nassert parse_price('(\\$1,000.00)') == Decimal('-1000.00')\nassert apply_discount(Decimal('10.05'), 10) == Decimal('9.05')\nassert apply_discount(Decimal('0.005'), 0) == Decimal('0.01')\nassert apply_discount(Decimal('100'), 15) == Decimal('85.00')\nassert low_stock([{'name':'Pen','qty':2},{'name':'apple','qty':1},{'name':'ink','qty':5}]) == ['apple','Pen']\nassert low_stock([{'name':'a','qty':5}], threshold=5) == []\nassert to_csv_row({'name':'pen','price':'2.5','qty':4}) == 'pen,2.50,4'\nassert to_csv_row({'name':'a,b','price':'1','qty':1}) == '\\\"a,b\\\",1.00,1'\nassert to_csv_row({'name':'a\\\"b','price':'1','qty':1}) == '\\\"a\\\"\\\"b\\\",1.00,1'\nassert total_value([{'name':'x','price':'(10.00)','qty':2}]) == Decimal('-20.00')\nprint('all docstring checks passed')\n\""}

### Tool result
<no output>
[Command succeeded with exit code 0]

### Tool call: execute
{"command": "cd workspace && python -c \"print('hello')\""}

### Tool result
hello

[Command succeeded with exit code 0]

### Tool call: write_file
{"file_path": "workspace/_check.py", "content": "from decimal import Decimal\n\nfrom inventory.export import to_csv_row\nfrom inventory.pricing import apply_discount, parse_price\nfrom inventory.report import low_stock, total_value\n\nassert parse_price(\"$1,299.50\") == Decimal(\"1299.50\")\nassert parse_price(\"  12 \") == Decimal(\"12\")\nassert parse_price(\"1299.5\") == Decimal(\"1299.5\")\nassert parse_price(\"(12.00)\") == Decimal(\"-12.00\")\nassert parse_price(\"($1,000.00)\") == Decimal(\"-1000.00\")\n\nassert apply_discount(Decimal(\"10.05\"), 10) == Decimal(\"9.05\")\nassert apply_discount(Decimal(\"0.005\"), 0) == Decimal(\"0.01\")\nassert apply_discount(Decimal(\"100\"), 15) == Decimal(\"85.00\")\n\nassert low_stock([{\"name\": \"Pen\", \"qty\": 2}, {\"name\": \"apple\", \"qty\": 1}, {\"name\": \"ink\", \"qty\": 5}]) == [\"apple\", \"Pen\"]\nassert low_stock([{\"name\": \"a\", \"qty\": 5}], threshold=5) == []\n\nassert to_csv_row({\"name\": \"pen\", \"price\": \"2.5\", \"qty\": 4}) == \"pen,2.50,4\"\nassert to_csv_row({\"name\": \"a,b\", \"price\": \"1\", \"qty\": 1}) == '\"a,b\",1.00,1'\nassert to_csv_row({\"name\": 'a\"b', \"price\": \"1\", \"qty\": 1}) == '\"a\"\"b\",1.00,1'\n\nassert total_value([{\"name\": \"x\", \"price\": \"(10.00)\", \"qty\": 2}]) == Decimal(\"-20.00\")\n\nprint(\"all docstring checks passed\")\n"}

### Tool result
Updated file /workspace/_check.py

### Tool call: execute
{"command": "cd workspace && python _check.py"}

### Tool result
all docstring checks passed

[Command succeeded with exit code 0]

### Assistant
All docstring-derived checks pass. Let me clean up the temporary check file and do a final review of the changed sources.

### Tool call: delete
{"file_path": "workspace/_check.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/pricing.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/report.py"}

### Tool call: read_file
{"file_path": "workspace/inventory/export.py"}

### Tool result
Deleted /workspace/_check.py

### Tool result
@@ lines 1-31 of 31 @@
"""Price utilities for the inventory package."""
from decimal import ROUND_HALF_UP, Decimal

_CENTS = Decimal("0.01")


def parse_price(text):
    """Convert a price string to a Decimal.

    Supported formats:
      "$1,299.50" -> Decimal("1299.50")
      "  12 "     -> Decimal("12")
      "1299.5"    -> Decimal("1299.5")
      "(12.00)"   -> Decimal("-12.00")   # accounting style: parentheses mean negative
    """
    cleaned = text.strip().replace("$", "").replace(",", "")
    negative = cleaned.startswith("(") and cleaned.endswith(")")
    if negative:
        cleaned = cleaned[1:-1]
    value = Decimal(cleaned)
    return -value if negative else value


def apply_discount(price, percent):
    """Return `price` after a percentage discount.

    The result is a Decimal rounded to 2 decimal places using the
    commercial rule "round half up" (0.005 -> 0.01).
    """
    discounted = price * (100 - percent) / 100
    return discounted.quantize(_CENTS, rounding=ROUND_HALF_UP)

### Tool result
@@ lines 1-26 of 26 @@
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
    names = [item["name"] for item in items if item["qty"] < threshold]
    return sorted(names, key=str.lower)

### Tool result
@@ lines 1-15 of 15 @@
"""Export helpers."""
from .pricing import parse_price


def to_csv_row(item):
    """Format one item as a CSV row: name,price,qty (price with 2 decimals).

    A name that contains a comma or a double quote is wrapped in double quotes,
    and every double quote inside it is doubled (RFC 4180).
    """
    price = parse_price(item["price"])
    name = item["name"]
    if "," in name or '"' in name:
        name = '"{}"'.format(name.replace('"', '""'))
    return f"{name},{price:.2f},{item['qty']}"

### Tool call: execute
{"command": "cd workspace && set PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 && python -m pytest tests -q & echo EXIT=%ERRORLEVEL%"}

### Tool result
......                                                                   [100%]
6 passed in 0.01s
EXIT=0

[Command succeeded with exit code 0]

### Assistant
All 6 tests pass and the docstring-specified behavior is verified.

## Summary

I fixed three source files in `workspace/inventory/` (no test files were touched):

**`inventory/pricing.py`**
- `parse_price`: now strips thousands separators (`,`) so `"$1,299.50"` parses, and handles accounting-style parentheses by returning a negative `Decimal` (`"(12.00)"` → `Decimal("-12.00")`), as the docstring specifies.
- `apply_discount`: now quantizes with `ROUND_HALF_UP` (via a module-level `_CENTS` constant) instead of the default banker's rounding, so `0.005` rounds up to `0.01` as documented.

**`inventory/report.py`**
- `low_stock`: changed the comparison from `<=` to `<` (docstring says *strictly* below the threshold) and now sorts the names case-insensitively with `sorted(..., key=str.lower)`.

**`inventory/export.py`**
- `to_csv_row`: now applies RFC 4180 quoting — a name containing a comma or a double quote is wrapped in double quotes and internal quotes are doubled.

These changes make the visible suite pass (6/6) and align every function with its docstring, including the cases the tests don't cover (parenthesized negatives, thousands separators, half-up rounding, strict threshold, case-insensitive sorting, and CSV quoting).