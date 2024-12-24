[![CI](https://github.com/KapJI/capital-gains-calculator/actions/workflows/ci.yml/badge.svg)](https://github.com/KapJI/capital-gains-calculator/actions)
[![PyPI version](https://img.shields.io/pypi/v/cgt-calc)](https://pypi.org/project/cgt-calc/)

# UK capital gains calculator

Calculate capital gains tax from transaction history exports and generate a PDF report with calculations.

Currently supported brokers:
- Charles Schwab
- Trading 212
- Morgan Stanley

Automatically convert all prices to GBP and apply HMRC rules to calculate capital gains tax: "same day" rule, "bed and breakfast" rule, section 104 holding.

## Report example

[calculations_example.pdf](https://github.com/KapJI/capital-gains-calculator/blob/main/calculations_example.pdf)

## Installation

Install it with [pipx](https://pypa.github.io/pipx/) (or regular pip):

```shell
pipx install cgt-calc
```

## Prerequisites

-   Python 3.9 or above.
-   `pdflatex` is required to generate the report.
-   [Optional] Docker

## Install LaTeX

### MacOS

```shell
brew install --cask mactex-no-gui
```

### Debian based

```shell
apt install texlive-latex-base
```

### Windows

[Install MiKTeX.](https://miktex.org/download)

### Docker

These steps will build and run the calculator in a self-contained environment, in case you would rather not have a systemwide LaTeX installation (or don't want to interfere with an existing one).
The following steps are tested on an Apple silicon Mac and may need to be slightly modified on other platforms.
With the cloned repository as the current working directory:

```shell
$ docker buildx build --platform linux/amd64 --tag capital-gains-calculator .
```

Now you've built and tagged the calculator image, you can drop into a shell with `cgt-calc` installed on `$PATH`. Navigate to where you store your transaction data, and run:

```shell
$ cd ~/Taxes/Transactions
$ docker run --rm -it -v "$PWD":/data capital-gains-calculator:latest
a4800eca1914:/data# cgt-calc [...]
```

This will create a temporary Docker container with the current directory on the host (where your transaction data is) mounted inside the container at `/data`. Follow the usage instructions below as normal,
and when you're done, simply exit the shell. You will be dropped back into the shell on your host, with your output report pdf etc..

## Usage

### Schwab

1. **Exported transaction history from Schwab**:
   - Export the transaction history in CSV format from the beginning or at least from when you first acquired the shares you held during the tax year.
   - Note: Schwab allows downloads for only the last 4 years, so keep it safe. After 4 years, you may need to restore transactions from PDF statements.
   - [Example CSV](https://github.com/KapJI/capital-gains-calculator/blob/main/tests/test_data/schwab_transactions.csv).

2. **Exported transaction history from Schwab Equity Awards**:
   - Export the transaction history from Schwab Equity Awards (e.g., for Alphabet/Google employees) in JSON format.
   - Instructions for this are available at the top of the [parser file](https://github.com/KapJI/capital-gains-calculator/blob/main/cgt_calc/parsers/schwab_equity_award_json.py).
   - Schwab now allows for the whole history of Equity Awards account transactions to be downloaded.

### Trading 212

- **Exported transaction history from Trading 212**:
   - You can use several files as Trading 212 limits statements to 1-year periods.
   - [Example Files](https://github.com/KapJI/capital-gains-calculator/tree/main/tests/test_data/trading212).

### Morgan Stanley

- **Exported transaction history from Morgan Stanley**:
   - Specify the directory produced from the report download page, as Morgan Stanley generates multiple files in a single report.

### Sharesight

- **Exported transaction history from Sharesight**:
   - Export the "All Trades" report and also optionally, the 'Taxable Income' report where applicable (if you have no dividends, distributions or interest payments received, the 'Taxable Income' report cannot be generated)
   - Select "Since Inception" for the date range period and "Do not group" for the 'Group by' option.
   - Download the report(s) as Excel (`.XLSX`) files or export to Google Sheets, convert to CSV, and place them in the same folder.
   - Sharesight aggregates transactions from multiple brokers but doesn't necessarily have balance information, so use the `--no-balance-check` flag to avoid errors.
   - For equity grants, add `Stock Activity` in the comment associated with any vesting transactions, ensuring the grant price is filled.
   - [Example Files](https://github.com/KapJI/capital-gains-calculator/tree/main/tests/test_data/sharesight).

### Initial Stock Prices

- **CSV file with initial stock prices in USD**:
   - The `initial_prices.csv` comes pre-packaged. Use the same format.
   - [Initial Prices CSV](https://github.com/KapJI/capital-gains-calculator/blob/main/cgt_calc/resources/initial_prices.csv).

### Exchange Rates

- **Monthly exchange rates prices from [gov.uk](https://www.gov.uk/government/collections/exchange-rates-for-customs-and-vat)**:
   - `exchange_rates.csv` gets generated automatically using HMRC API, but you can override it using the same format.

### Running the Calculation

```shell
cgt-calc --year 2020 --schwab schwab_transactions.csv --trading212 trading212/ --mssb mmsb_report/


## Disclaimer

The output of this tool has been prepared for informational purposes only, and is not intended to provide, and should not be relied on for, tax, legal or accounting advice. The calculations produced are for illustrative purposes only and do not constitute tax advice or recommendations in any jurisdiction.

The maintainers of this tool do not accept responsibility for any errors.

## Contribute

All contributions are highly welcomed.
If you notice any bugs please open an issue or send a PR to fix it.

Feel free to add new parsers to support transaction history from more brokers.

## Testing

This project uses [Poetry](https://python-poetry.org/) for managing dependencies.

-   For local testing you need to [install it](https://python-poetry.org/docs/#installation).
-   After that run `poetry install` to install all dependencies.
-   Then activate `pre-commit` hook: `poetry run pre-commit install`

You can also run all linters and tests manually with this command:

```shell
poetry run pre-commit run --all-files
```
