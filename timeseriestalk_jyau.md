
Time Series Forecasting using Statistical and Machine Learning Models: When and How
============================================================================
by Jeffrey Yau, October 17 2018

### Description
Time series forecasting techniques are applied in a wide range of scientific disciplines, business scenarios, and policy settings. This presentation discuss the applications of statistical time series models, such as ARIMA, VAR, and Regime Switching Models, and machine learning models, such as random forest and neural network-based models, to forecasting problems.

### Abstract
Time series data is ubiquitous: daily term structure of interest rates, tick level stock prices, daily foreign currency exchange rates, weekly initial unemployment claim, monthly company sales, daily foot traffic recorded by mobile devices, and daily number of steps taken recorded by a wearable, just to name a few.

Some of the most important and commonly used data science techniques in time series forecasting are those developed in the field of statistics and machine learning. A few basic time series statistical and machine learning modeling techniques for forecasting should be included in any data scientist’s toolkit.

This presentation discusses the application of statistical and machine learning models in real-world time series forecasting. Statistical models covered include Seasonal Autoregressive Integrated Moving Average (SARIMA) Model, Vector Autoregressive (VAR) Model, and Regime Switching Models, and machine learning models covered include Tree-based models and Neural Network-based models. I will discuss the advantages and disadvantages when using each of these models in time series forecasting scenarios. Real-world applications, demonstrated using jupyter notebooks, are used throughout the presentation to illustrate these techniques. While not the focus in this presentation, exploratory time series data analysis will also be included in the presentation.

This presentation is suitable for data scientists who have working knowledge of the classical linear regression model and a basic understanding of univariate time series models, such as the class of Seasonal Autoregressive Integrated Moving Average Models, and machine learning techniques.


Talk Sections:
Section 1. charateristics of time series problems; problem formulation
Section 2. statistical and machine learning approaches
Section 3. Approach comparison




### Models

1. naive, rule-based model

2. "Rolling" average model

3. more sophisticated models

*Forecasting*: predicting future values of the series using current set of info



### Data and modeling prep steps

- Time Series EDA
- series transformation
    - applying first differencing and ...
    
- model implementation via scikit-learn/stats models
- model diagnostic
    - remaining noise/error terms should be iid

**-> Lecturer suggested checking out a [3-hr online Tutorial] (https://www.youtube.com/watch?v=tJ-O3hk1vRw) at PyData SanFran in 2016 on Youtube**


##### Model formulation

Markov Switching Model: A Markov Chain model relates the past state to the current state. 
  - ex: A simple version with 2 states


#### ARMA Autoregressive Moving Average Model

Future value Y(t+H) is a function of previous Yis and a series of "shocks" or error terms.


#### ARIMA model/ Univariate statiscal time series models

An ARIMA model is ARMA applied/extended to a non-stationary series. 

- statistical relationship is uni-directional
  - the future is a function of the past


#### Seasonal ARIMA model

- applied to non-stationary series only
- auto-covariates only a function of the time-lag

#### Vector AutoRegressive model (VAR)

- a type of multivariate time series modeling
- a system of K linear equations estimated simultaneously
- only applies to stationary series
    - non-stationary series can be transformed into stationary ones using differencing
    - Feedback effects: K equations where a given Yi,t can be affected by Yj, t-1 of other equations
- example: 2 series of consumer sentiment and beer consumption
    - use VARMAX function in python
- model selection
- final needed step: inverse transform back to the original form Yt+H

- VAR model forecast using RMSE, root mean squared error
  - need to put the RMSE results in the original context or scale to get to "Percentage error relative to the mean"
  

#### (RNN) Recurrent Neural Network

- forecast output is a function of not only input Xt but also previous period h(t-1)
- implemented in Keras, Python's deep learning libary
    - LSTM model: formulate the RNN as a regularized linear ?regression model

### Approach comparison

**For uni-variate time series analyis**
- ARIMA - is state invariant
- MSAR - is state-dependent


**For multi-variate time series**
- VAR: 
    - numerical time series such as financial and business time series
    - applied to stationary series
    - data pre-processing is straightforward
    
- LSTM: images, texts, and all numerical time series
  - data preprocessing is more involved, time consuming


