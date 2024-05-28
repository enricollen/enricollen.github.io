---
title: Bike Rental Demand Prediction
date: 2022-07-22 21:30 +0200
categories: [Master Degree Projects, Bike Rental Demand Prediction]
tags: [machine learning, random forest, MLP, time series]
---
# Bike Rental Demand Predictor

🔍 Retailing and AI-based logistics optimizations.

This project aims to optimize logistics for companies in the outdoor rental field, such as electric scooters and bicycles, by predicting demand using historical rental data. 
In particular, the regressors used learn from historical data series of past rentals and provide a prediction regarding the demand for the upcoming days. 

Some determining factors are mainly related to:

* 🌦️ Weather (uncertain weather, rain, wind, humidity, etc.)
* 📅 Day of the week ( working day, holiday, weekend, etc.)
* 🌙 Time of day (night, day, etc.)


🎯 The aim is to provide a support tool to estimate with a certain degree of probability the days/times when regular maintenance can be scheduled or increase the number of vehicles in case of periods with higher demand, or distribution in different rental points.

This project was originally develop for the course 'Business & Project Management' in MSc in Artificial Intelligence & Data Engineering at University of Pisa.

### 🔄 Use Cases
- **Routine Maintenance**: Schedule during low-demand periods.
- **Dynamic Vehicle Provisioning**: Increase availability during anticipated demand spikes.
- **Demand Forecasting**: Uses AI to predict future demand to plan maintenance or adjust vehicle availability.

## 🏗️ Architectures Used
Two predictive models were developed:
1. **Multilayer Perceptron (MLP)**: A simple neural network model used for regression tasks, which was tuned for our task.
2. **Random Forest Regressor**: An ensemble learning method for regression that improves prediction accuracy by averaging multiple deep decision trees trained on different parts of the dataset.

### 🎯 Results and Observations
- **MLP Model**: Achieved better performance in predicting local minima and was more sensitive to fluctuations in demand.
- **Random Forest Regressor**: Provided robust predictions across a wide range of conditions but was less effective at predicting peaks in demand.

## 🔗 GitHub Repository
Visit the project repository [here](https://github.com/enricollen/bike-rental-demand-prediction) for project documentation and access to the codebase.

## Screenshots 📸
Here is a screenshot illustrating the prediction accuracy of the MLP model covering 2 weeks approximately:
![Screenshot 1](assets/img/posts/bike_rental_prediction/predictions.jpg)