<h1>Unit 5 - Financial Planning</h1>

Aidan Laird



* Initial imports

import os
import requests
import pandas as pd
from dotenv import load_dotenv
import alpaca_trade_api as tradeapi
from MCForecastTools import MCSimulation

%matplotlib inline

* Load .env enviroment variables
load_dotenv()


## Part 1

* Set current amount of crypto assets
my_btc = 1.2
my_eth = 5.3

* Crypto API URLs
btc_url = "https://api.alternative.me/v2/ticker/Bitcoin/?convert=GBP"
eth_url = "https://api.alternative.me/v2/ticker/Ethereum/?convert=GBP"

* Fetch current BTC price
response_data_btc = requests.get(btc_url)
response_data_btc
response_content_btc = response_data_btc.content
response_content_btc
data_btc = response_data_btc.json()
import json
print(json.dumps(data_btc, indent=7))

* Fetch current ETH price
response_data_eth = requests.get(eth_url)
response_content_eth = response_data_eth.content
data_eth = response_data_eth.json()
import json
print(json.dumps(data_eth, indent=7))

* Compute current value of my crpto

btc_value = data_btc["data"]["1"]["quotes"]["USD"]["price"]
my_btc_value = my_btc * btc_value

eth_value = data_eth["data"]["1027"]["quotes"]["USD"]["price"]
my_eth_value = my_eth * eth_value

btc_GBP_value = data_btc["data"]["1"]["quotes"]["GBP"]["price"]
my_btc_value_GBP = my_btc * btc_GBP_value

eth_GBP_value = data_eth["data"]["1027"]["quotes"]["GBP"]["price"]
my_eth_value_GBP = my_eth * eth_GBP_value

* Print current crypto wallet balance
print('\n')
print("My crypto balances in USD and GBP")
print(f'----------------------------'+'\n')
print(f"The current value of Aidan's {my_btc} BTC is US${my_btc_value:0.2f}")
print(f"The current value of Aidan's {my_eth} ETH is US${my_eth_value:0.2f}")
print('\n')
print(f"The current value of Aidan's {my_btc} BTC in Sterling is £{my_btc_value_GBP:0.2f}")
print(f"The current value of Aidan's {my_eth} ETH is Sterling is £{my_eth_value_GBP:0.2f}")


### Collect Investments Data Using Alpaca: `SPY` (stocks) and `AGG` (bonds)

* Set current amount of shares
my_agg = 200
my_spy = 50

* Set Alpaca API key and secret
alpaca_api_key = os.getenv("ALPACA_API_KEY")
alpaca_secret_key = os.getenv("ALPACA_SECRET_KEY")


* Create the Alpaca API object
api = tradeapi.REST(
    alpaca_api_key,
    alpaca_secret_key,
    api_version = "v2"
)

* Format current date as ISO format

today = pd.Timestamp("2021-12-16", tz="America/New_York").isoformat()

* Set the tickers
tickers = ["AGG", "SPY"]

* Set timeframe to '1D' for Alpaca API
timeframe = "1D"

* Get current closing prices for SPY and AGG
* (use a limit=1000 parameter to call the most recent 1000 days of data)
df_prices = api.get_barset(
    tickers,
    timeframe,
    limit=3
).df

* Preview DataFrame
df_prices.head()

* Pick AGG and SPY close prices
agg_close_price = df_prices['AGG']['close'][0]
spy_close_price = df_prices['SPY']['close'][0]

* Print AGG and SPY close prices
print(f"Current AGG closing price: ${agg_close_price}")
print(f"Current SPY closing price: ${spy_close_price}")

* Compute the current value of shares
my_spy_value = spy_close_price * my_spy
my_agg_value = agg_close_price * my_agg

* Print current value of shares
print(f"The current value of your {my_spy} SPY shares is ${my_spy_value:0.2f}")
print(f"The current value of your {my_agg} AGG shares is ${my_agg_value:0.2f}")

* Set monthly household income
monthly_income = 12000

* Consolidate financial assets data
total_crypto = my_btc_value + my_eth_value
total_shares = my_spy_value + my_agg_value
data = [ total_crypto, total_shares]

* Create savings DataFrame
df_savings = pd.DataFrame(data, index=['Crypto','Shares'], columns=['Amount in USD'])

* Display savings DataFrame
display(df_savings)

* Plot savings pie chart
df_savings.plot.pie(y='Amount in USD',title='Personal Savings')

* Set ideal emergency fund
emergency_fund = monthly_income * 3

* Calculate total amount of savings
total_savings = total_crypto + total_shares

* Validate saving health
if (total_savings > emergency_fund):
    print("Congratulations! You have enough money in this fund")
elif (total_savings == emergency_fund):
    print("Congratualtions on reaching this financial goal!")
else:
    diff = emergency_fund - total_savings
    print(f"You are ${diff} away from reaching your financial goal, keep saving!")
    
    
## Part 2 - Retirement Planning

### Monte Carlo Simulation


* Set start and end dates of five years back from today.
* Sample results may vary from the solution based on the time frame chosen
start_date = pd.Timestamp('2016-05-01', tz='America/New_York').isoformat()
end_date = pd.Timestamp('2021-05-01', tz='America/New_York').isoformat()

* Get 5 years' worth of historical data for SPY and AGG
* (use a limit=1000 parameter to call the most recent 1000 days of data)

tickers = ["AGG", "SPY"]
df_stock_data = api.get_barset(
    tickers,
    timeframe,
    start=start_date,
    end=end_date,
    limit= 1000
).df

df_stock_data = pd.concat([df_stock_data])

* Display sample data
df_stock_data.head()

* Configuring a Monte Carlo simulation to forecast 30 years cumulative returns
MC_stocks_dist = MCSimulation(
    portfolio_data = df_stock_data,
    weights = [0.4, 0.6],
    num_simulation = 500,
    num_trading_days = 252*30
)




* Printing the simulation input data
MC_stocks_dist.portfolio_data.head()

* Running a Monte Carlo simulation to forecast 30 years cumulative returns
MC_stocks_dist.calc_cumulative_return()

* Plot simulation outcomes
MC_stocks_dist.plot_simulation()

* Plot probability distribution and confidence intervals
MC_stocks_dist.plot_distribution()

* Fetch summary statistics from the Monte Carlo simulation results
summary = MC_stocks_dist.summarize_cumulative_return()

* Print summary statistics
summary

* Set initial investment
initial_investment = 20000

* Use the lower and upper `95%` confidence intervals to calculate the range of the possible outcomes of our $20,000
ci_lower = round(summary[8]*initial_investment, 2)
ci_upper = round(summary[9]*initial_investment, 2)

* Print results
print(f"There is a 95% chance that an initial investment of ${initial_investment} in the portfolio"
      f" over the next 30 years will end within in the range of"
      f" ${ci_lower} and ${ci_upper}")
      
      
      
* Set initial investment
initial_investment = 20000 * 1.5

* Use the lower and upper `95%` confidence intervals to calculate the range of the possible outcomes of our $30,000
ci_lower = round(summary[8]*initial_investment, 2)
ci_upper = round(summary[9]*initial_investment, 2)

* Print results
print(f"There is a 95% chance that an initial investment of ${initial_investment} in the portfolio"
      f" over the next 30 years will end within in the range of"
      f" ${ci_lower} and ${ci_upper}")


















---

© 2021 Trilogy Education Services, a 2U, Inc. brand. All Rights Reserved.
