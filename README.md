**The Strategy:**  
Initially, I aimed to implement a mean reversion strategy, trying to predict price reversal points based on support and resistance levels. However, I found this approach challenging, as it required a lot of predictive power. Eventually, I pivoted to a trend-following system using a machine learning algorithm, which turned out to be much more manageable for me.

**Historical Data and Indicators:**  
Reliable and consistent data is crucial. To ensure I always have ample historical data across different timeframes with technical indicators already calculated, I built my own system. I developed a C# application with a SQL Server database that connects to Tradovate’s WebSocket for real-time data. Initially, I purchased a bulk of historical data for about $100, but since then, the system has been running smoothly for over two years. It aggregates minute candles into higher timeframes and calculates around 30 different basic technical indicators for each timeframe. I also wrote an API and a Python pip package that allows me to retrieve any subset of this data—a component that has proven to be a valuable investment.

**Machine Learning Approach:**  
I experimented with various methods before realizing that what I was trying to solve was a classification problem, not a regression one. The key question became: How do we classify this candle? This boiled down to a simple three-class classification problem—should we buy, hold, or sell? While I tested deep learning methods like 2D convolutional layers and TabNet, I eventually settled on simpler models like gradient boosting, which performed better with high-precision tabular data. Currently, my system uses CatBoost.

**Data Labeling:**  
Data labeling is crucial—garbage in, garbage out. I opted for the triple barrier method (thanks, Marcos Prado). A lot of optimization and grid searching went into finding the right combination of barriers, but it’s essential to define your barriers dynamically based on market conditions (ATR is a good starting point).

**Feature Engineering and Candle Signature:**  
The idea here is to create a set of stationary features for any given candle in your training dataset, label it, and then train your model on a large dataset (in my case, around 1 million rows). In real-time, you recalculate these features, feed them into the model, and let it classify the candle, generating a trading signal. It’s vital to ensure all features are stationary, or historical data becomes meaningless. My system currently uses about 45 features for each candle, ranging from simple indicators like ROC and RSI to more complex ones like the percentage distance between EMAs of different timeframes. It’s also important to use different lookback periods for each indicator to capture the temporal aspects of each candle.

**Back-Testing and Walk Forward:**  
I implemented a moving window system for backtesting, which closely mirrors how the system operates in real-time. The model is trained on three years of historical data, trades for one day, then the window moves forward. This approach, while time-consuming (backtesting one year requires 252 model trainings), ensures that my backtest (walk forward) closely replicates the live system. I used Backtrader to simulate trades, storing the results in a CSV file. The SL/TP values are ATR-based multipliers optimized through a grid search, reducing the chance of overfitting by dynamically calculating these values across the entire time range.

**Live System:**  
For live trading, I use the Tradovate API to send trades to CME. I developed a trade management system in C# that connects to Tradovate, maintains its own SQL database for tracking trades, and receives signals from the model. While there are additional considerations in the live system, such as risk management, resiliency, and monitoring, my background as a senior technology lead in large enterprises has prepared me well for these challenges.

**Infrastructure and DevOps:**  
Everything runs on a (rather large) VPS server from Vultr, hosted in Chicago to be close to the CME and Tradovate servers (though this isn’t a high-frequency system). All components are deployed as Docker images, with code stored in Git and automated pipelines that deploy to production when I push to the main branch.

**System Architecture:**  
The attached image should give you some understanding of how the components work together. there are multiple components, each one deployed as a docker container and they communicate via a messaging system. I used RabbitMQ for messaging in the system.

![image](https://github.com/user-attachments/assets/56b95f43-98d9-45bf-b856-8c5f065262bf)


**Results:**  
This is where it gets a bit more interesting. all of these results are from my backtests from 2022, but the whole system has been running live for a while now, and as a validation, I run the backtest almost every day after the live session ends and compare the results. they are almost identical with the exception of tiny slippage differences. (which is actually a bit more stringent on my back-tests). on average., the system makes about 150 trades in a month. that translates to 4 or 5 trades per day. I have also built a Telegram bot to notify me of new trades and end of day trade summary.  
it is important to note that this is trading only 1 NQ future contract. so no budgeting/portfolio management here. just 1 NQ and only 1 open trade at a time. it is also a day-trading only system. it closes the position at 15:59 EST and does not hold over-night.

|Test Date Range|2022-01-02 to 2024-08-05|
|:-|:-|
|Starting Balance|5000|
|Total Profit|$992,012|
|Profit Factor|1.78|
|Maximum Drawdown|$14435.50 (10.89%)|
|Sharpe Ratio|3.15|
|Total Trades|4335|
|Avg Trade Duration|2.49 hours|

![image](https://github.com/user-attachments/assets/99d7cf79-5d08-4eb0-b24a-36d17e16ec6c)

![image](https://github.com/user-attachments/assets/0a48ee7b-1306-4891-9312-e2b940e64fde)

![image](https://github.com/user-attachments/assets/aec51336-1cf3-4454-92e4-65e9271277b1)

![image](https://github.com/user-attachments/assets/b1b910a1-17f6-4ffb-ae70-612bda0d2067)

![image](https://github.com/user-attachments/assets/4e1eea04-0a33-4945-a1fd-a383fac2becc)

![image](https://github.com/user-attachments/assets/8f0b4318-3df5-4896-be41-1b0fc3c70a3e)


I will share the links to the .csv file of trades and the analysis pdf in the first comment. Please have a look.

**Next Steps:**  
I am planning to use the same signals to trade future options on CME instead of trading future contracts. It will be a lot harder to simulate option pricing in back-test (as historical options data is very difficult to come by)  
but I am sure it can improve the performance by reducing the draw-down. initial calculation shows that I can reduce my drawdown by 60% trading on options, while profit will go down by 30%, still a good win.

sorry for the long post, hope it helps. let me know what you think.  
if you guys see any issues or something that I am overlooking, please let me know.

Cheers and happy (Algo)Trading!
