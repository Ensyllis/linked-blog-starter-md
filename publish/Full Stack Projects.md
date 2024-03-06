## Anchor Capital Management LLC
- This is my Baby, my Pride and my Joy. It's what lead me down the path of Programming, Software engineering, Statistics and Data Science.
- The problem it solves is fundamentally this. "Which stocks do I buy?". The answer has always been vague, but now If I have a risk tolerance of $\_\_\_\_$% annually, and I want to diversify with 10 stocks from the S&P 500, which stocks should I buy and how many stocks should I buy? 
	- The key to this was to let people diversify while keeping their portfolio's risk **anchored** to their personal risk tolerance so they can avoid bad practices such as going all in on one stock. 

The main function in the back-end takes in the 
- Stock list you want it to choose from
- Target standard deviation
- Number of Assets int he final Portfolio
- Amount of money you plan to invest in
```python
portfolio = []
list_of_prices, optimal_weights, final_shares, standard_deviation, returns, cash_at_hand = main_function(stock_list, target_sd, number_of_assets, Total_Allocation)
```
and it returns 
- Prices for each stock
- Weights for the Portfolio
- Number of Shares to buy
- Standard Deviation given the Shares
- Historical Returns given the weights
- Total amount of cash you will spend on the portfolio
- Portfolio of the selected Tickers

