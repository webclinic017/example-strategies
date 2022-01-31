# Example Strategies

[![Tests and linting](https://github.com/joksas/example-strategies/actions/workflows/tests-and-linting.yml/badge.svg)](https://github.com/joksas/example-strategies/actions/workflows/tests-and-linting.yml) [![CodeQL](https://github.com/joksas/example-strategies/actions/workflows/code-ql.yml/badge.svg)](https://github.com/joksas/example-strategies/actions/workflows/code-ql.yml)

This repository contains my attempts at implementing some simple statistical tests, trading strategies and optimisation techniques as I am learning about the fundamentals of algorithmic and quantitative trading.
It is very much work in progress.

## Estimating Stationarity

What strategy works best may depend on how stationary a particular stock is.
According to [this](https://www.quantstart.com/successful-algorithmic-trading-ebook/), one may use [Hurst exponent](https://en.wikipedia.org/wiki/Hurst_exponent) *H* to get an estimate of stock's stationarity:

* *H* < 0.5 -- time series is mean reverting
* *H* = 0.5 -- time series is a geometric Brownian motion (i.e. random walk)
* *H* > 0.5 -- time series is trending

We may estimate the Hurst exponent of Coca-Cola's stock price using the following snippet of code (see [this](/example_strategies/stats.py#L7) for implementation details):
```python
import datetime

from example_strategies import data, stats

# 2010s date range
from_date, to_date = datetime.date(2010, 1, 1), datetime.date(2019, 12, 31)

# Coca-Cola stock price
ko = data.load("KO", from_date=from_date, to_date=to_date)
ko_close = ko["Close"].to_numpy()

h = stats.hurst_exponent(ko_close)
print(f"Hurst exponent is {h:.2f}")
```

When executed, the code should yield the following output:
```text
Hurst exponent is 0.41
```

Thus, Coca-Cola's Hurst exponent indicates the stock price is biased towards mean-reverting behaviour.

## Example Optimisation

Suppose we wanted to optimise moving average crossover strategy using Sharpe ratio.
We could pick a number of companies and, to check if the procedure does not lead to overfitting, divide them into training and test sets.
Optimised parameters would be obtained using the training set and evaluated using the test set (see [this](/example_strategies/optimisation.py#L11) for implementation details).
This is summarised below:
```python
import datetime
import random

from example_strategies import optimisation, strategies

# Top 10 S&P 500 companies by market cap that went public before 2010.
tickers = ["AAPL", "MSFT", "AMZN", "GOOGL", "GOOG", "FB", "NVDA", "BRK-B", "JPM", "JNJ"]
random.seed(0)
random.shuffle(tickers)
# 70% in the training set.
num_training = int(0.7 * len(tickers))

optimal_params, train_avg_sharpe, test_avg_sharpe = optimisation.grid_search(
    strategies.MACrossoverStrategy,
    tickers[:num_training],
    tickers[num_training:],
    {
        "fast_length": [2, 5, 10],
        "slow_length": [20, 50, 100],
    },
    from_=datetime.date(2010, 1, 1),
    to=datetime.date(2019, 12, 31),
    metric="sharpe",
)
print(f'Optimal fast moving average length is {optimal_params["fast_length"]} days.')
print(f'Optimal slow moving average length is {optimal_params["slow_length"]} days.')
print("Sharpe ratio is")
print(f"* {train_avg_sharpe:.2f} in the training set")
print(f"* {test_avg_sharpe:.2f} in the test set")
```

When executed, the code should yield the following output:
```text
Optimal fast moving average length is 10 days.
Optimal slow moving average length is 50 days.
Sharpe ratio is
* 0.58 in the training set
* 0.52 in the test set
```

## Unit Testing

Execute
```text
pytest tests
```
