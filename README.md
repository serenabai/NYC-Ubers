# Visualizing and Predicting Uber Driver Pay in NYC


## Data Overview 
We are analyzing high volume for-hire vehicles (FHV) trip data. The data is from the NYC Taxi and Limousine Commission (TLC) and collected by technology providers authorized under the Taxicab & Livery Passenger Enhancement Programs. We decided to subset the data to only Uber trips for the first full week of April 2022 (April 3rd-10th) as the overall April 2022 dataset is too large, with over 16 million rows of data. Our data of interest filters only for Uber rides that were taken in NYC, which represents approximately 73% of the rides taken. We also chose to include weather data sourced from the open-meteo API in order to see how precipitation affected driver pay. 

Though our data represents just a sample of the Uber rides taken in the country because we are only focusing on NYC, we are unable to see the data for Uber rides spanning the U.S. However, we can compare the week of data from April 2022 to the NYC Uber rides data from the entirety of the month of April 2022. The collected data represents every single taxi or ride-share ride that was taken in NYC. Most riders and drivers may be unaware that this data was collected, but the NYC Taxi and Limousine Commission authorizes the collection of this data. Essentially, ride-share providers such as Uber and Lyft report their NYC trip data to the TLC.

Each row in our dataset represents a single trip in an Uber dispatched by one of NYC’s licensed high volume FHV bases. This means that our data is very granular, given there could be tens of thousands of rides within one hour. Thus, in order to effectively analyze our data, we will need to group by hour in many cases, meaning we could be missing important patterns that occur within one hour. High granularity does also have benefits, since it means that although we narrowed our frame to only one week, we still have many data points to work with.

There was selection bias in selecting our one week of April subset of the entire 2022 year’s ride-share dataset. We can’t account for proper randomization of our data and the sample obtained may not be a complete representation of our entire dataset, but because our dataset was too big, we had to sacrifice selection bias. Measurement error could also be relevant in our data as the website states “we cannot guarantee or confirm their accuracy or completeness”. 

Some additional features we wished we had that would improve the specificity of our data include the following: 
A measure for traffic: Traffic can affect the expected vs actual ETA which could then affect the driver’s pay and reviews. 
Number of stop lights: Similar to a measure of traffic, stop lights could affect the estimated drive time and the expected base pay and driver pay. 
Whether or not there was surge pricing: Popular areas and times of day affect pricing when there is an increased demand and limited supply for Ubers. This information could have been useful in estimating how certain geographic locations were affected surge pricing throughout the day. 
Expected length (minutes) of trip: The expected minutes for the trip may be useful in predicting driver pay because the driver pay likely doesn’t account for traffic that shows up in the middle of a ride.

## Research Questions
Our research questions that we seek to answer in our analysis are below:

- How does ride-share trip minutes have an effect on Uber driver pay? (Causal Inference)

- How do factors such as trip miles, trip minutes, and precipitation predict driver pay? (GLMs & Nonparametric Methods)

The first research question has the potential to give Uber driver’s more pay autonomy. Recently, Uber unveiled a new feature that allows drivers to choose the trips they want to, so by understanding the correlation between trip minutes and driver pay, Uber drivers can have the autonomy to choose durations and types of trips to maximize their earnings. Our second question also gives Uber drivers greater pay autonomy. If certain lengths and durations of trips and types of weather are associated with a greater pay, drivers can also have the autonomy to choose certain days to provide rides based on the weather forecast or types of trips depending on the weather forecast. 

We used Causal Inference to answer our first question of the effects of ride-share trip minutes on Uber driver pay, because we wanted to see the extent to which driver pay is influenced by trip minutes. We knew our dataset had confounding variables (trip_miles and weekend), so we took those into account by including them in our model. We used GLMs & Nonparametric Methods to answer how factors such as trip miles, trip minutes, and precipitation predict driver pay. Using GLMs helped us see the linear relationship between the response and predictors without assuming the underlying relationship is linear. We also had a lot of data with little prior knowledge about the distributions, so we used Nonparametric Methods to compare with our GLMs results.

## EDA

<img width="567" alt="Screen Shot 2023-07-01 at 10 35 22 PM" src="https://github.com/serenabai/NYC-Ubers/assets/78036684/376a3c5f-6daa-4197-bad1-82f338a7d141">

First we plotted the distribution of driver pay, which is important to understand as we explore the effect of other variables on this variable. We noticed that the distribution peaks close to zero, which makes sense since the majority of Uber rides within New York likely tend to be shorter. However, there seem to be some very large outliers, as the distribution extends to almost 1750. This is important to consider as we continue our analysis of the data. 


<img width="562" alt="Screen Shot 2023-07-01 at 10 29 15 PM" src="https://github.com/serenabai/NYC-Ubers/assets/78036684/b489b79b-c8ee-416f-8532-66ed0e63afe9">

Through the pairplot, we can see that there is a positive linear correlation between trip minutes and driver pay and trip miles and driver pay. We are interested in finding the coefficients of these relationships, so we can see just how much trip miles and trip minutes affect driver pay.  In addition, we found that the distributions of each of these variables seem approximately normal with a right skew, which will motivate how we build our Bayesian GLM model.

<img width="633" alt="Screen Shot 2023-07-01 at 10 29 23 PM" src="https://github.com/serenabai/NYC-Ubers/assets/78036684/9a390cc7-f976-4cd2-9a95-0c4b4e56a7a5">

<img width="410" alt="Screen Shot 2023-07-01 at 10 29 07 PM" src="https://github.com/serenabai/NYC-Ubers/assets/78036684/ab3c5c5c-5c16-42a7-b933-37979b4de9b0">

We also saw that there is a difference in the trajectory of driver pay throughout the day depending on the day of the week. By comparing the graphs of the cumulative mean pay per hour vs the graph separated by weekday, we can see that there are certain patterns that become lost when only considering the cumulative mean pay. To explore this further, we plotted histograms of driver pay on weekends and weekdays, and trip minutes on weekends and weekdays. Although the graph does not show a significant difference, our intuition says that whether the ride takes place on a weekend could be a confounder between driver pay and trip miles, so we want to explore this further in our analysis. 

<img width="420" alt="Screen Shot 2023-07-01 at 10 34 05 PM" src="https://github.com/serenabai/NYC-Ubers/assets/78036684/6f7b061b-7c06-44a8-95dd-f718d45961b0">

<img width="454" alt="Screen Shot 2023-07-01 at 10 34 09 PM" src="https://github.com/serenabai/NYC-Ubers/assets/78036684/6bf13a9d-b7e8-4a4b-a2d6-0179d2f2baa1">

Finally, we plotted the relationships of temperature and precipitation vs driver pay to see how weather affects our variable of interest. These scatterplots show a positive relationship between both temperature and driver pay and precipitation and driver pay, which we will investigate further in our analysis portion. 

The data cleaning steps we took were: 
- Included only Uber (over 50 percent of rides) out of taxi cabs, Uber, and Lyft
  - Only including makes our question less generalizable because there could be inherent differences between different rideshare services
- Changed the date time to 24 hour time and ensured that we accounted for the timezone when using the datetime module
  - Makes our findings more accurate
- Used only the first week of April 2022 (April 3 to April 9)
  - Limiting the dataset means we have less data to back up our findings. Also means our results could be confounded by things that only affected that week in particular.
  - Limiting our data was necessary to conducting our analysis since the unedited dataset was much too large to work with efficiently
- Adding a variable “weekend” which describes whether a ride took place on the weekend or a weekday
  - Allowed us to see whether there is a difference between weekends and weekdays
- Combined the Uber dataset with the weather dataset
  - Allowed us to perform analysis on driver pay based on the weather that day
- For our GLM model, filtered for only rides where driver pay was greater than zero
  - Necessary in order to run the GLM regression without linear algebra errors

Our visualizations are relevant for the following reasons:
Distribution of driver pay
- Since this is our dependent variable, we should understand the shape of its distribution before proceeding with our analysis, therefore it is relevant to both questions.

Trip miles vs trip minutes vs driver pay
- By plotting trip minutes and trip miles against driver pay, we can see that there is indeed a positive correlation between these variables and driver pay. This suggests that there could be a causal relationship, which suggests a solution to our first research question — that there is a causal relationship. This also motivates our second question, suggesting that trip miles and trip minutes should be included in our GLM and nonparametric models.

Driver pay vs hour (cumulative and separated by day of week)
- Plotting driver pay on different days of the week, as well as on weekends vs. weekdays showed that the trajectory of pay over the course of the day changes based on the day of the week. This shows a potential confounder, which we would need to take into account for our first research question.

Histogram of driver pay on weekends vs weekdays/Histogram of trip minutes on weekends vs weekdays
- These graphs showed that there is a slight difference between weekdays and weekends in terms of driver pay and trip minutes. This would also be evidence for a potential confounder in our causal inference model. However, since the difference is not obvious, we should explore this further.

Scatter plot of temperature vs driver pay/Scatterplot of precipitation vs driver pay
- The scatterplots of temperature and precipitation showed slight positive correlations with driver pay, so we should include it in our GLMs/random forest model, making it relevant to our second research question. 
## Causal Inference
### Set-Up
- Research Question: How does ride-share trip minutes have an effect on driver pay? 
- Treatment: Driving more ride-share trip minutes
- Control: Ride-share trip 
- Units: Rides
- Technique Used: OLS Regression



We assumed that the trip being on a weekday or weekend would affect trip_minutes, as weekdays have surges in traffic, particularly during rush hours, which would affect how long a trip would take. Additionally, a trip being on a weekday or weekend would also directly affect driver_pay, as there are different supply and demands on the weekday vs. weekend. Another assumption we made about trip_miles affects trip_minutes and driver_pay directly, as this is something that is provided by Uber and Lyft websites in determining how much a driver gets paid. The trip_miles determine the base_passenger_fare, which is accounted for in the overall driver_pay.
### Methods
Our causal inference methods include the relevant variables of weekend, trip_miles, trip_minutes, and driver_pay. The trip_minutes is the treatment variable while driver_pay is our outcome. The two confounding variables trip_miles and weekend (weekend being Friday, Saturday, Sunday and weekday being Monday, Tuesday, Wednesday, Thursday). Both of these confounding variables affect the treatment and the outcome but are not personally the treatment or the outcome, so the unconfoundedness assumption does not hold in this instance. The methods we used to adjust for confounders were to include both confounders, trip_miles and weekend, in our regression to account for their effect on the outcome. There are no colliders in our data set. 
### Results
Based on our OLS regression results, trip_minutes has a positive causal effect and increases driver_pay for Uber trips by 27 cents. The estimated causal effect on driver_pay by our confounding variable weekend (Friday-Sunday) is around 11 cents. Our other confounding variable trip_miles increases driver_pay by 53 cents. The causal effect on driver_pay on base_passenger_fare is 46 cents. We reject the null hypothesis, since our p-values are approximately all zero, and zero does not lie in any of the variables’ confidence intervals. One uncertainty in our estimate comes from not knowing for sure that our confounding variables weekend and trip_miles are the only confounders. Furthermore, because our dataset is very large, we could only use data for the first full week of April 2022 (April 3rd-9th), so there could also be uncertainties that come from not using a whole year’s data.



### Discussion
Our results show that for each minute increase in the Uber trip minute, a driver will get paid on average 27 cents more. This is after accounting for confounding variables of the trip being on a weekend of not, trip miles, and base passenger fare. Additionally, the ride occurring on a weekend (Friday, Saturday, or Sunday) will increase the driver pay by 11 cents on average per ride. Thus, we are able to say that there is some association between trip minutes and how much a driver gets paid, although this is quite intuitive in itself.

We were limited by the variables available in our datasets for our causal inference, so we couldn’t include all confounding variables in our model. If we had a more extensive dataset we could have made a more accurate interpretation of driver_pay. In addition, if we had better computing power we could have used our entire dataset, which included the entire year of 2022. Having a more robust dataset would give us better predictions and reduce the effects of specific factors that could have only occurred in our subset of a single week of April.

Although the variables we used for our causal inference model all have an expected positive causal effect on driver_pay, there is additional data that would be useful in predicting driver_pay. Another dataset we could use would be rider demand data, which would directly impact driver_pay as Uber makes use of surge pricing during times of peak demand. The type of road each driver took could also have an impact on trip_minute and act as an instrumental variable to determine driver_pay. For example, even if trip_miles was long, if the driver took the highway instead of a city street, trip_minutes would decrease significantly causing driver_pay to decrease.

From our research, we expected positive coefficients from all of the variables in our model as we believed that each variable has a positive causal effect on driver_pay. Our OLS regression results matched our expectations, giving us confidence that there’s a causal relationship between our chosen treatment and outcome. Furthermore, all our variables’ p-values are approximately zero, and zero does not lie in any of the variables’ confidence intervals, which strengthens the idea that there is a causal relationship. However, we are not as confident about the true coefficients of our variables’ effects on driver_pay as we did not consider the entire dataset of 2022.

## GLMs & Nonparametric Methods
### Methods
The analysis below will focus on the following prediction problem: predicting driver pay from weather, precipitation, trip time, and trip distance. The research question of interest here is: How do factors such as trip miles, trip minutes, and precipitation predict how much a driver will be paid? The features we will be using are `trip_miles`, `trip_minutes`, `precipitation`, and `hour`. These are valid features to be using in our model because each ride (row) in our dataset is associated with all of these features. Additionally, we see in our EDA that there is a slight correlation between each of these features and the `driver_pay` that we are trying to predict. 

GLMs — The GLM that we initially chose was the Gaussian GLM. We chose this since we assumed that driver pay is a positive continuous variable, and saw from our EDA that the distribution of driver pay seemed approximately normal. Since we didn’t have any strong underlying assumptions about the data, we chose not to use priors in the Bayesian GLM. We will be judging the performance of the GLMs by plotting and qualitatively observing the posterior predictive distributions, and calculating each coefficient’s credible intervals. We selected the Bayesian GLM model as it can flexibly integrate various probability models and parameter uncertainty with reference to the multiple driver pay parameters measured in this study. We evaluated its performance by plotting the posterior predictive distribution visualized for each of the predictive variables. 

Nonparametric Methods — The nonparametric methods we will be using are Decision Trees and Random Forests. The assumptions we made in choosing these are models are as follows
Our input data (variables of `trip_miles`, `trip_minutes`, and `precipitation`) are continuous.
There are no missing values in the input data (utilized drop_na).
For this analysis in particular, a Random Forest is preferred as our data is quite noisy, and Random Forests predict better than Decision Trees do with noisy data. For the Random Forest, we will be predicting with a maximum of 1 feature per tree in the algorithm, based on our maximum of K/3, where K here is 4.

The non-parametrics will be trained on a training dataset that is made up of 70% of our dataset, and its performance will be evaluated on the test data using MAE and RMSE. The MAE will treat all errors equally, whereas the RMSE will penalize larger errors more.
### Results

#### Bayesian Approach

While we initially chose to use the Gaussian GLM, using trip_miles, trip_minutes, and precipitation as predictors. However, plotting the predictive posterior distribution showed that it was not a great fit for our data, as the credible intervals were much too narrow. Reasoning that because the driver_pay data is very dispersed, we also tried to apply the Poisson and Negative Binomial GLMs. Pictured below are the posterior predictive distributions for each GLM, visualized against each of the predictive variables.


Based on these visualizations, it seems that the Gaussian model, though not perfect, is the best option, as its 95% confidence interval fits the data most closely for trip_minutes and trip_miles. We also see that none of the PPDs for predicting driver pay using precipitation seem suited to the data, suggesting that we should not include precipitation in our model. Therefore, we run the model again using only trip minutes and trip miles, getting the following posterior results. 


As we can see in these results, using the Gaussian model with variables trip_miles and trip_minutes gave coefficients of approximately 2.70 for trip_miles and 1.50 for trip_minutes. Since the Gaussian model uses the identity link function, this means that driver_pay is predicted to increase by 2.7 for every increase in trip_miles and by 1.5 for every increase in trip minutes. However, we also see that the spread for each coefficient is very low, much lower than the spread of the actual data. Therefore, we can conclude that the Gaussian GLM is not the most optimal way to predict driver_pay from trip_miles and trip_minutes.

#### Frequentist Approach
We used the Gaussian, Poisson, and Tweedie Models of Frequentist GLM to model the distribution of our drive pay data. We used the Tweedie distribution to account for the frequency of values close to zero and more spread out further away from zero. 

In the frequentist model, the parameter we’re estimating is a fixed quantity. Our estimate, however, is random because it depends on the data, which is random. In the frequentist approach, the goal is to find the best - fitted distribution of the random estimate. 

The log-likelihood measures how well a parameter theta explains the observed data. Out of the three models, the Poisson model had the highest log-likelihood meaning that out of the three models, the Poisson GLM model is the best fitted for this data set. 

The chi square value measures the difference between observed and expected frequencies between outcomes of a set of events or variables which can also be approximated by the number of observations subtracted by the number of parameters. Here, all the models’ chi square value is far above the chi-square statistic, meaning that there is too much variance and that all the models are not an ideal fit for the data. 

Gaussian Model 

Under the Gaussian model, we see that trip_miles increases driver pay by 1.4915, trip_miles increases driver pay by 0.4780, precipitation increases driver pay by 0.7983, and hour increases driver pay by 0.0572. The confidence intervals all greater than 0 and the large z score values indicate that the confidence interval is small but still statistically significant. 

Poisson Model 

Under the Poisson model, we see that trip_miles increases driver pay by 0.0009, trip_miles increases driver pay by 0.0174, precipitation increases driver pay by 0.0304, and hour increases driver pay by 00012. Here, the z-score values are still quite large, but particular values, such as a z-score of 59.453 for hour indicate a greater confidence for that coefficient. 

Tweedie Model 

Under the Tweedie model, we see that trip_miles increases driver pay by 0.0009, trip_miles increases driver pay by 0.0174, precipitation increases driver pay by 0.0304, and hour increases driver pay by 00012. These are the same coefficient values we observed in the Poisson distribution except here, there is even greater variation in z-score values. This model features the smallest z-score value from our analysis of 27.790 for hour which indicates a higher confidence interval. 

### Nonparametric Results
The performance of the non-parametrics results are shown in the table below:

Decision Tree
Random Forest
Training Set MAE
-2.72e-17
-2.75e-17
Test Set MAE
0.00121
0.0013
Training Set RMSE
5.40025
5.5936
Test Set RMSE
7.9454
7.3594


Based on the MAE, we see that the Decision Tree predicted `driver_pay` slightly better than the Random Forest did on the test set, which is interesting, since the Random Forest usually predicts better than Decision Trees do. However, using RMSE to measure performance, we see that the Random Forest predicted better on the test set, with an error of 7.36 compared to 7.95 from the Decision Tree. 

Though the error values from the MAE and RMSE are both relatively small, they are not necessarily indicative of “accurate” prediction from the features of interest (`trip_miles`, `trip_minutes`, `precipitation`, and `hour`). The MAEs are extremely close to 0, which is pretty good considering that `driver_pay` in our dataset ranges from 0 to 1723. However, looking at the RMSEs of our test sets, 7.9 and 7.4 are not necessarily large numbers, but they are over 3 times the size of the standard deviation of our `driver_pay` data points, which is approximately 1.94. This indicates that the predictions in our Decision Tree and Random Forests definitely do differ quite a bit from its actual value.
### Discussion
The model that performed the best was the nonparametric model. However, we are not confident in applying this model to future datasets of similar caliber. Specifically for rideshare data, none of the models we used to model ride-share driver pay is particularly strong. With the Bayesian GLMs, we determined that the Gaussian GLM was the most appropriate model for predicting driver pay from trip miles and trip minutes. Of the Frequentist GLMs, the Poisson model performed the best, which we determined from its high log-likelihood value. Though we determined the “best-performing” models, it does not necessarily mean these models are suitable for answering our research question of interest. The nature of rideshare driver pay creates overdispersion in the data, especially since driver pay is determined by a number of factors beyond the ones we modeled above. 

For the Bayesian implementation of the GLM, the Gaussian model fit the data the best out of all of the models that we tried. However, although the shape of the posterior seems to fit the data fairly well, the spread is much too small, showing that we had overdispersion in our data. This happened because our driver_pay data has a very high variance, which is not accounted for by the Gaussian model. Thus, the model ends up being overconfident that its narrow range is correct. Our results for the Bayesian and Frequentist implementations of the GLM were very different. While for the Bayesian implementation, we found that the Gaussian model was the best fit, the Frequentist method actually found the Poisson model to be the best.

We were limited in our Bayesian implementation of the GLM because we did not have strong underlying assumptions about the data. Therefore, we could not take full advantage of the main benefit of using this model, which is the ability to include priors. In addition, given our limited knowledge and experience working with pymc3, we were unable to find a GLM that could fit the data better than the ones we had explored in class. For example, we researched the Gamma model, and found that it might have fit our data better than the Gaussian model, but were unable to successfully implement it using pymc3. The limitations of the frequentist approach relies on volume of observation and the reliability of results are only confirmed at the end. The frequentist GLM approach may be better fitted if we sampled larger than the first week in April 2022. 

Another limitation is that we used Decision Tree and Random Forest regressors and not classifiers in our prediction. Our results may have been more telling if we created classes for our dependent variable of `driver_pay` ($0-5, 5-10, etc.), and ran classifiers instead. This way, we would not be necessarily predicting the exact value of how much a driver gets paid, rather the range of what they would get paid for each ride. 

Additional data that might be useful is if we had any priors for the distributions of trip_miles, trip_minutes, and precipitation. Additionally, similar to the causal inference above, traffic data and price surge in Uber rides would be useful data, and probably much more useful in predicting Uber driver pay.

## Conclusions
To answer our initial research questions below:

How does ride-share trip minutes have an effect on Uber driver pay?
We find that for each minute increase in Uber ride minutes, a driver’s pay will increase on average by 27 cents, while accounting for controls such as day of week (weekend/weekday), ride miles, and base passenger fare

How do factors such as trip miles, trip minutes, and precipitation predict how much a driver will be tipped?
From the variables of trip miles, trip minutes, and precipitation , we find that trip miles and trip minutes do the best at predicting how much a driver is paid, whereas precipitation is not that telling of a predictor variable. Essentially, precipitation does not necessarily affect how much a driver is paid all that much on its own, but could be useful in predicting driver pay when used in combination with trip miles and trip minutes.

As we were forced to narrow down our dataset in order to perform our analysis more efficiently, our findings are quite narrow. We only used data about one week in April in New York, for Uber rides. Therefore, our findings are quite narrow as they only account for 1/52 of the weeks in a year for one year. Our results could be generalized for perhaps the rest of the month of April and perhaps for driver pay across Aprils across multiple years. Besides that, it would be difficult to generalize the data, especially the weather data, for other months in the year. 

Currently, ride-share companies such as Uber & Lyft auto-generate a driver’s next ride through an algorithm, and the driver gets very little autonomy over which ride is chosen for them and therefore how much they can potentially be compensated for. 

Our findings have the potential to provide more work-hour & compensation to Uber drivers if integrated in some sort of analytics dashboard for the driver and if ride-share companies allow drivers to have some choice over the rides they choose to accept (only recently has Uber allowed drivers to do this). This way, driver’s can have more autonomy over their work hours, whether they work on weekends and weekdays, and whether they will work in a certain type of weather condition to optimize their driver pay.  

We merged TLC data and weather data sourced from the open-meteo API. The benefits of this was that we were able to incorporate another dimension of data into our predictions, using weather data to predict driver pay. There are not any noticeable consequences of combining different sources beyond the fact that the weather data was not useful in predicting driver pay.

Some limitations in our data include the fact that we only used data from one week due to the overwhelming volume of data in our original set. Performing the same analysis on data from different months could reveal meaningful insights about how driver pay could differ depending on the time of the year. In addition, we only used less than 1% of this already narrow frame to build our GLMs. In the future, it may be meaningful to use a larger portion of our data and see if the results are the same. To continue our exploration, we could test our current GLMs and nonparametric model on newer test data to see how generalizable our models are. We also did not use priors for our Bayesian GLMs since we did not have any meaningful assumptions. In the future, bringing in additional domain knowledge to build those priors could be beneficial to building a more accurate model. Finally, since we narrowed our dataset to only Uber rides in New York, future studies could explore similar research questions relating to other rideshare options such as Lyft, or in other major cities in the US to see if there are any meaningful differences. 

