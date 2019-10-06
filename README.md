# Overview 

_Created by Bernard Wong (bew030@ucsd.edu) and Jonathan Gong (jgong11@illinois.edu)_

We were recently prompted with the task of creating a business report for Lyft's Data Challenge. With the given datasets involving driver, ride, and time, we were asked to answer a few questions: 
1. What's a Driver's Lifetime Value (i.e., the value of a driver to Lyft over the entire projected lifetime of a driver)
2.What are the main factors that affect a driver's lifetime value?
3. What is the average projected lifetime of a driver? 
4. Do all drivers act alike? Are there specific segments of drivers that generate more value for Lyft than the average driver?
5. What actionable recommendations are there for the business? 

The following repository contains our [notebooks](https://github.com/bew030/lyft-challenge/tree/master/work%20notebooks) with the code used to clean, analyze, and visualize the data, and the [written reports](https://github.com/bew030/lyft-challenge/tree/master/writings) that show our findings. Our [final submitted report](https://github.com/bew030/lyft-challenge/blob/master/writings/submitted_lyft_paper.pdf) contains our thought process as well as our results and answers to the questions. You can also find our results of our week long exploration in the rest of this README. 

Out of respect to Lyft and their data scientists we've gone ahead and deleted the datasets to keep the data safe. We'd like to thank Lyft for providing us with this amazing opportunity and hope that you find our findings interesting. If you have any questions or would like to learn more, please feel free to reach out by email or leave an issue.  


# Installation 
To ensure that the Jupyter Notebooks run properly, use the package manager [pip](https://pip.pypa.io/en/stable/) to install requirements.txt

```bash
pip install requirements.txt
```

# The Report
**Introduction**

The ride sharing industry has increased significantly in the past few years. With the [Ride Sharing Market predicted to reach 220.5 billion USD by 2025](https://www.reuters.com/brandfeatures/venture-capital/article?id=83120) and company goals of worldwide expansion,  it’s undeniable that companies such as Lyft are playing a dominant role in the way that society thinks of transportation. With our analysis on Lyft drivers and the business as a whole, we hope that we’ll be able to provide valuable insight on what the next best steps are for Lyft so that they can successfully increase business while still providing high quality ride sharing. 

**Understanding and Cleaning the Data** 

We were given 3 different datasets; one giving us the information about drivers (driver_ids), one giving us information about rides (ride_ids), and one giving us information about the timing with rides (ride_timestamps). Before any work can be done, we needed to make sure we understood the data and that the data was pure and wasn’t missing or erroneous.

_Drivers Dataset_

The Drivers dataset is relatively simple and clean. The dataset contains driver IDs and when each driver joined Lyft. Overall, the onboard dates were realistic and there were no repeats of driver IDs, making this dataset ready for use.

_Rides Dataset_ 

The Ride dataset contains driver IDs, ride IDs, ride distances in meters, ride duration in seconds, and the prime time percentage. Unlike the Drivers Dataset, the Rides Dataset has repeats of Driver ID. However, each ride ID is unique which means that we have distinct information for a variety of different Lyft rides. 

Because we knew we wanted to do an analysis regarding different types of costs, we decided to create extra data about the cost to the rider with and without prime time as well as calculate the profit that Lyft would get with each ride. Because our mileage rates were in miles and our duration rates were in minutes, we first converted the distance and duration to the proper units. Afterwards, we created formulas for the cost of the ride for the rider including prime time and San Francisco tax and the profit Lyft made per ride assuming that they take 20% of the cost of the ride (according to [Money Under 30](https://www.moneyunder30.com/driving-for-uber-or-lyft)). Here were the formulas that we created:
  
As a result, we now had a dataframe that also included information about the cost to the rider as well as the profits that Lyft makes for each ride. 

_Time Dataset_

The time dataset contained information about ride IDs, types of events (which include when the ride was requested by the rider, when a driver accepted the request, when the driver arrived at the requested location, when the rider entered the driver’s vehicle, and when the driver dropped off the rider), and the timestamp of each event occurring. We noticed that each ride ID repeated five times and instead had unique timestamps for the five different events that occurred in each ride. As a result, rather than have a ride repeat five times, we decided to transform and pivot the data frame to make it more organized.

Along with that, we created some additional columns that had the measurements between each of the events. This information tells us how long the rider has to wait to get a ride, how long the driver has to wait to pick up the rider, and how long the trip is overall. 

While double checking the data for validity and missingness, we noticed that the amount of time the driver had to wait to pick up a rider was incorrect for some of the rides. Some of the measurements were negative, meaning that the driver had picked up the rider before arriving at the location. Because there was a reasonably sized portion of rides with negative pick up times, we decided to replace the times with a placeholder (np.NaN) rather than get rid of the data. We were cognisant of this erroneous data and as a result we didn’t do too much data manipulation regarding that column. However, the other columns had relatively reasonable data points and no missing data or negative times, so as a result we feel that this data can still prove to be valuable. After rotating and cleaning, the dataset is now ready for use. 

**Combining the Dataframes**

Because all three datasets had matching driver or ride IDs, we merged all three using the ride data frame as a middle ground. While merging the datasets together we noticed that not all driver IDs from the driver dataset matched the driver IDs from the ride dataset, as well as not all the ride IDs from the time dataset matching the ride IDs from the ride dataset. We dropped any datapoints that were not fully matching because we had plenty of data to work with (184209 data points or 95% of the ride data). Along with that, data imputation for the driver or time dataset wouldn’t have been very valuable because we weren't sure whether the drivers drove no rides or simply were not measured in the rides dataset. Because the missing time data only accounted for 1% anyways, data imputation wouldn't have been wise. In the end, we were able to merge everything and create a master dataset of 184209 points that contains information about the ride, the driver, and the times.

**Analysis and Results** 

**Conclusion**

# What's Next? 

# Badges
[![GitHub issues](https://img.shields.io/github/issues/bew030/lyft-challenge?color=purple)](https://github.com/bew030/lyft-challenge/issues)
[![GitHub forks](https://img.shields.io/github/forks/bew030/lyft-challenge?color=orange)](https://github.com/bew030/lyft-challenge/network)
[![GitHub stars](https://img.shields.io/github/stars/bew030/lyft-challenge)](https://github.com/bew030/lyft-challenge/stargazers)
[![Views](http://hits.dwyl.io/bew030/lyft-challenge.svg)](http://hits.dwyl.io/bew030/lyft-challenge)
