---
layout: post
title: "Using Python and Bash to Print Daily Crypto Prices"
date: "2025-01-18 10:00:00 +0000"
tags: [python, coding, bash, crypto]
# excerpt: "NONE"
---

I went through a phase of copying down the daily crypto open prices for several crypto currencies (BTC, ETH, LINK, DOT, ADA). This forced me to pay more attention to the crypto markets when I was following it more closely. I would go to binance.com and manually find each card, find the daily open, and copy it down. This was a bit tedious and annoying, so I decided to automate the process using the `requests` python library and Binance's api (`url = 'https://api.binance.com/api/v3/klines'`). I put the `crypto_simple.py` script in a `bash` script and added it to the zsh path so it could be called globally. The result was something like this:

```bash
$ crypto

rices - Current
Symbol      Price (USDT)
------------------------
BTC           $103337.79
ETH             $3295.88
LINK              $24.17
DOT                $7.02
ADA                $1.06
```

I've since extended the functionality with `argparse` allowing the user (me) to update the amount of each currency, stored locally in `wallet.json`, and print the user wallet value, and print the all-time-high value for each crypto currency. This TIL post explains how the code works. [N.B. The code is copied and explained below, but it is also stored at this repo: [seamusbarnes/crypto-prices](https://github.com/seamusbarnes/crypto-prices), and may be developed further there).

## crypto_simple.py

```python
import requests

def get_binance_price(symbol):
    url = f"https://api.binance.com/api/v3/ticker/price?symbol={symbol}"
    response = requests.get(url)
    data = response.json()

    return data['price']

def display_prices(symbols, unit):
    align = {'left': 8, 'right': 15}

    print('\nPrices - Current')
    header = f"{'Symbol'.ljust(align['left'])} {'Price (USDT)'.rjust(align['right'])}"
    print(header)
    print('-' * len(header))

    for symbol in symbols:
        price = get_binance_price(f'{symbol}{unit}')
        print(f"{symbol.ljust(align['left'])} {f'${float(price):.2f}'.rjust(align['right'])}")

    return

if __name__ == "__main__":

    unit = 'USDT'
    symbols = ['BTC', 'ETH', 'LINK', 'DOT', 'ADA']

    display_prices(symbols, unit)
```

This code uses the request URL to send a HTTP GET request to Binance's API, checks for response errors, and parses the JSON data to extract the return price (type: str) and return the price as a float. It then displays the price using the `display_prices` function.

## cyrpto_complete.py

The "complete" version of the code uses `argparse` to allow the user to specify which functionality to implement. Ran in base mode without arguments yields the following output:

```bash
$ python crypto_complete.py

--- Prices ---
Symbol    Open Price  Current Price  Change (%)   Wallet Value
--------------------------------------------------------------
BTC       $104077.47     $103591.61      -0.47%          $0.00
ETH         $3473.64       $3275.21      -5.71%          $0.00
LINK          $25.10         $23.72      -5.50%          $0.00
DOT            $7.50          $6.94      -7.56%          $0.00
ADA            $1.13          $1.06      -6.21%          $0.00
```

It's quite nice! It gives the open price, current price and the current intraday change %, as well as the wallet value for each currency if the user has added the amount of any currency to the wallet. The user can update the wallet, show all the prices with wallet value, and show just the wallet currencies:

```bash
$ python crypto_complete.py -wallet update BTC 1.0
Updated wallet: BTC = 1.0

$ python crypto_complete.py -wallet update ETH 5.0
Updated wallet: ETH = 5.0

$ python crypto_complete.py -wallet update DOT 42.0
Updated wallet: DOT = 42.0

$ python crypto_complete.py

--- Prices ---
Symbol    Open Price  Current Price  Change (%)   Wallet Value
--------------------------------------------------------------
BTC       $104077.47     $104062.59      -0.01%     $104062.59
ETH         $3473.64       $3284.31      -5.45%      $16421.55
LINK          $25.10         $23.77      -5.30%          $0.00
DOT            $7.50          $6.95      -7.38%        $291.90
ADA            $1.13          $1.07      -5.94%          $0.00
--------------------------------------------------------------
Total Wallet Value:                                 $120776.04

$ python crypto_complete.py -wallet show

--- Prices ---
Symbol    Open Price  Current Price  Change (%)   Wallet Value
--------------------------------------------------------------
BTC       $104077.47     $104020.29      -0.05%     $104020.29
ETH         $3473.64       $3276.38      -5.68%      $16381.90
DOT            $7.50          $6.94      -7.46%        $291.65
--------------------------------------------------------------
Total Wallet Value:                                 $120693.84
```

I'm rich! (as long as I'm not fudging the wallet numbers...). You can also set the flag `-ath` which shows you exactly how much worse off you are now because you didn't sell at the all-time-high:

```bash
$ python crypto_complete.py -ath

--- All-Time Highs ---
Symbol              ATH  Current Price Change from ATH (%)
----------------------------------------------------------
BTC          $108353.00     $104007.03              -4.01%
ETH            $4868.00       $3272.90             -32.77%
LINK             $53.00         $23.71             -55.26%
DOT              $55.09          $6.94             -87.41%
ADA               $3.10          $1.06             -65.74%
```

The code is as follows:

```python
#!/usr/bin/env python3

import argparse
import json
from pathlib import Path
import requests

class Wallet:
    """Handles wallet loading and saving."""
    def __init__(self, file_path):
        self.file_path = Path(file_path)
        self.data = self._load()

    def _load(self):
        """Load wallet data from a file."""
        if self.file_path.exists():
            with self.file_path.open('r') as file:
                return json.load(file)
        return {}

    def save(self):
        """Save wallet data to the file."""
        with self.file_path.open('w') as file:
            json.dump(self.data, file, indent=4)

    def update(self, symbol, amount):
        """Update the wallet with a new symbol and amount."""
        self.data[symbol] = amount
        self.save()

    def clear(self, symbol=None):
        """
        Clear the wallet. If a symbol is provided, remove only that symbol.

        Args:
            symbol (str, optional): The symbol to remove. If None, clears the entire wallet.
        """
        if symbol:
            if symbol in self.data:
                del self.data[symbol]
                self.save()
                print(f"Cleared {symbol} from wallet.")
            else:
                print(f"Symbol '{symbol}' not found in wallet.")
        else:
            self.data = {}
            self.save()
            print("Cleared entire wallet.")

    def get(self, symbol):
        """Get the amount for a symbol."""
        return self.data.get(symbol, 0)

    def symbols(self):
        """Return all symbols in the wallet."""
        return list(self.data.keys())

def fetch_price_data(symbol, interval=None):
    """
    Fetch price data from Binance.

    API documentation: https://developers.binance.com/docs/derivatives/coin-margined-futures/market-data/Kline-Candlestick-Data

    Args:
        symbol (str): The trading pair symbol (e.g., BTCUSDT).
        interval (str): If provided, fetches kline data for the given interval.

    Returns:
        float or tuple: Current price if no interval; (open price, percent change) otherwise.
    """
    if interval:
        url = 'https://api.binance.com/api/v3/klines'
        params = {'symbol': symbol, 'interval': interval, 'limit': 2}
        response = requests.get(url, params=params)
        response.raise_for_status()
        data = response.json()
        open_yesterday = float(data[0][1])
        open_today = float(data[1][1])
        percent_change = (open_today - open_yesterday) / open_yesterday
        return open_today, percent_change
    else:
        url = f"https://api.binance.com/api/v3/ticker/price?symbol={symbol}"
        response = requests.get(url)
        response.raise_for_status()
        data = response.json()
        return float(data['price'])

def fetch_all_time_high(symbol):
    """
    Fetch the all-time high price for a given symbol.

    Args:
        symbol (str): The trading pair symbol (e.g., BTCUSDT).

    Returns:
        float: The all-time high price.
    """
    url = 'https://api.binance.com/api/v3/klines'
    params = {'symbol': f'{symbol}USDT', 'interval': '1M'}  # 1M = 1-month candles
    response = requests.get(url, params=params)
    response.raise_for_status()
    data = response.json()
    return max(float(candle[2]) for candle in data)  # High price is at index 2

def print_all_time_highs(symbols, unit):
    """
    Print the all-time high prices and percentage change for each symbol.

    Args:
        symbols (list): List of cryptocurrency symbols.
        unit (str): The trading pair unit (e.g., "USDT").
    """
    format = {
        'symbol': 8,
        'ath': 15,
        'current': 15,
        'change': 20,
    }
    format['line'] = sum(value for value in format.values())

    print(f"\n{'--- All-Time Highs ---'.ljust(format['line'])}")
    print(f"{'Symbol'.ljust(format['symbol'])}" +
          f"{'ATH'.rjust(format['ath'])}" +
          f"{'Current Price'.rjust(format['current'])}" +
          f"{'Change from ATH (%)'.rjust(format['change'])}")
    print('-' * format['line'])

    for symbol in symbols:
        try:
            # Fetch ATH and current price
            ath = fetch_all_time_high(symbol)
            current_price = fetch_price_data(f"{symbol}{unit}")

            # Calculate percentage change from ATH
            ath_change = ((current_price - ath) / ath) * 100

            # Print values
            print(f"{symbol.ljust(format['symbol'])}" +
                  f"{f'${ath:.2f}'.rjust(format['ath'])}" +
                  f"{f'${current_price:.2f}'.rjust(format['current'])}" +
                  f"{f'{ath_change:.2f}%'.rjust(format['change'])}")
        except Exception as e:
            print(f"Error fetching data for {symbol}: {e}")

def print_prices(symbols, unit, wallet=None):
    """
    Print today's open price, current price, daily change (%), and wallet value (if specified).

    Args:
        symbols (list): List of cryptocurrency symbols.
        unit (str): The trading pair unit (e.g., "USDT").
        wallet (Wallet, optional): Wallet instance for calculating wallet value.
    """
    format = {
        'symbol': 8,
        'open': 12,
        'current': 15,
        'change': 12,
        'wallet': 15,
    }
    format['line'] = sum(value for value in format.values())

    print(f"\n--- Prices ---")
    print(f"{'Symbol'.ljust(format['symbol'])}" +
          f"{'Open Price'.rjust(format['open'])}" +
          f"{'Current Price'.rjust(format['current'])}" +
          f"{'Change (%)'.rjust(format['change'])}" +
          f"{'Wallet Value'.rjust(format['wallet'])}")
    print('-' * format['line'])

    total_value = 0

    wallet = Wallet('wallet.json')
    for symbol in symbols:
        try:
            daily_open, _ = fetch_price_data(f"{symbol}{unit}", interval='1d')
            current_price = fetch_price_data(f"{symbol}{unit}")
            percent_change = ((current_price - daily_open) / daily_open) * 100

            wallet_value = wallet.get(symbol)
            if wallet:
                amount = wallet.get(symbol)
                wallet_value = amount * current_price
                total_value += wallet_value

            print(f"{symbol.ljust(format['symbol'])}" +
                  f"{f'${daily_open:.2f}'.rjust(format['open'])}" +
                  f"{f'${current_price:.2f}'.rjust(format['current'])}" +
                  f"{f'{percent_change:.2f}%'.rjust(format['change'])}" +
                  f"{f'${wallet_value:.2f}'.rjust(format['wallet'])}")
        except Exception as e:
            print(f"Error fetching data for {symbol}: {e}")

    if wallet:
        print('-' * format['line'])
        print(f"{'Total Wallet Value:'.ljust(format['line'] - format['wallet'])}" +
              f"{f'${total_value:.2f}'.rjust(format['wallet'])}")

def main():
    """Main program logic."""
    parser = argparse.ArgumentParser(description="Crypto Price Checker and Wallet Manager")
    parser.add_argument('-wallet', choices=['update', 'show', 'clear'],
                        help="Manage wallet: 'update', 'show', or 'clear'")
    parser.add_argument('-ath', action='store_true',
                        help="Show all-time highs for each symbol.")
    parser.add_argument('symbol', nargs='?',
                        help="Crypto symbol to update or clear (e.g., BTC)")
    parser.add_argument('amount', nargs='?', type=float,
                        help="Amount of the crypto symbol to update")

    args = parser.parse_args()

    wallet = Wallet('wallet.json')
    symbols = ['BTC', 'ETH', 'LINK', 'DOT', 'ADA']
    unit = 'USDT'

    if args.ath:
        # Show all-time highs
        print_all_time_highs(symbols, unit)
    elif args.wallet == 'update':
        if args.symbol and args.amount is not None:
            wallet.update(args.symbol, args.amount)
            print(f"Updated wallet: {args.symbol} = {args.amount}")
        else:
            print("Error: Must provide SYMBOL and AMOUNT to update the wallet.")
    elif args.wallet == 'show':
        print_prices(wallet.symbols(), unit, wallet)
    elif args.wallet == 'clear':
        if args.symbol:
            wallet.clear(args.symbol)
        else:
            wallet.clear()
    else:
        print_prices(symbols, unit)

if __name__ == '__main__':
    main()
```

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
