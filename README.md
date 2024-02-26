# faye-yang
To provide a comprehensive guide for researchers and policymakers on using interrupted time series (ITS) analysis with autoregressive integrated moving average (ARIMA) models to evaluate the impact of large-scale health interventions. ITS analysis is a powerful statistical technique for assessing the effect of interventions over time, especially when randomized controlled trials are not feasible. ARIMA models, which account for underlying trends, seasonality, and autocorrelation in time series data, enhance the robustness of ITS analyses. The objective is to improve the understanding and application of ITS-ARIMA in evaluating health policy and program interventions, ensuring accurate, credible results that can inform policy decisions and public health practice.
\item Methodology:
The guide will outline the step-by-step methodology for conducting ITS analysis using ARIMA models, including:
Data Requirements: Specification of the types of data needed, such as continuous and evenly spaced time series data on health outcomes before and after the intervention.
Model Specification: Guidance on selecting appropriate ARIMA models based on the data's characteristics, including the process of identifying the order of autoregression, degree of differencing, and moving average components.
Intervention Analysis: Techniques for estimating the intervention's impact, including changes in level and trend of the health outcome after the intervention, while controlling for pre-intervention trends.
Model Validation: Methods for validating the fit of the ARIMA model, including diagnostics checks for autocorrelation, partial autocorrelation, and residuals analysis.
\item Expected Outcomes:
The guide is expected to:
Enhance researchers' capacity to rigorously evaluate health interventions using time series analysis.
Provide a standardized approach to ITS-ARIMA analysis, facilitating comparability across studies.
Offer insights into the interpretation of ITS-ARIMA results, including understanding the magnitude and significance of the intervention's impact.
\item Policy Implications:
By improving the quality of evaluation research, the guide aims to support evidence-based policymaking in health. It will help policymakers and public health officials make informed decisions about the continuation, modification, or termination of health interventions based on their demonstrated effectiveness.
\item Target Audience:
The guide is intended for epidemiologists, health economists, public health researchers, and policymakers involved in the design, implementation, and evaluation of health interventions.
\item Dissemination:
Plans for disseminating the guide include publication in peer-reviewed journals, presentations at public health and epidemiology conferences, and workshops for health policymakers and researchers. An online version of the guide will also be made available to ensure broad access.
\item Collaboration and Support:
The development of the guide will involve collaboration with experts in epidemiology, biostatistics, and health policy. Support from public health organizations and research institutions will be sought to ensure the guide's relevance and applicability across various health interventions and contexts.

```{r}
library(astsa)
library(forecast)
library(dplyr)
library(zoo)

# Load data
quet <- read.csv(file = 'quiet.csv')

# Convert data to time series object
quet.ts <- ts(quet[,2], frequency=12, start=c(2011,1))

# View data
quet.ts

# Plot data to visualise time series
options(scipen=5)
plot(quet.ts, ylim=c(0,40000), type='l', col="blue", xlab="Month", ylab="Dispensings")
# Add vertical line indicating date of intervention (January 1, 2014)
abline(v=2014, col="gray", lty="dashed", lwd=2)

# View ACF/PACF plots of undifferenced data
acf2(quet.ts, max.lag=24)

# View ACF/PACF plots of differenced/seasonally differenced data
acf2(diff(diff(quet.ts,12)), max.lag=24)

# Create variable representing step change and view
step <- as.numeric(as.yearmon(time(quet.ts))>='Jan 2014')
step

# Create variable representing ramp (change in slope) and view
ramp <- append(rep(0,36), seq(1,12,1))
ramp  

# Use automated algorithm to identify p/q parameters
# Specify first difference = 1 and seasonal difference = 1
model1 <- auto.arima(quet.ts, seasonal=TRUE, xreg=cbind(step,ramp), max.d=1, max.D=1, stepwise=FALSE, trace=TRUE)

# Check residuals
checkresiduals(model1)
Box.test(model1$residuals, lag = 24, type = "Ljung-Box")

# Estimate parameters and confidence intervals
summary(model1)
confint(model1)

# To forecast the counterfactual, model data excluding post-intervention time period
model2 <- Arima(window(quet.ts, end=c(2013,12)), order=c(2,1,0), seasonal=list(order=c(0,1,1), period=12))

# Forecast 12 months post-intervention and convert to time series object
fc <- forecast(model2, h=12)
fc.ts <- ts(as.numeric(fc$mean), start=c(2014,1), frequency=12)

# Combine with observed data
quet.ts.2 <- ts.union(quet.ts, fc.ts)
quet.ts.2

# Plot
plot(quet.ts.2, type="l", plot.type="s", col=c('blue','red'), xlab="Month", ylab="Dispensings", linetype=c("solid","dashed"), ylim=c(0,40000))
abline(v=2014, lty="dashed", col="gray")

```
