# Project-stocks
"""
Created on Mon May  6 11:03:02 2024

@author: user
I created TwoWeekReturns is created to calculate returns from a data file. Then, 
an instance of TradingStrategy is initialized with the calculated returns 
and an initial capital of $10,000. The strategy is run, and the results
 are printed, including the portfolio composition and return. Finally, 
 a line graph illustrating cumulative returns for each stock over three
 weeks is plotted.
"""
import pandas as pd
import matplotlib.pyplot as plt

class TwoWeekReturns:
    """
    This class calculates two-week returns for each stock from the provided
    data file.
    """
    def __init__(self, data_file):
        self.data = pd.read_csv(data_file)
        
    def calculate_returns(self):
        # Calculate the two-week returns for each stock
        self.data['Date'] = pd.to_datetime(self.data['Date'])
        self.data.set_index('Date', inplace=True)
        returns = self.data.resample('W-THU').last().pct_change(periods=1)  
        # Weekly returns (considering Thursday as end-of-week)
        returns.dropna(inplace=True)
        return returns

#Usage:
returns_calculator = TwoWeekReturns(
    "C:/Users/user/Desktop/Module 5 data/tr_eikon_eod_data.csv")
two_week_returns = returns_calculator.calculate_returns()

class TradingStrategy:
    """
    This class implements a trading strategy based on two-week returns.
    """
    
    def __init__(self, returns, initial_capital):
        self.returns = returns
        self.capital = initial_capital
        self.portfolio = []
        self.portfolio_returns = []
        self.stock_returns = {}
        self.colors = ['red', 'green', 'blue'] 
        # Define colors for each stock return
        
    def rebalance_portfolio(self):
        # Select the three best performers
        best_performers = self.returns.mean().nlargest(3).index
        
        # Rebalance the portfolio by investing equally in the best performers
        investment_per_stock = self.capital / len(best_performers)
        self.portfolio = {stock: investment_per_stock for stock in \
                          best_performers}
        
    def kelly_criterion(self):
        # Calculate Kelly's criterion for optimal allocation
        mean_returns = self.returns.mean()
        var_returns = self.returns.var()
        kelly_fraction = (mean_returns / var_returns).sum() / len(mean_returns)
        optimal_allocation = kelly_fraction * self.capital
        return optimal_allocation
    
    def run_strategy(self):
        # Rebalance portfolio and calculate optimal allocation
        self.rebalance_portfolio()
        
        # Track individual stock returns
        for idx, stock in enumerate(self.portfolio):
            self.stock_returns[stock] = self.returns[stock].cumsum() *\
                self.portfolio[stock], self.colors[idx] 
                # Cumulative returns for each stock
        
        # Calculate portfolio return after two weeks
        portfolio_return = sum(self.returns[stock].mean() * \
                               self.portfolio[stock] for stock in\
                               self.portfolio)
        self.portfolio_returns.append(portfolio_return)
        
        # Print results
        print("Portfolio:", self.portfolio)
        print("Portfolio Return after Two Weeks:", portfolio_return)
        
    def plot_line_graph(self):
        # Plot line graph of returns for each stock over two weeks
        plt.figure(figsize=(10, 6))
        for stock, (returns, color) in self.stock_returns.items():
            plt.plot(returns.index, returns.values, linestyle='-', \
                     color=color, label=f'{stock} Returns')
        plt.xlabel('Date')
        plt.ylabel('Cumulative Return')
        plt.title('Cumulative Returns for Each Stock over Two Weeks')
        plt.legend(loc='upper left')
        plt.grid(True)
        plt.show()

# Usage:
strategy = TradingStrategy(two_week_returns, 10000)
strategy.run_strategy()
strategy.plot_line_graph()
