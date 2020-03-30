# Churn-user-prediction-on-sparkify

## 1 Introduction
The project is aiming to predict the churn users from the user logging data on a musical service platform called **Sparkify**.

In the project, I used `spark.sql` to do the EDA, and used `spark.ml` to built the prediction model to identify users who are at risk to churn . For more detailed information, please visit [my blog](https://zhuanlan.zhihu.com/p/120244148).
## 2 Dataset 
The [dataset(231M)](https://video.udacity-data.com/topher/2018/December/5c1d6681_medium-sparkify-event-data/medium-sparkify-event-data.json) is a subset of the full dataset(12G), it helps to explore and train the model on local server. I downloaded it from Udacity's classroom. 
The dateset has 18 columns as follow:

column name | column meaning
------------|----------------
artist | string (artist name)
auth | string (logging in /logging out .etc)
firstName | string (user's first name)
gender | string (user gender)
itemInSession | long (the sequence number for each page within one session)
lastName | string ((user's last name)
length | double (the length of the song)
level | string (paid or free)
location | string (user's location)
method | string (PUT/GET)
page | string (the pages users requesting)
registration | long (the time user registed)
sessionId | long (everytime users use Sparkify they got an sessionId)
song | string (name of the song)
status | long (200/307/404)
ts | long (time)
userAgent | string (the os and browser the user use)
userId | string (user Id)

## 3 Model Constructing
## 3.1 Data Cleaning
* evaluate the data set：
  * missing data (without userId)
  * data with wrong type
  * more than one feature in some columns
* clean the dataset
  * remove the rows with missing userId
  * change the data type of 'ts', 'registation' and 'status'
  * exact 'state' from 'location'
  * exact 'os' and 'browser' from 'userAgent'
 
## 3.2 EDA
* define the churn user
* explore and visualize the different activities between normal user and churn user
* conclusion about the EDA:
  1. paid_level users are more likely to churn than free_level
  2. days since regist are shorter for churn users
  3. churn users are unevenly distributed in different states
  4. churn users are unevenly distributed with different os and browsers
  5. focusing on the latest log record, the churn users have lower song_page_ratio, lower active_page_ratio, but higher bad_feeling_page_ratio and higher adjust_level_page_ratio.

## 3.3 Feature Engineering
Based on the conclusion of the EDA, create features as follow:
* Label based on "cancellation Confirmation" page
* Static features which are firm for each user
  * 'userId'
  * 'gender'
  * 'state'
  * 'os'
  * 'browser'
* Dynamic features which are changing with time
  * 'now_level': the latest level for each user
  * 'days_since_regist'
  * 'song_ratio': song_pages among all the pages with in the last sessionId for each user
  * 'active_ratio': active_pages (such as: Thumbs up, Add to Playlist, Add Friend) among all the pages with in the last sessionId for each user
  * 'bad_ratio': bad_feeling_pages (such as: Roll Adevet, Error, Thumbs Down) among all the pages with in the last sessionId for each user
  * 'adjust_ratio': adjust_level_pages (such as: Upgrade, Downgrade, Submit Upgrade, Submit Downgrade) among all the pages with in the last sessionId for each user
  * 'newest_timestamp': timestamp between the last two sessionId for each user
## 3,4 Modeling
* Split the dataset into train, cvset and test
* Model selection

model name | train F1 score | cvset F1 score
-----------|----------------|----------------
LogisticRegression | 0.894 | 0.696
GBDT | 1 | 0.731
NaiveBayes | 0.854 | 0.624
Randomforest | 0.776 |0.642

* Model tuning
Choose the GBDT to be our final model, and gain the F1_score for cvset 0.819 and F1_score for testset 0.751 after tuning.

# 4 Result
with the model i built, i find out that the 5 most important features are "newest_timestamp", "song_ratio", "days_since_regist", "bad_ratio" and "active_ratio".

# Packages Needed
* `pyspark`
* `numpy`
* `pandas`
* `datetime`
* `matplotlib`
* `httpagentparser`

# Reference link
Dataset: https://video.udacity-data.com/topher/2018/December/5c1d6681_medium-sparkify-event-data/medium-sparkify-event-data.json
Model evaluation: https://www.zhihu.com/question/30643044/answer/224360465；https://zh.wikipedia.org/wiki/ROC%E6%9B%B2%E7%BA%BF
