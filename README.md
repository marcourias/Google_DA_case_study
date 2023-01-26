# Google_DA_case_study
This is the solution to the Google Data Analysis Case Study

## ASK

What is the problem you are trying to solve? 
In this case study the goal is to find out how consumers use non-Bellabeat smart devices.

How can your insights drive business decisions?
By finding out what the consumers are behaving and looking for in products of a similar category, it’s possible to focus the efforts of the company.


## PREPARE

The data set is stored in a BigQuery account (Data analyst’s account)
Data is presented in a long format, because there are values that repeat, for example id’s are repeated several times in the left column. Which constitutes the characteristics of a long format. (Zach, 2021)
The data is reliable. It complies with ROCCC characteristics: 
Reliable, because it’s coming from a reputable source, well organized and not missing data. 
Original, it was captured by a first party that uploaded it publicly on the Kaggle platform.
Comprehensive, in this case the data is related to the business task, because it contains information about how consumers are doing their activities, including fitness and overall health performance, which is the main market of Bellabeat.
Current, considering the date of last update (2 years ago) we can say it’s still useful and current to fulfill the objectives.
Cited, the information comes from a reputable source and will be cited for reference.
The data will be presented to the stakeholders, in a way that can be interpreted in an inclusive way, also the data doesn’t contain names or a way to identify people with their data.
Analyzing this dataset may provide insights of how customers are behaving, their needs and how the company can fulfill those needs.
The data is organized in a long format in 18 csv files, containing data about activity, calories, steps, sleep and weight. Each file contains thousands of rows representing changes in each point, using ints, dates, hours.
Using complementary data will be useful, such as data about health organizations about recommended activities: daily exercise recommended, daily calorie suggested for an adult, among other data will allow us to compare how customers are behaving in comparison to find opportunity areas. Complementary data to be used:
Calorie intake: https://www.who.int/news-room/fact-sheets/detail/healthy-diet.
Daily Steps recommended:
 https://jamanetwork.com/journals/jamanetworkopen/fullarticle/2783711?utm_source=For_The_Media&utm_medium=referral&utm_campaign=ftm_links&utm_term=090321
Hours of sleep recommended:
https://www.cdc.gov/sleep/about_sleep/how_much_sleep.html



## PROCESS

The chosen tools to process the data are BigQuery and Tableau, according to the characteristics of the data set I find that the easiest tools are using SQL provided by Google’s platform for data cleaning and manipulation, in conjunction with Tableau for visualization.
The data was loaded (.csv files) to a personal BigQuery account (free) where there was a dataset created, named caseS.
Checking several files and loading, it was needed to load them as string, even when some columns had date time data or int. The aforementioned due to BigQuery not recognizing data as Data time automatically.

Then some data had to be converted once loaded. To do this, cast and parse functions were used, for example:

SELECT Id, PARSE_DATETIME('%m/%d/%Y %I:%M:%S %p', Time) AS Time, CAST(Value AS int) AS Value
FROM `caseS.heartrate`;


SELECT Id, CAST(Date AS DATE) as Date, MAX(sum(Value) AS sleepMinutes)
FROM `caseS.minuteSleepCast`
GROUP BY Id, CAST(Date AS DATE), LogId;


Once selected the data, it was saved in a new BigQuery table, as updating the same dataset with that query was not possible in a free account (to update it, BigQuery required a payment, according to an error message)

Saving a new files with correct attributes (datetime, int and float) makes it possible to run queries such as AVG or extract dates.



PROCESS

To make analysis possible it’s convenient to extract data from some of the datasets, and create new tables that are simpler and easier to understand, using less columns to perform calculations such as addition or average of values.
Even when at the beginning the dataset had some issues to be imported to BigQuery, once functions as CAST were executed, it was possible to make calculations.
Some of the data was really surprising, among others, I can mention:
The daily steps taken by each person ranges between less than 2,000 to even more than 16,000 daily steps.
The daily average calorie intake ranged between 1483 and 3436 calories.
The logged activity distance is not used that much, only 4 of 33 used it, the rest of users had 0.
Another interesting data is the Daily Hours of sleep each user logged. It’s remarkable that some users registered barely more than an hour of sleep (average in a day) which obviously it's impossible, letting us know that users are not logging their activity so strictly.
The process of analysis was registered in a queries in BigQuery:


-- Create a table with the total of steps each costumer took in the period of time.
SELECT  Id,  sum(TotalSteps) AS StepsbyCostumer
FROM `case-study-375101.caseS.dailyActivity`
GROUP BY Id;


-- Create a table with the total distance each costumer took in the period of time.
SELECT  Id,  sum(TotalDistance) AS DistancebyCostumer
FROM `case-study-375101.caseS.dailyActivity`
GROUP BY Id;


-- Create a table with the total of steps each costumer took in the period of time.
SELECT  Id,  sum(Calories) AS Calories
FROM `case-study-375101.caseS.dailyActivity`
GROUP BY Id;




SELECT ActivityDate
FROM `case-study-375101.caseS.dailyActivity`
GROUP BY ActivityDate;




-- What was the period of time the data was collected?
-- Let's check the starting and last dates, so we search for the min and max, in the column ActivityDate:


SELECT min(ActivityDate) as FirstDate
FROM `caseS.dailyActivity`;
-- Result: 2016-04-12




Select max(ActivityDate) as LatestDate
FROM `caseS.dailyActivity`;
-- Result: 2016-05-12




-- MEAN


-- Checking the mean of Daily Steps


Select AVG(TotalSteps) as MeanTotalSteps
FROM `caseS.dailyActivity`;
-- Result: 7637.9106




-- Checking the mean Daily Calories


Select AVG(Calories) as MeanCalories
FROM `caseS.dailyActivity`;
-- Result: 2303.6095




-- Checking the mean Daily TotalDistance


SELECT AVG(TotalDistance) as MeanDistance
FROM `caseS.dailyActivity`;
-- Result: 5.4897




-- Using the dailyCalories table to find the mean 
select Id, avg(Calories) as AverageDailyCalories
from `caseS.dailyCalories`
group by Id;




-- Using dailySteps table to find the average DailySteps per ostumer


SELECT Id, AVG(StepTotal) as AverageDailySteps
FROM `caseS.dailySteps`
GROUP BY Id;




-- Using dailyActivity to check costumer behavior
-- Checking how many Distances activities are logged
SELECT SUM(LoggedActivitiesDistance)
FROM `caseS.dailyActivity`
GROUP BY Id;


-- Using heartrate to check the average heartrates by costumer


-- It's necessary to parse the datetime and Value to convert them to Time and Int, instead of string:
SELECT Id, PARSE_DATETIME('%m/%d/%Y %I:%M:%S %p', Time) AS Time, CAST(Value AS int) AS Value
FROM `caseS.heartrate`;


-- Getting the average and grouping by 
SELECT Id, AVG(Value) as heartrateAvg
FROM `caseS.heartrateCast`
GROUP BY Id;


-- Using minuteSleep to check how many hours of sleep are costumers getting


-- First we need to convert time and value to datetime and int format.


SELECT Id, PARSE_DATETIME('%m/%d/%Y %I:%M:%S %p', Date) AS Date, CAST(Value AS int) AS Value, LogId
FROM `caseS.minuteSleep`;


-- Now that we have a new table with correct formating of the data we can find how many minutes of sleep the costumers got.


SELECT *
FROM `caseS.minuteSleepCast`
LIMIT 2000;




SELECT Id,sum(Value)
FROM `caseS.minuteSleepFix`
GROUP BY Id;




SELECT Id, CAST(Date AS DATE) as Date, MAX(sum(Value) AS sleepMinutes)
FROM `caseS.minuteSleepCast`
GROUP BY Id, CAST(Date AS DATE), LogId;


-- Based on the last table created we can take the average sleeps periods as the main sleep.


SELECT Id, Round((Avg(sleepMinutes)/60), 2) as DailyHoursSleep
FROM `caseS.dailySleep`
GROUP BY Id;


## ANALYZE

The data was all collected in a BigQuery database called caseS. In there were the main csv files (converted into BigQuery’s format)
Some of the data was imported as string raw data, because there was an issue importing the column containing datetime. However, those files were later converted (using CAST por PARSE) to the proper format (int64 and datetime)
The main trends or relationships were found in 5 topics:
Heart rate
Daily distance
Daily Calorie intake
Daily steps
Daily hours of sleep
To acquire this data, it was needed to perform some calculations, using addition, average, division, and grouping data according to costumer or date.
While performing this operations, new tables were created in order to prepare the data for exporting and later use it into Tableau for visualization.
Finally, for reference and comparison health and fitness data was consulted in trusted sources such as the World Health Organization, or National Health institutes. This way, it was possible to compare how customers are behaving compared to what is recommended. For example, how many calories are clients consuming compared to the recommended value.

## SHARE

The data was exported to Tableau Desktop, through the BigQuery import option. That way, 5 data visualization charts were created, and then sent to 3 dashboards in Tableau Public.

<div class='tableauPlaceholder' id='viz1674708195719' style='position: relative'><noscript><a href='#'><img alt='Insights on Fitness Data ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Go&#47;GoogleCertification-CaseStudy&#47;Historia2&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='GoogleCertification-CaseStudy&#47;Historia2' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;Go&#47;GoogleCertification-CaseStudy&#47;Historia2&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='es-ES' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1674708195719');                    var vizElement = divElement.getElementsByTagName('object')[0];                    vizElement.style.width='1016px';vizElement.style.height='991px';                    var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>


