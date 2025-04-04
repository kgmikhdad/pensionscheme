# Implementing the Methodology: NPS vs UPS Analysis

## 1. Portfolio Simulation and Efficient Frontier Construction

To compare pension fund performance, we first simulate **portfolio returns** and construct the **efficient frontier**. We treat each asset class (e.g. equity, corporate bonds, government bonds) as a separate asset and compute its expected return (e.g. historical mean) and covariance matrix ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=Expected%20Returns%3A%20The%20expected%20returns,for%20each%20asset)) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=Covariance%20Matrix%3A%20A%20covariance%20matrix,between%20equity)). Using these, we iterate over possible weight allocations (summing to 100%) across the three assets to compute each **portfolio’s return and risk (standard deviation)** ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=%E2%80%A2%20Portfolio%20Return%20)) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=expected%20return%20vector)). For each portfolio, we also compute the Sharpe ratio given an assumed risk-free rate ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=%E2%80%A2%20Sharpe%20Ratio%20)). The efficient frontier is the set of portfolios offering the **maximum return for a given level of risk** (i.e. the Pareto-optimal risk-return combinations) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=The%20Efficient%20Frontier%20is%20constructed,by%20iterating%20through%20all)). 

**Code – Efficient Frontier:** In Python, we can brute-force weight combinations or use optimization libraries. Below, we simulate random asset returns and use a grid-search over weights to plot the efficient frontier. We highlight the **tangency portfolio** (max Sharpe ratio) as well. This approach follows the method described by Markowitz’s modern portfolio theory ([github.com](https://github.com/psthomas/efficient-frontier/raw/refs/heads/master/efficient_frontier.ipynb#:~:text=ge%5D%28https%3A%2F%2Fupload.wikimedia.org%2Fwikipedia%2Fcommons%2Fe%2Fe1%2FMarkowitz_frontier.jpg%29%20%5Cn,of%20my%20concept%2C%20feel%20free)), iterating over weight combinations to find the frontier ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=The%20Efficient%20Frontier%20is%20constructed,by%20iterating%20through%20all)).
```python

import numpy as np
import matplotlib.pyplot as plt

# Simulate expected returns and covariance for 3 assets (e.g., Equity, CorpBond, GovBond)
mu = np.array([0.15, 0.07, 0.05])  # expected returns for each asset
cov = np.array([                 # covariance matrix of asset returns
    [0.0400, 0.0040, 0.0020],
    [0.0040, 0.0100, 0.0010],
    [0.0020, 0.0010, 0.0025]
])
rf = 0.07  # assumed risk-free rate 7%&#8203;:contentReference[oaicite:8]{index=8}

# Enumerate portfolios by looping over weights w1, w2, w3 that sum to 1
weights_list = []
for w1 in np.linspace(0, 1, 101):
    for w2 in np.linspace(0, 1 - w1, 101):
        w3 = 1 - w1 - w2
        weights_list.append([w1, w2, w3])
weights = np.array(weights_list)

# Compute portfolio return and risk for each weight combination
port_returns = weights.dot(mu)                     # μ_p = w^T μ
port_vars = np.einsum('ij,jk,ik->i', weights, cov, weights)  # σ_p^2 = w^T Σ w
port_risks = np.sqrt(port_vars)

# Identify efficient frontier (highest return for each risk level)
# Sort portfolios by risk and then filter by return
portfolios = sorted(zip(port_risks, port_returns, weights_list), key=lambda x: x[0])
efficient_pts = []
max_ret = -np.inf
for risk, ret, w in portfolios:
    if ret > max_ret:
        efficient_pts.append((risk, ret))
        max_ret = ret
eff_risks, eff_returns = zip(*efficient_pts)

# Plot all portfolios and highlight the efficient frontier
plt.figure(figsize=(6,4))
plt.scatter(port_risks, port_returns, c=(port_returns-rf)/port_risks, cmap='viridis', s=10, label='All Portfolios')
plt.plot(eff_risks, eff_returns, 'r-o', label='Efficient Frontier', linewidth=2)
# Mark the max Sharpe portfolio
sharpe = (port_returns - rf) / port_risks
idx_max_sharpe = np.argmax(sharpe)
plt.scatter(port_risks[idx_max_sharpe], port_returns[idx_max_sharpe], color='blue', marker='X', s=100, label='Max Sharpe')
plt.xlabel('Risk (Std Dev)')
plt.ylabel('Expected Return')
plt.title('Efficient Frontier of Simulated Portfolio')
plt.legend()
plt.show()

``` 
 ([image]()) *Output:* *Simulated efficient frontier for a 3-asset portfolio. Each dot is a portfolio; the red line is the efficient frontier. We see the classic upward-sloping concave frontier (higher returns require higher risk). A color gradient indicates Sharpe ratio, and the blue ‘X’ marks the portfolio with maximum Sharpe ratio.* 

The code above generates random portfolios and computes their returns and risks. As described in the paper’s methodology, this lets us identify the optimal fund mix for a given risk-profile ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=To%20conduct%20a%20comparative%20analysis,between%20NPS%20and%20UPS)) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=Here%20we%20considered%20the%20equity%2C,corporate%20bond%2C%20and)). For instance, one can determine which pension fund manager’s asset allocation yields the best risk-adjusted return by seeing where it lies relative to the efficient frontier ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=The%20efficient%20frontier%20analysis%20for,LIC%20Pension%20Fund)). In practice, one could use optimization solvers or libraries like **PyPortfolioOpt** to directly solve for frontier points, but brute-force simulation is straightforward and aligns with the approach (“iterating through all possible weight combinations”) described in the study ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=%F0%9D%9B%94%F0%9D%92%91)).

## 2. Vector Autoregression (VAR) and Impulse Response Function (IRF) Analysis

To analyze macroeconomic sensitivity, we employ a **Vector Autoregression (VAR)** model on multiple time series (e.g. inflation, interest rates, fund returns) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=interdependencies%20between%20financial%20returns%20and)). A VAR treats each variable as potentially influenced by its own past values and the past values of all other variables ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=macroeconomic%20indicators%20%28e,repo%20rate)). Once the VAR is fitted, we can perform **Impulse Response Function (IRF)** analysis, which quantifies how a shock to one variable propagates through the system over time ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=and%20%CF%B5t%E2%88%BCN,The%20IRF)). For example, we might examine how a one-time unexpected increase in inflation (an “impulse”) affects pension fund returns or other macro variables in subsequent months ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=interdependencies%20between%20financial%20returns%20and)). 

**Code – VAR and IRF:** Using Python’s `statsmodels`, we fit a VAR model to simulated data and then compute IRFs. In this example, we simulate two economic time series `X` and `Y` with known cross-effects, fit a VAR(1), and then plot the impulse responses for a shock to each variable. The IRF output shows, for each shock (e.g. `X→X` means shock in X and its effect on X itself, `X→Y` means shock in X and effect on Y, etc.), the trajectory of the response over time along with confidence bands ([Vector Autoregressions tsa.vector_ar — statsmodels 0.6.1 documentation](https://www.statsmodels.org/0.6.1/vector_ar.html#:~:text=)).

```python
import pandas as pd
from statsmodels.tsa.api import VAR

# Simulate a bivariate VAR(1) process: X_t = 0.5*X_{t-1} + 0.2*Y_{t-1} + e1;  Y_t = -0.1*X_{t-1} + 0.4*Y_{t-1} + e2
np.random.seed(0)
T = 200
e1, e2 = np.random.normal(scale=1, size=T), np.random.normal(scale=1, size=T)
X, Y = np.zeros(T), np.zeros(T)
for t in range(1, T):
    X[t] = 0.5*X[t-1] + 0.2*Y[t-1] + e1[t]
    Y[t] = -0.1*X[t-1] + 0.4*Y[t-1] + e2[t]

data = pd.DataFrame({'X': X, 'Y': Y})
model = VAR(data)
results = model.fit(1)                   # fit VAR(1)
irf = results.irf(10)                    # 10-period impulse response
irf.plot(orth=False)                     # plot IRFs (blue: response, dashed: ±2 SE bands)
plt.show()
``` 

 ([image]()) *Output:* *Impulse Response Functions (IRFs) from the VAR model. Each subplot shows the response of a variable (row) to a shock in a variable (column). For example, the top-left is the response of X to an X shock, top-right is response of X to a Y shock, etc. The blue line is the estimated response, and dashed lines are confidence intervals.* 

In the IRF plots, we observe how a shock dissipates: e.g. a shock to `X` causes `X` itself to jump and then decay over ~5 periods, while `Y` might initially dip slightly (from the negative cross-effect) before returning to baseline. This matches the expected behavior since we constructed `X` to positively influence `Y` and vice-versa (albeit with different signs). In a real analysis, we would use actual macroeconomic and return data. The **IRF allows us to gauge the sensitivity** of pension fund returns to macro shocks ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=macroeconomic%20variables%20using%20the%20Impulse,Response)) – for instance, how a surprise inflation increase might gradually reduce fund returns over months, or how a shock to equity markets propagates to bond returns. The study uses IRFs within a VAR framework to capture such interdependencies between fund performance and macro variables ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=interdependencies%20between%20financial%20returns%20and)), with orthogonal (Cholesky) shocks used to isolate individual effects ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=and%20%CF%B5t%E2%88%BCN,The%20IRF)). This helps answer questions like *“how do NPS returns respond to a 1% inflation shock over time?”* in a dynamic setting.

## 3. Forecasting Using Machine Learning Models

For **long-term forecasting**, the study employs advanced machine learning models – specifically the **Temporal Fusion Transformer (TFT)**, **Long Short-Term Memory (LSTM) neural networks**, and **XGBoost** gradient boosting (optionally in hybrid with ARIMA) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=using%20machine%20learning%20models%20like,that%20although%20NPS%20offers%20higher)). These models are used to project macroeconomic variables (like GDP growth, inflation) and pension fund returns into the future, complementing the VAR analysis. We demonstrate each method with code using simulated data. All models will involve common steps: preparing the time-series data (including any lag features or covariates), training the model, and visualizing the forecast vs actual values.

### 3.1 Temporal Fusion Transformer (TFT)

The **Temporal Fusion Transformer** is a state-of-the-art deep learning model for multi-horizon forecasting that can incorporate both static features and time-varying covariates ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=The%20Temporal%20Fusion%20Transformer%20,is%20at%20the%20core)). In the paper’s methodology, TFT is the core forecasting model for long-term projections of GDP, inflation, bond yields, etc. ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=3,for)). It leverages attention mechanisms to learn which time steps and features are most important, and can produce interpretable predictions (e.g. quantiles). 

**Code – Using TFT (via PyTorch Forecasting):** Implementing TFT from scratch is complex, but we can use the `pytorch-forecasting` library’s built-in `TemporalFusionTransformer` class ([Demand forecasting with the Temporal Fusion Transformer — pytorch-forecasting  documentation](https://pytorch-forecasting.readthedocs.io/en/stable/tutorials/stallion.html#:~:text=from%20pytorch_forecasting%20import%20Baseline%2C%20TemporalFusionTransformer%2C,tuning%20import%20%28%20optimize_hyperparameters%2C)). Below is a conceptual snippet: we simulate a toy dataset of a seasonal time series and show how one would set up the data and model. (Note: for brevity we do not run a full training here.) This aligns with the paper’s approach of using TFT for multi-variate, multi-horizon forecasts ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=The%20Temporal%20Fusion%20Transformer%20,is%20at%20the%20core)).

```python
import pandas as pd
import numpy as np
from pytorch_forecasting import TimeSeriesDataSet, TemporalFusionTransformer

# Simulate a time series with trend and seasonality for demonstration
dates = pd.date_range("2000-01-01", periods=120, freq="M")
values = np.linspace(100, 150, 120) + 5*np.sin(np.linspace(0, 12*np.pi, 120))  # trend + seasonality
series = pd.DataFrame({"time": dates, "value": values, "group": "series1"})

# Prepare data for model (TFT expects a TimeSeriesDataSet with specified sequence length)
max_history = 24
max_forecast_horizon = 6
ts_dataset = TimeSeriesDataSet(
    series,
    time_idx="time",
    target="value",
    group_ids=["group"],
    max_encoder_length=max_history,
    max_prediction_length=max_forecast_horizon,
    time_varying_known_reals=["time"],    # here 'time' index could be converted to numeric
    time_varying_unknown_reals=["value"], # the target itself
)
# Define TFT model
tft = TemporalFusionTransformer.from_dataset(ts_dataset, learning_rate=0.01, hidden_size=16, attention_head_size=4)
# (In practice, call tft.fit() on a PyTorch Lightning trainer and then tft.predict() on new data)
``` 

In practice, after training the TFT on historical data (and perhaps proxy data as described later), we would use it to forecast multiple steps ahead. The **TFT excels at multi-step forecasting with multi-variable inputs**, which is why the study chose it to predict trajectories of GDP, etc., up to year 2070 ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=The%20Temporal%20Fusion%20Transformer%20,is%20at%20the%20core)). The model can incorporate static data (e.g. country or scenario identifiers) and dynamic covariates (e.g. future policy assumptions). For example, if we had additional features like projected population or a scenario flag, we could include them as covariates. The output of TFT can be visualized with predicted intervals. (Due to complexity, we rely on high-level APIs; see the [PyTorch Forecasting documentation] for a complete example ([Demand forecasting with the Temporal Fusion Transformer — pytorch-forecasting  documentation](https://pytorch-forecasting.readthedocs.io/en/stable/tutorials/stallion.html#:~:text=from%20pytorch_forecasting%20import%20Baseline%2C%20TemporalFusionTransformer%2C,tuning%20import%20%28%20optimize_hyperparameters%2C)).)

### 3.2 Long Short-Term Memory (LSTM) Networks

**LSTM networks** are a type of recurrent neural network well-suited for sequential data. They are used in the paper to model temporal dependencies in macroeconomic time series ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=Term%20Memory%20,XGBoost%2C%20as%20applied)) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=Long%20Short,are)). LSTMs maintain an internal state (memory cell) that allows them to capture long-term patterns and lags in the data. In our context, an LSTM can learn relationships like how past GDP and inflation values influence future values, or how pension fund NAVs evolve over time with momentum and seasonality.

**Code – LSTM Forecasting:** We demonstrate an LSTM model using Keras to predict a univariate time series. First we create a supervised learning dataset by using the last `n` observations as features to predict the next value. Then we define a simple LSTM model with one hidden layer and train it. This example uses a single sequence; in practice, one could use multiple features and a more complex architecture. (This mirrors typical time-series LSTM setups ([Time Series Analysis using LSTM Keras - Kaggle](https://www.kaggle.com/code/hassanamin/time-series-analysis-using-lstm-keras#:~:text=Time%20Series%20Analysis%20using%20LSTM,add%28Dense%281%29%29%20model)).)

```python
import numpy as np
from tensorflow.keras.models import Sequential
from tensorflow.keras.layers import LSTM, Dense

# Create a simple sinusoidal time series
np.random.seed(0)
time = np.arange(0, 100, 0.1)
y = np.sin(time) + 0.1*np.random.normal(size=len(time))  # sine wave with noise

# Prepare supervised data: use past 10 points to predict next point
look_back = 10
X_data, y_data = [], []
for i in range(len(y) - look_back):
    X_data.append(y[i:i+look_back])
    y_data.append(y[i+look_back])
X_data = np.array(X_data)
y_data = np.array(y_data)
X_data = X_data.reshape((X_data.shape[0], X_data.shape[1], 1))  # reshape to [samples, timesteps, features]

# Split into train and test
train_size = int(0.8 * len(X_data))
X_train, X_test = X_data[:train_size], X_data[train_size:]
y_train, y_test = y_data[:train_size], y_data[train_size:]

# Define LSTM model
model = Sequential()
model.add(LSTM(20, input_shape=(look_back, 1)))  # 20 LSTM units
model.add(Dense(1))  # output layer
model.compile(optimizer='adam', loss='mse')
model.fit(X_train, y_train, epochs=10, batch_size=16, verbose=0)

# Forecast on test set
y_pred = model.predict(X_test)
```

After training, we can compare `y_pred` to `y_test` to evaluate performance. The LSTM’s ability to **capture temporal patterns** comes from its recurrent nature: it processes the input sequence one time step at a time, updating its hidden (`h_t`) and cell (`c_t`) states ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=For%20a%20given%20sequence%20of,X1%2CX2%2C%E2%80%A6%2CXt%7D%2C%20the%20LSTM)). This allows it to remember information from earlier in the sequence when making predictions later – e.g. picking up a seasonal cycle or a slow-moving trend. The code above uses a univariate series for clarity, but LSTMs can handle multivariate input (just increase feature dimensions) and can be stacked in layers. The paper specifically notes using LSTMs to model sequential relationships among macro variables ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=Long%20Short,are)) – for instance, feeding in sequences of past inflation, GDP, and rates to predict future values. In our snippet, one could extend it by including additional series (as additional features) or using sequences of different variables as input.

### 3.3 XGBoost Forecasting (with optional ARIMA hybrid)

**XGBoost** is a powerful ensemble tree method that can model non-linear patterns in time series once the data is appropriately transformed (e.g. using lags as features) ([Use XGBoost for Time-Series Forecasting - Analytics Vidhya](https://www.analyticsvidhya.com/blog/2024/01/xgboost-for-time-series-forecasting/#:~:text=XGBoost%20offers%20several%20advantages%20that,series%20forecasting)). In a pure machine-learning approach, we can use XGBoost regressors to predict future values from past observations (treating it like a regression problem). Additionally, the paper’s methodology combines XGBoost with an **ARIMA** model to capture both linear and nonlinear components ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=To%20capture%20both%20linear%20trends,linear)) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=The%20ARIMA%20model%20captures%20linear,dependencies%2C%20which)). Specifically, an ARIMA might first model the baseline linear trend, and XGBoost is then used on the residuals to explain additional variance (or vice versa). This hybrid approach leverages ARIMA for what it does best (linear, short-memory patterns) and XGBoost for complex nonlinear relationships ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=To%20capture%20both%20linear%20trends,linear)). Such ARIMA+XGBoost (or more general statistical+ML) hybrids have been found effective in M4 forecasting competitions and other studies ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=Term%20Memory%20,XGBoost%2C%20as%20applied)).

**Code – XGBoost Forecasting:** Below we illustrate using XGBoost to forecast a time series by creating lag features. We simulate a time series with trend and seasonality and then train an `XGBRegressor` to predict the next value given the previous 5 values. We then produce an out-of-sample forecast for the test period. (In practice, one might also include exogenous variables or more sophisticated feature engineering ([Use XGBoost for Time-Series Forecasting - Analytics Vidhya](https://www.analyticsvidhya.com/blog/2024/01/xgboost-for-time-series-forecasting/#:~:text=Before%20applying%20XGBoost%20to%20time,resamplin%20to%20ensure%20a%20consistent)) ([Use XGBoost for Time-Series Forecasting - Analytics Vidhya](https://www.analyticsvidhya.com/blog/2024/01/xgboost-for-time-series-forecasting/#:~:text=,1%3A%20Data%20Cleaning%20and%20Preprocessing)).)

```python
from xgboost import XGBRegressor

# Simulate a time series (trend + seasonality + noise)
np.random.seed(42)
T = 120
t = np.arange(T)
y = 0.1*t + 5*np.sin(2*np.pi*t/12) + np.random.normal(0, 0.5, T)  # trend + annual seasonality

# Prepare data with lag features (lags 1-5)
lags = 5
X = []
y_target = []
for i in range(lags, T):
    X.append(y[i-lags:i])
    y_target.append(y[i])
X = np.array(X); y_target = np.array(y_target)

# Split into train/test
train_size = 100
X_train, X_test = X[:train_size], X[train_size:]
y_train, y_test = y_target[:train_size], y_target[train_size:]

# Train XGBoost model
model = XGBRegressor(n_estimators=100, max_depth=3)
model.fit(X_train, y_train)

# Predict iteratively for test points
y_pred = []
history = list(y_target[:train_size])  # start with training data
for i in range(len(y_test)):
    x_input = np.array(history[-lags:]).reshape(1, -1)
    pred = model.predict(x_input)[0]
    y_pred.append(pred)
    history.append(y_test[i])  # or append pred for a fully autonomous forecast

# Compare predictions to actual
import matplotlib.pyplot as plt
plt.plot(range(T), y, label="Actual")
plt.axvline(train_size, color='gray', linestyle='--', label="Train/Test split")
plt.plot(range(train_size, T), y_pred, label="XGBoost Forecast", marker='o')
plt.plot(range(train_size, T), y_test, label="Actual Future", marker='x')
plt.legend(); plt.xlabel('Time'); plt.ylabel('Value'); plt.show()
``` 

*(The plot will show the model tracking the general trend and seasonality in the test period, though with some deviation.)* 

In this example, XGBoost captures the sinusoidal seasonality and upward trend reasonably well by learning from the lagged patterns. Feature engineering is crucial: we used recent lags as features (a common approach to use tree models for time series ([[D] How does xgboost work with time series? : r/MachineLearning](https://www.reddit.com/r/MachineLearning/comments/1aoo7gc/d_how_does_xgboost_work_with_time_series/#:~:text=r%2FMachineLearning%20www,for%20each%20window%20in%20time)) ([Use XGBoost for Time-Series Forecasting - Analytics Vidhya](https://www.analyticsvidhya.com/blog/2024/01/xgboost-for-time-series-forecasting/#:~:text=Before%20applying%20XGBoost%20to%20time,resamplin%20to%20ensure%20a%20consistent))). We could add more features (month of year, moving averages, etc.) to improve accuracy. The **hybrid ARIMA-XGBoost** approach would typically involve two steps: (1) fit an ARIMA on the series to forecast the linear part, (2) compute the residuals (actual – ARIMA forecast) and train XGBoost on those residuals (using lag features or other exogenous inputs), and then (3) add the XGBoost-predicted residual to the ARIMA’s forecast ([Time Series : State Space + XGBoost Hybrid||Mamba - Kaggle](https://www.kaggle.com/code/waqasali51/time-series-state-space-xgboost-hybrid-mamba#:~:text=Time%20Series%20%3A%20State%20Space,XGBoost%2C%20short%20for)). The paper implements a version of this: ARIMA captures baseline trends and XGBoost then models the remaining structure ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=To%20capture%20both%20linear%20trends,linear)) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=The%20ARIMA%20model%20captures%20linear,dependencies%2C%20which)). This helps ensure that well-understood patterns (like a steady growth rate) are handled by ARIMA, while XGBoost can focus on complex interactions or regime changes. 

## 4. Scenario-Based Macroeconomic Simulation with Proxy Countries

Finally, the study introduces a **scenario-based simulation** to project India’s macroeconomic future using trajectories of similar “proxy” countries ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=3)). The idea is to leverage the historical data of countries like **Brazil, China, and South Korea** – which are 20–40 years ahead of India in development – as analogues for what India’s future might look like ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=structurally%20and%20demographically%20similar%20to,India%2C%20are%2020%E2%80%93)). By aligning India’s current economic indicators with a point in another country’s past, we can use that country’s subsequent path as a **scenario for India**. For example, we might assume *“India’s GDP growth from 2025 onward follows the trajectory that South Korea experienced starting in 1985”*. Multiple scenarios (e.g. a high-growth scenario akin to China’s boom, a moderate scenario like South Korea, and a low-growth scenario) are considered. Additionally, known future targets or projections (like certain GDP levels by 2030, 2050, etc.) are used as **constraints** to keep simulations realistic ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=3)) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=GDP%20projections%20for%20India%2C%20spanning,2025%E2%80%932029%2C%202036)).

**Code – Scenario Simulation:** We illustrate scenario generation with a simplified example. Suppose we have three proxy growth trajectories (fast, medium, slow). We’ll simulate GDP index values for 45 years under each scenario. We align all at a common starting point (index = 100) for year 0 (which represents India’s current GDP level normalized). Then each year’s growth rate is applied. In a real case, the alignment would involve finding the offset τ such that India’s current GDP per capita equals the proxy country’s GDP at some past year ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=simulate%20India%27s%20future%20economic%20conditions,Using)), and then mapping forward. Here we assume year0 is aligned for simplicity.

```python
import numpy as np
import matplotlib.pyplot as plt

years = np.arange(0, 46)  # 46 years, e.g. 2025 to 2070
start_val = 100.0

# Scenario A: Fast growth (e.g., like China's trajectory)
growth_A = [0.08 * np.exp(-0.05*t) + 0.02 for t in years]  # starts ~8%, decays to ~2%
GDP_A = [start_val]
for t in range(1, len(years)):
    GDP_A.append(GDP_A[-1] * (1 + growth_A[t]))

# Scenario B: Moderate growth (e.g., like South Korea's trajectory)
growth_B = [0.05 * np.exp(-0.03*t) + 0.015 for t in years]  # starts ~5%, to ~1.5%
GDP_B = [start_val]
for t in range(1, len(years)):
    GDP_B.append(GDP_B[-1] * (1 + growth_B[t]))

# Scenario C: Slow growth (e.g., lower steady growth)
growth_C = [0.03 for t in years]  # constant ~3% growth
GDP_C = [start_val]
for t in range(1, len(years)):
    GDP_C.append(GDP_C[-1] * (1 + growth_C[t]))

# Plot the scenario trajectories
plt.plot(years, GDP_A, label="Scenario A (Fast Growth)")
plt.plot(years, GDP_B, label="Scenario B (Moderate Growth)")
plt.plot(years, GDP_C, label="Scenario C (Slow Growth)")
plt.xlabel("Years from now"); plt.ylabel("GDP (index)"); plt.legend(); plt.show()
``` 

 ([image]()) *Output:* *Simulated GDP index under three scenarios. All start at 100 (Year 0). Scenario A grows rapidly then plateaus (red); B grows moderately; C grows slowly but steadily. Such scenarios emulate proxy countries’ historical paths.* 

In the plot above, **Scenario A** (perhaps analogous to China's boom) shows a sharp early rise in GDP (8% growth initially) that slows over decades as the economy matures. **Scenario B** (like South Korea) has a more modest rise, while **Scenario C** is a slow-growth scenario. In the context of the paper, these scenarios would be used to drive forecasts: for instance, feeding these GDP paths (and possibly other variables like inflation) into the forecasting models (TFT, LSTM, etc.) to see how pension outcomes differ under each scenario. The code here is deterministic, but one could introduce randomness or shocks for Monte Carlo simulation of variability.

In the study, *scenario training* means the models were trained on data including those proxy country trajectories ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=3)). Essentially, the model learns from how those countries evolved (with appropriate time lag alignment) and applies that knowledge to India. Additionally, **constraints** such as hitting certain GDP targets by specific years were enforced. For example, if an official projection says “India’s GDP in 2047 will be \$X trillion”, the scenario paths can be adjusted to ensure they meet that constraint (perhaps by slightly tweaking growth rates). This approach blends data-driven forecasting with policy targets.

Finally, using these scenarios, the **pension scheme outcomes** can be evaluated. For instance, we can feed the scenario-based macro variables into our earlier portfolio simulation or VAR to estimate things like the fiscal liability under UPS vs NPS in each scenario. The methodology’s flowchart (Figure 2 in the paper) indicates that macro scenarios inform the forecast of scheme returns ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=Figure%202%20,Algorithm%20for%20Forecasting%20Scheme%20Returns)) ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=%F0%9D%91%85%F0%9D%91%A1%20%F0%9D%91%86%20%3D%20%F0%9D%91%A4%F0%9D%90%B8%F0%9D%91%85%F0%9D%91%A1)). This allows a comprehensive analysis: e.g., *if India experiences a China-like boom, NPS returns could be much higher (but volatility too), whereas a stagnation scenario could make UPS’s defined benefit more attractive.* 

**References:** The implementations above draw from both the paper’s methodology and external sources. For efficient frontier and portfolio simulation, we referenced a Quantopian blog on Markowitz optimization ([github.com](https://github.com/psthomas/efficient-frontier/raw/refs/heads/master/efficient_frontier.ipynb#:~:text=ge%5D%28https%3A%2F%2Fupload.wikimedia.org%2Fwikipedia%2Fcommons%2Fe%2Fe1%2FMarkowitz_frontier.jpg%29%20%5Cn,of%20my%20concept%2C%20feel%20free)). VAR modeling and IRF plotting utilized StatsModels documentation ([Vector Autoregressions tsa.vector_ar — statsmodels 0.6.1 documentation](https://www.statsmodels.org/0.6.1/vector_ar.html#:~:text=)). The TFT usage follows the PyTorch Forecasting library tutorial for the Stallion dataset ([Demand forecasting with the Temporal Fusion Transformer — pytorch-forecasting  documentation](https://pytorch-forecasting.readthedocs.io/en/stable/tutorials/stallion.html#:~:text=from%20pytorch_forecasting%20import%20Baseline%2C%20TemporalFusionTransformer%2C,tuning%20import%20%28%20optimize_hyperparameters%2C)). The LSTM example structure was inspired by common Keras patterns ([Time Series Analysis using LSTM Keras - Kaggle](https://www.kaggle.com/code/hassanamin/time-series-analysis-using-lstm-keras#:~:text=Time%20Series%20Analysis%20using%20LSTM,add%28Dense%281%29%29%20model)). The XGBoost approach follows best practices of feature creation for time series ([Use XGBoost for Time-Series Forecasting - Analytics Vidhya](https://www.analyticsvidhya.com/blog/2024/01/xgboost-for-time-series-forecasting/#:~:text=Before%20applying%20XGBoost%20to%20time,resamplin%20to%20ensure%20a%20consistent)) and aligns with hybrid methods discussed in forecasting literature ([Time Series : State Space + XGBoost Hybrid||Mamba - Kaggle](https://www.kaggle.com/code/waqasali51/time-series-state-space-xgboost-hybrid-mamba#:~:text=Time%20Series%20%3A%20State%20Space,XGBoost%2C%20short%20for)). The scenario analysis approach is guided by the paper’s description of using proxy country data ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=3)) and is analogous to “what-if” analyses in financial planning ([Part 2: Make Your Time Series Model Actionable with Scenario Analysis: From Basic to Advanced Methods | by Tim Zhao | Medium](https://medium.com/@tzhaonj/part-2-make-your-time-series-model-actionable-with-scenario-analysis-from-basic-to-advanced-b53137b39fe9#:~:text=To%20simulate%20different%20growth%20rates%2C,can%20adjust%20the%20forecasted%20values)). By combining these components – portfolio optimization, VAR/IRF, ML forecasting, and scenario analysis – we can implement the full methodology to compare NPS and UPS pension outcomes under various economic futures, as done in the study ([ssrn-5088591 (1).pdf](file://file-EyUFz5wmbUppyo6CohQVrj#:~:text=using%20machine%20learning%20models%20like,that%20although%20NPS%20offers%20higher)). This provides a robust computational experiment to support policy conclusions on pension reforms.
