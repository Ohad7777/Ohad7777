- 👋 Hi, I’m @Ohad7777
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
Ohad7777/Ohad7777 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
import time
import pandas as pd
from binance.client import Client
from binance.enums import *

api_key = 'your_api_key_here'
api_secret = 'your_api_secret_here'

client = Client(api_key, api_secret)

def get_historical_data(symbol, interval, lookback):
    klines = client.get_historical_klines(symbol, interval, lookback)
    df = pd.DataFrame(klines, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume', 'close_time', 'quote_av', 'trades', 'tb_base_av', 'tb_quote_av', 'ignore'])
    df['close'] = pd.to_numeric(df['close'])
    return df

def calculate_sma(df, window):
    sma = df['close'].rolling(window=window).mean()
    return sma

def trade(symbol, sma_short, sma_long):
    balance = client.get_asset_balance(asset='USDT')
    usdt_balance = float(balance['free'])

    df = get_historical_data(symbol, '1h', '100 hours ago UTC')

    sma_short_term = calculate_sma(df, sma_short).iloc[-1]
    sma_long_term = calculate_sma(df, sma_long).iloc[-1]

    if sma_short_term > sma_long_term:
        if usdt_balance > 10:
            buy_order = client.order_market_buy(symbol=symbol, quantity=calculate_quantity(symbol, usdt_balance))
            print(f"BUY order placed: {buy_order}")
    elif sma_short_term < sma_long_term:
        crypto_balance = float(client.get_asset_balance(asset=symbol.replace('USDT', ''))['free'])
        if crypto_balance > 0.001:
            sell_order = client.order_market_sell(symbol=symbol, quantity=crypto_balance)
            print(f"SELL order placed: {sell_order}")

def calculate_quantity(symbol, usdt_balance):
    price = float(client.get_symbol_ticker(symbol=symbol)['price'])
    quantity = round(usdt_balance / price, 5)
    return quantity

def trading_bot(symbol, sma_short, sma_long, interval):
    while True:
        trade(symbol, sma_short, sma_long)
        time.sleep(interval)

if __name__ == "__main__":
    symbol = 'BTCUSDT'
    sma_short = 5
    sma_long = 20
    interval = 3600

    print("Starting trading bot...")
    trading_bot(symbol, sma_short, sma_long, interval)
