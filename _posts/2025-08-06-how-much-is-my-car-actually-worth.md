---
layout: post
title: "Scraping and analysing autotrader advert data to see exactly how much I'm getting ripped off"
date: "2025-08-06 08:00:00 +0000"
categories: general
tags: [python, data science, cars]
excerpt: "Selling a used car at the right price is hard! Most of us have no idea how much used cars are worth, so it's easy to get fooled into selling way below market value. Here's how I used autotrader's UK public adverts to create a dataset with make, model, year, mileage and cost, see where my car stacks up, and get a more precise idea of what it should be worth."
---

## Why?

Selling a used car at the right price is hard! The majority of us have very little idea how much used cars are worth, so it's easy for us to be fooled into selling massively below market value. To get a reasonable ballpark figure, we can use Autotrader's (in the UK) public adverts to create a dataset with make, model, year, mileage and cost, see where our car stacks up in that distribution, and get a precise value for what our car should be worth on that market.  
Obviously there are loads of other factors at play that aren't covered by these top level stats, but at least this points you in the right direction.

## Getting the Data (Autotrader makes it annoyingly hard)

[Autotrader](https://www.autotrader.co.uk) lets you search for a particular make/model, and in my case, there are currently **2,563 Honda Jazzes** for sale, from 2002 models up to 2025 brand new ones.  
The search results page gives you a list of adverts with a picture, a subtitle (often the sub-model), maybe a description, the mileage, the year, and the price. There's more info behind each link, but Autotrader makes it very hard to systematically scrape all the details, so we're basically stuck with this top-level info.

The page is also dynamically generated as you scroll (lazy loading), and if you go for _all_ the years at once, it starts to take a long time to load each new block. I therefore stuck to chunks of about 500 adverts: pick years 2002-2009, scroll to the bottom, inspect the page, and copy-paste the HTML from `<body>` onwards into a `.html` file like `2002-2009.html`. Then just do the same for later years in chunks until you have the whole dataset.

## Parsing the Data

Once I had the `.html` files, I ran a **simple Python + BeautifulSoup script** to extract and clean the HTML and get these fields: title, subtitle, price, mileage, and year.

> **If you want the full code and the example data, you can check it out here:**  
> [github.com/seamusbarnes/analyse-autotrader-data](https://github.com/seamusbarnes/analyse-autotrader-data)

Here’s the key bit that parses the files:

```python
from bs4 import BeautifulSoup
import pandas as pd
import re

def clean_price(s):
    # Extract integer price in pounds, e.g. '£1,590' -> 1590
    if not s:
        return pd.NA

    s = s.replace('£', '').replace(',', '').strip()
    m = re.search(r'\d+', s)

    return int(m.group()) if m else pd.NA

# ...similar for mileage, year...

adverts = []
for fname in HTML_FILES:
    with open(fname, encoding="utf-8") as f:
        soup = BeautifulSoup(f, "html.parser")

    for card in soup.find_all("div", {"data-testid": "advertCard"}):
        title = card.find("a", {"data-testid": "search-listing-title"}).get_text(strip=True)

        # ... etc ...

        entry = {"title": title, "price": price, ...}
        adverts.append(entry)

df = pd.DataFrame(adverts)
```

## Analysis

With everything in a DataFrame, I could:

- Plot the spread of prices by year or mileage bin.
- Get mean and 25/75th percentile prices for the year or mileage bin I care about.
- Fit a (simple) regression to the data and predict price for any year/mileage—only if it's in the same distribution! Extrapolation is a no-go.

Example: For a 2010 Honda Jazz with 100,000 miles, the average is about £2,000—but my first garage offer was £500! Now I've seen the numbers, I know that's way below market value and can negotiate (a bit) more confidently.

## Quick Plots

```python
import matplotlib.pyplot as plt
import seaborn as sns

plt.figure(figsize=(14, 6))
sns.violinplot(data=df, x='year_num', y='price_num', inner='quartile', cut=0)
plt.title("Honda Jazz Price Spread by Year")
plt.xlabel("Year")
plt.ylabel("Price (£)")
plt.show()
```

## Regression: Predicting Price (Linear and Polynomial)

Car prices don’t fall in a perfect straight line with year and mileage—big shock. Linear regression gives a rough idea, but a polynomial regression (degree 2) actually fits the trend a bit better.

Here’s both:

```python
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures

# Linear Regression
model = LinearRegression()

X = reg_df[['year_num', 'mileage_num']]
y = reg_df['price_num']

model.fit(X, y)

linear_pred = model.predict([[2010, 105000]])
print(f"Linear prediction: £{linear_pred[0]:,.0f}")

# Polynomial Regression (degree=2)
poly = PolynomialFeatures(degree=2, include_bias=False)

X_poly = poly.fit_transform(X)

model.fit(X_poly, y)

poly_pred = model.predict(poly.transform([[2010, 105000]]))

print(f"Polynomial prediction: £{poly_pred[0]:,.0f}")
```

This tends to fit the data a bit better (at least, visually, from the plot). Still, don’t trust extrapolation—it’ll get weird outside the observed data range.

## TL;DR

- Scrape HTML in chunks (manual copy-paste is fine for a one-off project)
- Parse out title, price, year, mileage
- Plot distributions and check the 25th/75th percentiles to get a sense of fair market price
- Ignore “dealer” offers unless you really want rid, they are taking a huge margin

Full Code and Data: [github.com/seamusbarnes/analyse-autotrader-data](https://github.com/seamusbarnes/analyse-autotrader-data)
