import yfinance as yf
import pandas as pd
import time
import random
import os
import matplotlib.pyplot as plt

CACHE_DIR = 'cache'

def get_cache_file_path(ticker, interval):
    return os.path.join(CACHE_DIR, f"{ticker.replace('=', '_')}_{interval}.json")

def save_to_cache(ticker, interval, data):
    if not os.path.exists(CACHE_DIR):
        os.makedirs(CACHE_DIR)
    file_path = get_cache_file_path(ticker, interval)
    with open(file_path, 'w') as f:
        data.to_json(f, orient='split')

def load_from_cache(ticker, interval):
    file_path = get_cache_file_path(ticker, interval)
    if os.path.exists(file_path):
        with open(file_path, 'r') as f:
            df = pd.read_json(f, orient='split')
        return df
    return pd.DataFrame()

def get_forex_data(ticker, period='1d', interval='1m', retries=3, delay=5):
    cached_data = load_from_cache(ticker, interval)
    if not cached_data.empty:
        return cached_data
    attempt = 0
    while attempt < retries:
        try:
            df = yf.download(tickers=ticker, period=period, interval=interval)
            if df.empty:
                print(f"Warning: No data retrieved for {ticker} with interval {interval}")
            save_to_cache(ticker, interval, df)
            return df
        except Exception as e:
            attempt += 1
            print(f"Error fetching data for {ticker} with interval {interval}: {e}")
            if attempt < retries:
                wait_time = delay + random.uniform(0, 2)
                print(f"Retrying in {wait_time:.2f} seconds...")
                time.sleep(wait_time)
            else:
                return pd.DataFrame()

def calculate_macd(df, short_window=12, long_window=26, signal_window=9):
    if df.empty:
        return df
    df['EMA12'] = df['Close'].ewm(span=short_window, adjust=False, min_periods=50).mean()
    df['EMA26'] = df['Close'].ewm(span=long_window, adjust=False, min_periods=50).mean()
    df['MACD'] = df['EMA12'] - df['EMA26']
    df['Signal_Line'] = df['MACD'].ewm(span=signal_window, adjust=False, min_periods=50).mean()
    df['Buy_Signal'] = (df['MACD'] > df['Signal_Line']) & (df['MACD'] > 0)
    df['Sell_Signal'] = (df['MACD'] < df['Signal_Line']) & (df['MACD'] < 0)
    return df

def calculate_obv(df):
    if df.empty:
        return df
    df['OBV'] = (df['Volume'] * (~df['Close'].diff().le(0) * 2 - 1)).cumsum()
    df['OBV_EMA'] = df['OBV'].ewm(span=20, adjust=False).mean()  # EMA of OBV for smoothing
    df['OBV_Buy_Signal'] = (df['OBV'] > df['OBV_EMA'])
    df['OBV_Sell_Signal'] = (df['OBV'] < df['OBV_EMA'])
    return df

def calculate_rsi(df, window=14):
    if df.empty:
        return df
    delta = df['Close'].diff()
    gain = (delta.where(delta > 0, 0)).rolling(window=window).mean()
    loss = (-delta.where(delta < 0, 0)).rolling(window=window).mean()
    rs = gain / loss
    df['RSI'] = 100 - (100 / (1 + rs))
    df['RSI_Buy_Signal'] = df['RSI'] < 30
    df['RSI_Sell_Signal'] = df['RSI'] > 70
    return df

def calculate_williams_r(df, window=14):
    if df.empty:
        return df
    high_max = df['High'].rolling(window=window).max()
    low_min = df['Low'].rolling(window=window).min()
    df['Williams_%R'] = -100 * ((high_max - df['Close']) / (high_max - low_min))
    df['Williams_Buy_Signal'] = df['Williams_%R'] < -80  # Buy when %R crosses above -80
    df['Williams_Sell_Signal'] = df['Williams_%R'] > -20  # Sell when %R crosses below -20
    return df

def calculate_rating(forex_data, weight_macd=1, weight_obv=1, weight_rsi=1, weight_williams_r=1):
    rating = 0
    # MACD Rating
    if forex_data['Buy_Signal'].iloc[-1]:
        rating += weight_macd
    elif forex_data['Sell_Signal'].iloc[-1]:
        rating -= weight_macd

    # OBV Rating
    if forex_data['OBV_Buy_Signal'].iloc[-1]:
        rating += weight_obv
    elif forex_data['OBV_Sell_Signal'].iloc[-1]:
        rating -= weight_obv

    # RSI Rating
    if forex_data['RSI_Buy_Signal'].iloc[-1]:
        rating += weight_rsi
    elif forex_data['RSI_Sell_Signal'].iloc[-1]:
        rating -= weight_rsi


    # Williams %R Rating
    if 'Williams_Buy_Signal' in forex_data.columns and forex_data['Williams_Buy_Signal'].iloc[-1]:
        rating += weight_williams_r
    elif 'Williams_Sell_Signal' in forex_data.columns and forex_data['Williams_Sell_Signal'].iloc[-1]:
        rating -= weight_williams_r

    # Ensure rating is between 0 and 5
    rating = min(max(rating, 0), 5)
    return rating

def get_star_rating(rating):
    return '★' * rating + '☆' * (5 - rating)

def plot_star_rating(rating, ticker, timeframe):
    if rating < 1:
        return  # Do not plot if rating is less than 1
    plt.figure(figsize=(10, 2))
    plt.text(0.5, 0.6, f'{ticker} - {timeframe}', fontsize=20, ha='center', va='center', color='green', fontweight='bold')
    plt.text(0.5, 0.4, f'{get_star_rating(rating)}', fontsize=30, ha='center', va='center')
    plt.text(0.5, 0.2, 'Author: EMAD KHOSRAVI', fontsize=12, ha='center', va='center', color='green', fontweight='bold', bbox=dict(facecolor='white', edgecolor='green', boxstyle='round,pad=0.5'))
    plt.axis('off')
    plt.show()

def generate_signals(tickers):
    timeframes = {
        '1m': '1 Minute',
        '5m': '5 Minute',
        '15m': '15 Minute',
        '30m': '30 Minute',
        '1h': '1 Hour'
    }

    for ticker in tickers:
        for interval, name in timeframes.items():
            period = '1d' if interval == '1m' else '5d'
            forex_data = get_forex_data(ticker, period=period, interval=interval)
            forex_data = calculate_macd(forex_data)
            forex_data = calculate_obv(forex_data)
            forex_data = calculate_rsi(forex_data)

            if interval in ['15m', '30m', '1h']:
                forex_data = calculate_williams_r(forex_data)

            if forex_data.empty or len(forex_data) < 50:
                print(f"{ticker}: Not enough data for {name} timeframe")
                continue

            rating = calculate_rating(forex_data)
            star_rating = get_star_rating(rating)

            print(f"\n{'=' * 50}")
            print(f"Results for {ticker} ({star_rating}) on {name} timeframe")
            print(f"{'=' * 50}\n")

            print("MACD Signals:")
            if forex_data['Buy_Signal'].iloc[-1]:
                print(f"\033[92m{ticker}: MACD BUY signal on {name} timeframe\033[0m")
            elif forex_data['Sell_Signal'].iloc[-1]:
                print(f"\033[91m{ticker}: MACD SELL signal on {name} timeframe\033[0m")
            else:
                print(f"\033[95m{ticker}: MACD KEEP signal on {name} timeframe\033[0m")

            print(f"\n{'-' * 50}")

            print("OBV Signals:")
            if forex_data['OBV_Buy_Signal'].iloc[-1]:
                print(f"\033[92m{ticker}: OBV BUY signal on {name} timeframe\033[0m")
            elif forex_data['OBV_Sell_Signal'].iloc[-1]:
                print(f"\033[91m{ticker}: OBV SELL signal on {name} timeframe\033[0m")
            else:
                print(f"\033[95m{ticker}: OBV KEEP signal on {name} timeframe\033[0m")

            print(f"\n{'-' * 50}")

            print("RSI Signals:")
            if forex_data['RSI'].iloc[-1] > 60 or forex_data['RSI'].iloc[-1] < 40:
                if forex_data['RSI_Buy_Signal'].iloc[-1]:
                    print(f"\033[92m{ticker}: RSI BUY signal on {name} timeframe\033[0m")
                elif forex_data['RSI_Sell_Signal'].iloc[-1]:
                    print(f"\033[91m{ticker}: RSI SELL signal on {name} timeframe\033[0m")
                else:
                    print(f"\033[95m{ticker}: RSI KEEP signal on {name} timeframe\033[0m")
            else:
                print(f"\033[93m{ticker}: RSI outside the trading range on {name} timeframe\033[0m")


            if interval in ['15m', '30m', '1h']:
                print(f"\n{'-' * 50}")
                print("Williams %R Signals:")
                if forex_data['Williams_Buy_Signal'].iloc[-1]:
                    print(f"\033[92m{ticker}: Williams %R BUY signal on {name} timeframe\033[0m")
                elif forex_data['Williams_Sell_Signal'].iloc[-1]:
                    print(f"\033[91m{ticker}: Williams %R SELL signal on {name} timeframe\033[0m")
                else:
                    print(f"\033[95m{ticker}: Williams %R KEEP signal on {name} timeframe\033[0m")

            plot_star_rating(rating, ticker, name)

# Example tickers
tickers = ['EURUSD=X', 'GBPUSD=X', 'USDJPY=X', 'USDCHF=X', 'AUDUSD=X', 'NZDUSD=X', 'USDCAD=X', 'USDTRY=X', 'USDZAR=X', 'USDMXN=X']
generate_signals(tickers)
