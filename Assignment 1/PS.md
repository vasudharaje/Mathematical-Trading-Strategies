Daily Returns, Cumulative Returns, Max Drawdowns, the Sharpe Ratio, and Sortino Ratio for 5 different equities and international indices
# %%
import yfinance as yf
import pandas as pd
import numpy as np
import math

# %%
equities = ["AAPL","AMZN","TSLA","MSFT","NVDA"]
indices = ["^GSPC","^GDAXI","^IXIC","^N225","^XAX"]

eq_data = []
ind_data = []

for i in range(5):
    data1 = yf.download(equities[i],"2010-01-01","2023-05-01")
    data2 = yf.download(indices[i],"2010-01-01","2023-05-01")
    eq_data.append(data1)
    ind_data.append(data2)


# %% [markdown]
# # Daily Returns and Cumulative Returns:
# 
# Here I assume all stocks are bought and sold at opening and closing prices on the same day.

# %%
#adding daily returns to the list itself
for i in range(5):
    eq_data[i]['Daily Returns'] = 100*(eq_data[i]['Close'] - eq_data[i]['Open'])/(eq_data[i]['Open'])
    ind_data[i]['Daily Returns'] = 100*(ind_data[i]['Close'] - ind_data[i]['Open'])/(ind_data[i]['Open'])

#cumulative returns
eq_cr = []
ind_cr = []
for i in range(5):
    eq_cr.append(100*(eq_data[i]['Close'][-1] - eq_data[i]['Open'][0])/eq_data[i]['Open'][0])
    ind_cr.append(100*(ind_data[i]['Close'][-1] - ind_data[i]['Open'][0])/ind_data[i]['Open'][0])

# %% [markdown]
# # Maximum drawdown

# %%
df_mdd = []
for i in range(5):
    data = list(eq_data[i]['Close'])
    days = len(data)
    peak = data[0]
    mdd = []
    n = []
    for i in range(1, days):
        if data[i] >= peak:
            if n:
                mdd.append(100*(peak - min(n))/peak)
            peak = data[i]
            n = []
        else:
            n.append(data[i])
    df_mdd.append(-1*max(mdd))

# %% [markdown]
# # Volatility

# %%
eq_vol = []
ind_vol = []
for i in range(5):
    days1 = eq_data[i].shape[0]
    days2 = ind_data[i].shape[0]
    vol = []
    for j in range(1, days1):
        vol.append(100*np.log(eq_data[i]['Close'][j] / eq_data[i]['Close'][j-1]))
    eq_vol.append(np.std(vol))
    for j in range(1, days2):
        vol.append(100*np.log(ind_data[i]['Close'][j] / ind_data[i]['Close'][j-1]))
    ind_vol.append(np.std(vol))

eq_anvol = [math.sqrt(252)*i for i in eq_vol]
ind_anvol = [math.sqrt(252)*i for i in ind_vol]

# %% [markdown]
# # Sharpe Ratio
# Taking risk free rate as 0.01%

# %%
risk_free_return = 0.01
for i in range(5):
    eq_exc_return =  eq_data[i]['Daily Returns'] - risk_free_return
    eq_sr = np.mean(eq_exc_return)/np.std(eq_exc_return)
    ind_exc_return =  ind_data[i]['Daily Returns'] - risk_free_return
    ind_sr = np.mean(ind_exc_return)/np.std(ind_exc_return)


# %% [markdown]
# # Final Table

# %%
col1 = ["Metrics","AAPL","AMZN","TSLA","MSFT","NVDA"]
fin1 = pd.DataFrame(columns=col1)

row = ['Cummulative Returns'] + eq_cr[:5]
row = pd.Series(row, index=fin1.columns)
fin1 = fin1.concat(row, ignore_index=True)

row = ['Volatility'] + eq_anvol[:5]
row = pd.Series(row, index=fin1.columns)
fin1 = fin1.concat(row, ignore_index=True)

row = ['Sharpe Ratio'] + eq_sr[:5]
row = pd.Series(row, index=fin1.columns)
fin1 = fin1.concat(row, ignore_index=True)

row = ['Max Drawdown'] + eq_mdd[:5]
row = pd.Series(row, index=fin1.columns)
fin1 = fin1.concat(row, ignore_index=True)

col2 = ["Metrics","^GSPC","^GDAXI","^IXIC","^N225","^XAX"]
fin2 = pd.DataFrame(columns=col2)

row = ['Cummulative Returns'] + ind_cr[:5]
row = pd.Series(row, index=fin2.columns)
fin2 = fin2.concat(row, ignore_index=True)

row = ['Volatility'] + ind_anvol[:5]
row = pd.Series(row, index=fin2.columns)
fin2 = fin2.concat(row, ignore_index=True)

row = ['Sharpe Ratio'] + ind_sr[:5]
row = pd.Series(row, index=fin2.columns)
fin2 = fin2.concat(row, ignore_index=True)

row = ['Max Drawdown'] + ind_mdd[:5]
row = pd.Series(row, index=fin2.columns)
fin2 = fin2.concat(row, ignore_index=True)


