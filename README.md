Features
Data Caching: Save and reuse financial data locally to reduce redundant API calls.
MACD: Tracks momentum changes to generate buy/sell signals based on the crossover of short and long-term moving averages.
OBV: Uses trading volume to confirm price trends and signal potential reversals.
RSI: Identifies overbought and oversold conditions, offering buy/sell opportunities.
Williams %R: Tracks recent price highs/lows to detect entry/exit points.
Star Rating: Rates signal strength on a 5-star scale, displayed visually for easy understanding.
Graphical Visualization: Displays currency pair analysis and star ratings in a user-friendly format.

1. Installation
To get started, follow these steps:

Clone this repository:git clone https://github.com/YourUsername/Forex-Trading-Signal-Analyzer.git
cd Forex-Trading-Signal-Analyzer

2. Install the required dependencies:pip install -r requirements.txt

Usage
To run the program and analyze Forex currency pairs, execute the following command:python fx_analyzer.py

The program will analyze predefined currency pairs and generate buy/sell signals across multiple timeframes. Results will be displayed in the terminal with the option to visualize star ratings.

Supported Timeframes
The tool supports multiple timeframes for in-depth analysis:

1 Minute
5 Minutes
15 Minutes
30 Minutes
1 Hour
Each timeframe is analyzed independently to provide insights on short-term and medium-term trading opportunities.

Star Rating System
The tool calculates a star rating for each analyzed currency pair based on the strength of the signals from different indicators (MACD, OBV, RSI, and Williams %R). Ratings are displayed from 1 to 5 stars, where 5 stars represent strong signals for trading.

Buy signals are highlighted in green.
Sell signals are highlighted in red.
Keep (no action) signals are highlighted in purple.
Customization
You can customize several aspects of the program:

Modify indicator parameters like the RSI window size or MACD short/long window values to suit your trading strategy.
Add or remove currency pairs by editing the tickers list in the code.
Adjust the weighting of different indicators for signal strength calculation.
Examples
Here’s an example of how the signals are displayed for a currency pair:
==================================================
Results for EURUSD=X (★★★☆☆) on 1 Hour timeframe
==================================================

MACD Signals:
EURUSD=X: MACD BUY signal on 1 Hour timeframe

--------------------------------------------------

OBV Signals:
EURUSD=X: OBV KEEP signal on 1 Hour timeframe

--------------------------------------------------

RSI Signals:
EURUSD=X: RSI outside the trading range on 1 Hour timeframe

--------------------------------------------------

Williams %R Signals:
EURUSD=X: Williams %R BUY signal on 1 Hour timeframe

Additionally, star ratings are plotted visually using Matplotlib, with details on each currency pair and timeframe.

Contributing
Contributions are welcome! Feel free to submit a pull request or open an issue to discuss potential changes or improvements.

