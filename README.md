# Overview <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/a/a0/Lyft_logo.svg/220px-Lyft_logo.svg.png" align="right" height="45">



_Created by Bernard Wong (bew030@ucsd.edu) and Jonathan Gong (jgong11@illinois.edu)_

We were recently prompted with the task of creating a business report for Lyft's Data Challenge. With the given datasets involving driver, ride, and time, we were asked to answer a few questions: 
1. What's a Driver's Lifetime Value (i.e., the value of a driver to Lyft over the entire projected lifetime of a driver)
2.What are the main factors that affect a driver's lifetime value?
3. What is the average projected lifetime of a driver? 
4. Do all drivers act alike? Are there specific segments of drivers that generate more value for Lyft than the average driver?
5. What actionable recommendations are there for the business? 

The following repository contains our [notebooks](https://github.com/bew030/lyft-challenge/tree/master/work%20notebooks) with the code used to clean, analyze, and [visualize the data](https://github.com/bew030/lyft-challenge/tree/master/docs), and the [written reports](https://github.com/bew030/lyft-challenge/tree/master/writings) that show our findings. Our [final submitted report](https://github.com/bew030/lyft-challenge/blob/master/writings/submitted_lyft_paper.pdf) contains our thought process as well as our results and answers to the questions. You can also find our results of our week long exploration in the rest of this README. 

Out of respect to Lyft and their data scientists we've gone ahead and deleted the datasets to keep the data safe. We'd like to thank Lyft for providing us with this amazing opportunity and hope that you find our findings interesting. If you have any questions or would like to learn more, please feel free to reach out by email or leave an issue.  


# Installation 
To ensure that the Jupyter Notebooks run properly, use the package manager [pip](https://pip.pypa.io/en/stable/) to install requirements.txt. The packages that we used were Numpy, Matplotlib, Pandas, and SciPy.

```bash
pip install requirements.txt
```

# The Report
**Introduction**

The ride sharing industry has increased significantly in the past few years. With the [Ride Sharing Market predicted to reach 220.5 billion USD by 2025](https://www.reuters.com/brandfeatures/venture-capital/article?id=83120) and company goals of worldwide expansion,  it’s undeniable that companies such as Lyft are playing a dominant role in the way that society thinks of transportation. With our analysis on Lyft drivers and the business as a whole, we hope that we’ll be able to provide valuable insight on what the next best steps are for Lyft so that they can successfully increase business while still providing high quality ride sharing. 

**Understanding and Cleaning the Data** 

We were given 3 different datasets; one giving us the information about drivers (driver_ids), one giving us information about rides (ride_ids), and one giving us information about the timing with rides (ride_timestamps). Before any work can be done, we needed to make sure we understood the data and that the data was pure and wasn’t missing or erroneous.

_Drivers Dataset_

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/1.png" width=300/>
</p>

The Drivers dataset is relatively simple and clean. The dataset contains driver IDs and when each driver joined Lyft. Overall, the onboard dates were realistic and there were no repeats of driver IDs, making this dataset ready for use.

_Rides Dataset_ 

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/2.png"/>
</p>
The Ride dataset contains driver IDs, ride IDs, ride distances in meters, ride duration in seconds, and the prime time percentage. Unlike the Drivers Dataset, the Rides Dataset has repeats of Driver ID. However, each ride ID is unique which means that we have distinct information for a variety of different Lyft rides. 

Because we knew we wanted to do an analysis regarding different types of costs, we decided to create extra data about the cost to the rider with and without prime time as well as calculate the profit that Lyft would get with each ride. Because our mileage rates were in miles and our duration rates were in minutes, we first converted the distance and duration to the proper units. Afterwards, we created formulas for the cost of the ride for the rider including prime time and San Francisco tax and the profit Lyft made per ride assuming that they take 20% of the cost of the ride (according to [Money Under 30](https://www.moneyunder30.com/driving-for-uber-or-lyft)). Here were the formulas that we created:

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/3.png"/>
</p>
  
As a result, we now had a dataframe that also included information about the cost to the rider as well as the profits that Lyft makes for each ride. 

_Time Dataset_

The time dataset contained information about ride IDs, types of events (which include when the ride was requested by the rider, when a driver accepted the request, when the driver arrived at the requested location, when the rider entered the driver’s vehicle, and when the driver dropped off the rider), and the timestamp of each event occurring. We noticed that each ride ID repeated five times and instead had unique timestamps for the five different events that occurred in each ride. As a result, rather than have a ride repeat five times, we decided to transform and pivot the data frame to make it more organized.

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/4.png"/>
</p>

Along with that, we created some additional columns that had the measurements between each of the events. This information tells us how long the rider has to wait to get a ride, how long the driver has to wait to pick up the rider, and how long the trip is overall.

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/5.png"/>
</p>

While double checking the data for validity and missingness, we noticed that the amount of time the driver had to wait to pick up a rider was incorrect for some of the rides. Some of the measurements were negative, meaning that the driver had picked up the rider before arriving at the location. Because there was a reasonably sized portion of rides with negative pick up times, we decided to replace the times with a placeholder (np.NaN) rather than get rid of the data. We were cognisant of this erroneous data and as a result we didn’t do too much data manipulation regarding that column. However, the other columns had relatively reasonable data points and no missing data or negative times, so as a result we feel that this data can still prove to be valuable. After rotating and cleaning, the dataset is now ready for use. 

**Combining the Dataframes**

Because all three datasets had matching driver or ride IDs, we merged all three using the ride data frame as a middle ground. While merging the datasets together we noticed that not all driver IDs from the driver dataset matched the driver IDs from the ride dataset, as well as not all the ride IDs from the time dataset matching the ride IDs from the ride dataset. We dropped any datapoints that were not fully matching because we had plenty of data to work with (184209 data points or 95% of the ride data). Along with that, data imputation for the driver or time dataset wouldn’t have been very valuable because we weren't sure whether the drivers drove no rides or simply were not measured in the rides dataset. Because the missing time data only accounted for 1% anyways, data imputation wouldn't have been wise. In the end, we were able to merge everything and create a master dataset of 184209 points that contains information about the ride, the driver, and the times.

**Analysis and Results** 

_WHAT IS THE AVERAGE PROJECTED LIFETIME OF A DRIVER?_

Our final merged data frame combines both information about driver and times which means we can start to predict average projected lifetime of a driver. Before we could even begin to calculate averages though,  we needed to determine what the lifetime of each Lyft driver in our dataset was. We determined that the number of days between the driver’s onboard date and the driver’s last drive would represent the lifetime of a driver. As a result, we were able to manipulate the data frame and come up with a lifetime expectancy for each of the drivers in our dataset. 

The first item that we created was a boxplot of driver lifetimes. This boxplot helps us easily identify any outliers that lie outside the 1.5 IQR range. Because there are no data points outside the bounds we see that there are no outliers in driver lifetimes, which is a great sign for the data. 
<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/6.png"/>
</p>

We also calculated the skewness and kurtosis of the lifetimes, which we found to be -0.48 and -0.56 respectively. The skewness tells us that the data is slightly skewed to the left, but because the absolute value of the skewness is still less than .5 we can conclude that the data is relatively normal. The kurtosis value tells us that there’s a low chance for outliers, which reinforces the results we got from the boxplot. 

Plotting out the density histogram of lifetime confirms our previous results. We see that the lifetimes are relatively normal meaning that there shouldn’t be too big of a difference between the mean and median being a representation of the average lifetime. Because there is still a very slight skewness though, our conclusion is that the median will most likely be a better representation of the average projected lifetime, which means that once a driver is onboarded, we predict that ___they will typically continue driving for Lyft for 55 days.___ 
<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/7.png" width=500/>
</p>

_WHAT IS A DRIVER'S LIFETIME VALUE, OR THE VALUE A DRIVER BRINGS TO LYFT OVER THE PROJECTED LIFETIME?_
<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/8.png" width=1000/>
</p>

There are a few factors that we believe make a driver valuable to Lyft which include profitability, quantity of business, and quality of business. Profitability is the most obvious and measurable amount and is the amount of profit Lyft makes (in USD). However, profit is not the only thing that makes a company successful; quantity of quality of their business also strongly determines its success. As a result, we decided to also measure the number of rides (which is a measure of the quantity of business that Lyft does) and the speed of service (which is a measure of the quality of business and can be correlated to customer satisfaction) that a rider provides in a projected lifetime. 

We went ahead and plotted Driver Lifetime vs. Lyft profit, Driver Lifetime vs. Number of Rides, and Driver Lifetime vs. Total Amount of Time a Customer Waits in order to find some trends. Because we wanted to predict these values as well, we plotted a few lines of best fit to help us with our predictions.  For each one of these factors, we utilized machine learning and linear regression algorithms to find the line of best fit. 

Our process was simple. For each of these factors, we created 200 70/30 testing/training datasets and then used them to create polynomial lines of best fit from the 1st degree to the 13th degree. We then took the average r<sup>2</sup> for each of these polynomial lines and selected the polynomial line that had to largest r<sup>2</sup> to use on our data. By dividing up the data into testing/training data, we avoid overfitting or underfitting the data. 

Using these methods, we were able to find the lines of best fit for each of these factors. After plugging in the expected driver lifetime of 55 days to each of the respective lines of best fit, we were able to find that ___the average driver brings in around $637.04 in profits, completes around 267 rides, and on average makes a potential rider wait around 235.45 seconds for a ride for their projected lifetime of 55 days.___

_WHAT ARE THE MAIN FACTORS THAT AFFECT A DRIVER'S LIFETIME VALUE?_

We defined our Driver’s Lifetime Value based off of three factors: the pure capital they bring to Lyft, the amount of business that they bring (in number of rides), and customer satisfaction (inadvertently based off of the low wait time). We hypothesized that the primary factors that would impact these values would be the number of rides a driver completes in his/her lifetime, a driver’s average ride duration, a driver’s average ride distance, the average time it takes for the driver to accept a rider’s request, the average time it takes for the driver to arrive at their rider’s location after accepting the request, and the average time it takes for the driver to successfully pick up the rider.

* Pure Capital 

The factor we found most impactful on the sheer capital that a driver brings in was ___the number of rides that a driver completes___. We plotted both drivers’ lifetime number of rides and average number of rides per day versus their value to Lyft (Number of Drives vs. Lyft’s Profit (Dollars), Average Number of Rides/Day vs. Lyft’s Profit (Dollars)).

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/9.png" width=1000/>
</p>

Both relations show clear positive linear correlations. This intuitively makes sense; the more rides a driver completes, the more money the driver makes and the more revenue is generated for Lyft. In addition to this, it seems there’s a slight negative correlation between the driver’s arrival time after accepting a request versus profit. There are many possible explanations for this. One possible one is that drivers who are quick to arrive after accepting a request tend to me more efficient and industrious, and therefore generate more profit.

There is also a slight positive correlation between the time it takes a driver to pick up their rider after arriving versus profit. Again, there are many possible explanations for this. A possible reason is that drivers who wait for less time are perhaps happier with their jobs and are more likely to continue doing more rides and not quitting.

* Business

Business, which is closely related to the company’s publicity, is another form of value that a driver can bring. The quantity of business that a driver presents can be naively represented by the number of rides that a driver performs in his/her lifetime; more rides mean more exposure for Lyft.

We compared the times that it took drivers to accept drives after they had been requested versus the lifetime number of drives those drivers performed as well as the times that it took drivers to arrive at their riders’ locations after accepting the requests versus the drivers’ lifetime drives. Both showed slightly negative correlations, seeming to imply that ___drivers who experience less downtime before their drives tend to be able to complete more drives over their lifetimes and bring more business to the company.___

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/10.png" width=1000/>
</p>

* Customer Satisfaction

Customer satisfaction is the third and final factor that goes into the value that a driver can bring Lyft. If a driver gives a rider a good experience, the rider is more likely to do another ride with Lyft again in the future, bringing in more potential revenue.

We naively represented customer satisfaction as the time difference between a rider requesting a ride and the driver picking the rider up. There’s many other factors that can potentially impact customer satisfaction, such as the quality of the driver’s car/driving, however we don’t have access to data describing these factors. The aforementioned time difference tends to be a large factor that customers consider when using ride sharing services like Lyft; riders don’t like having to wait significant amounts of time for their ride to arrive. It should be an accurate gauge of customer satisfaction, at least to some degree.

We found a negative correlation between a driver’s average number of rides/day and customer satisfaction. There are numerous reasons why this might be the case. It’s possible that as drivers do more rides throughout the day, they begin to treat successive rides with less thought and care than they might if they did fewer rides per day. We also found a positive correlation between a driver’s average ride duration and customer satisfaction. Perhaps drivers who conduct longer rides tend to maintain better driver-rider relationships due to increased interaction time.

Both correlations show relatively small statistical significance; points are scattered in the plot like a cloud. Thus, only the correlations’ positivity or negativity were considered and any deductions discussed should be taken with a grain of salt. ___Overall, however, the data suggests that drivers who give more and longer rides tend to garner greater customer satisfaction.___

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/11.png" width=1000/>  
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/12.png" width=1000/>
</p>

_DO ALL DRIVERS DRIVE ALIKE?_

On many of the plots that we’ve obtained, there was a distinct separation of drivers into two groups. This difference could be observed most clearly when plotting the number of drives a driver undertook versus various other factors, such as average ride duration and profitability (Average Ride Duration (Seconds) vs. Number of Drives, Number of Drives vs. Lyft’s Profit (Dollars)). Drivers that had either greater than or less than 100 lifetime rides tended to group together.

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/13.png" width=1000/>
</p>

A likely explanation of this separation may involve the learning curve associated with becoming a more experienced driver. Less experienced beginner drivers tend to behave similarly to one another, as do more experienced drivers. In terms of performance and profitability, the two groups have very different tendencies. Across the board, beginner drivers underperform compared to the average profitability.

For sheer capital profitability, experienced drivers show a strong positive relationship between the number of rides that they perform per day versus their profitability. Less experienced drivers, however, show little correlation at all between the two factors; their below average profitability remains unaffected by the number of rides they perform per day (Average Number of Rides/Day vs. Lyft’s Profit (Dollars)). Other factors, such as the time between a request accepts and driver arrival (Average Time Between Accepted and Arrived (Seconds) vs. Lyft’s Profit (Dollars)) or time between driver arrival and rider pickup (Average Time Between Arrived and Picked Up (Seconds) vs. Lyft’s Profit (Dollars)), don’t show differences in the direction of the correlation however still illustrate the fact that beginner drivers consistently perform below average.

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/14.png" width=1000/>
</p>

Quality of business is the final measure of profitability. Comparing the two groups in Average Ride Duration (Seconds) vs. Time Between Drive Requested and Picked Up (Seconds) and Average Number of Rides/Day vs. Time Between Drive Requested and Picked Up (Seconds), it can be seen that the positivity of the correlations remain consistent between the two groups however, once again, beginner drivers consistently tend to take longer to pick up their riders than the more experienced drivers.

<p align="center">
	<img src="https://github.com/bew030/lyft-challenge/blob/master/docs/ss/15.png" width=1000/>
</p>

___For all notable measures of profitability, experienced drivers generate more value for Lyft.___ This should make sense; drivers who take their roles seriously and commit to more rides tend to be the ones who make more money. ___Action should be taken to maximize the number of drivers that fall into this category.___

_What actionable recommendations are there for the business?_

As we discussed in the previous question, there seems to be a large disparity between beginner drivers who have done less than 100 drives  and more experienced drivers. There are a few potential explanations for this disparity. One reason could be due to the fact that beginners are not getting paid enough to have the desire to continue. Less experienced drivers are consistently making less profits for Lyft (and therefore making less money themselves) and consistently have a smaller lifetime (meaning that they’re quitting early). 

Our recommendation for the company is to focus on this issue and perhaps create incentives that promote being a long time driver for Lyft. By incentivizing drivers to drive more than 100 rides, we believe that Lyft will be able to maintain a long term relationship that will be beneficial both to Lyft and the driver. By either offering a bonus after hitting 100 drives or potentially adding a ‘prime time rate’ for beginners, this could potentially accelerate the activity beginner drivers have and generally increase the average value drivers bring to Lyft. 

**Conclusion**

Overall, we were able to discover many valuable lessons from the data that was measured. However, there are a few points that are worth acknowledging.

The data primarily consisted of rides within a three month period which could greatly affect many things, the largest being the projected life time. We assumed that the ride dataset contained the full careers of drivers but this may not necessarily be true. In the future it would be useful to have information about a broader range of rides so that we can account items such as seasonal changes and general changes in driver behavior. 

We also strongly valued business quality and customer satisfaction, and while we used rider waiting time as a measure of business quality, there are many other measurements that could be used as well. Rider ratings and rider tip amounts would be a great way to measure the quality of drivers and could help us quantify business quality accurately. 

As the ride sharing industry grows, the impact Lyft has on the transportation industry and on drivers will grow with it. We believe that these points that we’ve discovered will not only help Lyft grow as a company but also ensure that the impact that Lyft has on its driver and the transportation industry is a positive one. 


# What's Next? 

We'd like to make some further improvements on coming up with actionable recommendations for the company. We mentioned the disparity between beginners and experienced drivers and a potential bonus to encourage drivers to encourage drivers to continue, but we'd like to incorporate some analytical methods to come up with a concrete amount for the bonus. 

# Badges
[![GitHub issues](https://img.shields.io/github/issues/bew030/lyft-challenge?color=purple)](https://github.com/bew030/lyft-challenge/issues)
[![GitHub forks](https://img.shields.io/github/forks/bew030/lyft-challenge?color=orange)](https://github.com/bew030/lyft-challenge/network)
[![GitHub stars](https://img.shields.io/github/stars/bew030/lyft-challenge)](https://github.com/bew030/lyft-challenge/stargazers)
[![Views](http://hits.dwyl.io/bew030/lyft-challenge.svg)](http://hits.dwyl.io/bew030/lyft-challenge)
