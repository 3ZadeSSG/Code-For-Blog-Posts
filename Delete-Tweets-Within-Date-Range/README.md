If you wanted to delete your old tweets within a given date range, this post will walk you through the steps using python-twitter & data obtained from your twitter profile
[GitHub Link of Jupyter Notebook](https://github.com/3ZadeSSG/Code-For-Blog-Posts/blob/master/Delete-Tweets-Within-Date-Range/Tweet_Delete_within_a_date_Range.ipynb)


## **Prerequisite:-**

#### 1. Twitter data archive: 
   We will require twitter data to get two attributes of each tweet:- 
I) Tweet Id and 
II) Tweet date
If you already have these two items in any csv file or any other form then you can skip this step.

To get you twitter data follow this [link](https://twitter.com/settings/your_twitter_data?lang=en) using desktop site
**OR** 
alternatively you can reach it from **Settings & Privacy -> Account -> Download an archive of your twitter data** (After request it will take up to 48 Hrs to make your data available for download)
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/9mvh9kfyikei07xkwnz4.PNG)
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ki2p2w3gm6jtthp21pdm.PNG)

#### 2. Twitter API Credentials

Go to https://developer.twitter.com/en/apps if you have no existing app then create one.
After your app has been created go to it's **"Details"**, make sure the permissions are assigned to **Read, Write and Direct Messages** , otherwise edit and make the changes

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/e9hho4cg1alqpzi17xdy.PNG)

After that you need to generate the Consumer API key and Access Tokens

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/cx4t8y62l4vpvpdzbi1j.PNG)




## **Handling the Downloaded Twitter Data:-** 

After extracting the twitter data you will find two folders one is data and other is assets. The HTML file can be used to browse your tweets offline.
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/n6ro559xbuif5c1pu5ui.PNG)

Before we use the code we need to make some changes in the **tweets.js** file located inside **assets** folder

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/3p2oddm7k3hwf87gn9tl.PNG)

Current twitter data download service lists all your tweet assets inside a tweet.js file as an array of objects. Since we will want this file to be read in python, so we need to make it a JSON file, hence remove the first line 

       window.YTD.tweet.part0 = [ {

and replace it with 
      
       {"data": [ {

for last line you need to add an extra **"}"** to make it a JSON object, hence your file's last line will look like this:-

      } ]}

save it as a JSON file, for example **"editedTweet.js"**


## **Code**:- 

The python jupyter notebook can be found at GitHub with this [link](https://github.com/3ZadeSSG/Code-For-Blog-Posts/blob/master/Delete-Tweets-Within-Date-Range/Tweet_Delete_within_a_date_Range.ipynb)

1) First make sure to uninstall standalone twitter package and install the python version of that (since the normal twitter package doesn't includes the **"_twitter.Api()_"** method), you can do it directly from Jupyter Notebook (you will need to restart notebook after installation)

```python      
!pip uninstall twitter
!pip install python-twitter
```
**OR** from terminal using
```bash
pip uninstall twitter
pip install python-twitter
```




2) Initialize the **CONSUMER_KEY, CONSUMER_SECRET, ACCESS_TOKEN_KEY,
ACCESS_TOKEN_SECRET** variables with your own twitter API credentials, passing tweet id string into **"deleteTweet(tweetId)"** function will delete that tweet. The tweet id is located in the JSON file as **"id_str"** for each tweet. 

```python
# ==================================================================
# Import statements
# ==================================================================

import sys
import time
from datetime import datetime
import os
import twitter
from dateutil.parser import parse
import numpy as np
import pandas as pd
import json


# ==================================================================
# API Credentials
# ==================================================================

CONSUMER_KEY = ""
CONSUMER_SECRET = ""
ACCESS_TOKEN_KEY = ""
ACCESS_TOKEN_SECRET = ""


# ==================================================================
# Initialize
# ==================================================================

api = twitter.Api(consumer_key = CONSUMER_KEY,
                  consumer_secret = CONSUMER_SECRET,
                  access_token_key = ACCESS_TOKEN_KEY,
                  access_token_secret = ACCESS_TOKEN_SECRET)


# ======================================================================================
# Function to delete tweet by ID
# ======================================================================================

def deleteTweet(tweetId):
   try:
     print("Deleting tweet #{0})".format(tweetId))
     api.DestroyStatus(tweetId)
     print("Deleted")
   
   except Exception as err:
      print("Exception: %s\n" % err)
```





3) Read the JSON file into a json variable in python 

```python
myData = None
with open('editedTweet.json') as json_file:
    myData = json.load(json_file)
```
 you can now browse the tweet attributes for each tweet in the array, for reference I have printed contents of element located at index 0, here we are only interested in **"created_at"** and **"id_str"** attributes:-
```
{'tweet': {'created_at': 'Thu Sep 11 12:26:39 +0000 2014',
  'display_text_range': ['0', '137'],
  'entities': {'hashtags': [],
   'symbols': [],
   'urls': [{'display_url': 'fb.me/3oL0wLoge',
     'expanded_url': 'http://fb.me/3oL0wLoge',
     'indices': ['115', '137'],
     'url': 'http://t.co/spMVNltxDk'}],
   'user_mentions': []},
  'favorite_count': '0',
  'favorited': False,
  'full_text': 'for galaxy y , galaxy pocket, galaxy ace, galaxy music, galaxy y dous lite and any other low end android device... http://t.co/spMVNltxDk',
  'id': '510041733099712513',
  'id_str': '510041733099712513',
  'lang': 'en',
  'possibly_sensitive': False,
  'retweet_count': '0',
  'retweeted': False,
  'source': '<a href="http://www.facebook.com/twitter" rel="nofollow">Facebook</a>',
  'truncated': False}}
```





4) We now just need to create an array with id of tweets which needs to be deleted, hence select the date range for tweets which should be deleted, remember to keep the UTC offset into consideration. Use **"range_start"** and **"range_end"**. 

```python
# Range (in UTC offset) within which tweets will be deleted
# =================================================================

range_start = datetime.strptime('Sep 10 00:00:00 +0000 2012','%b %d %H:%M:%S %z %Y')
range_end = datetime.strptime('Sep 10 00:00:00 +0000 2017','%b %d %H:%M:%S %z %Y')

# ==================================================================
# I am creating a list of tweet IDs for consideration, where tweetsToBeDeleted will be
# used for deleting tweet
# ==================================================================

tweetsToBeDeleted = []
tweetsToBeIgnored = []

for element in myData["data"]:
  tweet_post_time = datetime.strptime(element["tweet"]["created_at"],'%a %b %d %H:%M:%S %z %Y')
  if (tweet_post_time>= range_start and tweet_post_time<= range_end ):
    tweetsToBeDeleted.append(element["tweet"]["id_str"])
  else:
    tweetsToBeIgnored.append(element["tweet"]["id_str"])

print(len(tweetsToBeDeleted),len(tweetsToBeIgnored))
```





5) Finally you can iterate over the array and pass each id from **"tweetsToBeDeleted"** array into delete function for the tweets to be removed. 


```python
for id in tweetsToBeDeleted:
  deleteTweet(id)
```
