---
layout: post
title:  "Live Stream Tweets Into Oracle"
date:   2019-02-20 19:38:03 +0000
permalink: /tweetoracle
---

As usual I'm ~~ripping off other people's work~~ standing on the shoulders of giants. Remember that the point of this blog is to learn more about database technologies and how developers interact with them. I hope to be able to review (amongst other things) database deployment, resilience and performance. As part of that I figured I'm going to need some kind of test harness and for fun (and because I'd seen it done before) I thought I'd work out how to get live data from Twitter and push that into a database. As I'm an Oracle DBA I'm starting with Oracle but ideally we'll be able to use the same method for other databases too - making it repeatable.

I used a couple of other blogs to set this up. I started with Adil Moujahid's great post [here](http://adilmoujahid.com/posts/2014/07/twitter-analytics/). He got me streaming the data but he puts them into a text file and then does clever analysis on that file but that's not what I wanted. Still, his steps are really useful to get you started with Twitter's API, python and the python library Tweepy. What has changed since his post though is it's less straight-forward to get access to Twitter's API. You can follow the same steps but you'll be forced to create a Developer account. This is just more checks and balances to stop Twitter's service from being abused. Fill in the necessary information and you'll get your API credentials right away.

In order to get the tweets to be ingested into my Oracle database I used the information available in Arthur Dayton's post [here](http://www.vlamis.com/blog/2016/10/3/twitter-live-feed-with-oracle-database-as-a-service-and-business-intelligence-cloud-service). He does some cool BI things with the data and he does it all in Oracle's cloud but I'm just doing mine on an internal database. I'm going to reproduce the code here as I had a few typos I had to deal with that weren't easy to diagnose due to my lack of python experience.

Here's the python code for stream.py:
``` python
#import libraries
from tweepy import Stream
from tweepy import OAuthHandler
from tweepy.streaming import StreamListener
import cx_Oracle
import datetime
import json


#connection string for database
conn_str='twitter_user/din0saur@localhost:1521/ORCL'

#get me a connection
conn =cx_Oracle.connect(conn_str)

#turn on autocommit
conn.autocommit=1

#object for executing sql
c=conn.cursor()

#clob variable
bvar=c.var(cx_Oracle.CLOB)

#twitter api application keys
#consumer key, consumer secret, access token, access secret.
ckey='Dont'
csecret='Tell'
atoken='Anybody'
asecret='The Password'

#listen to the stream
class listener(StreamListener):

#get some
    def on_data(self, data):
         try:

            #barf response insto json object
            all_data = json.loads(data)

            #parse out tweet text, screenname and tweet id
            tweet = all_data["text"]
            if (all_data["user"]["screen_name"]) is not None:
                username = all_data["user"]["screen_name"]
            else:
                username = 'No User'
            tid = all_data["id"]

            #set clob variable to json doc
            bvar.setvalue(0,data)
            try:
                #create sql string with bind for clob var

                sql_str="INSERT INTO EAT_MY_TWEET (ID,USERNAME,TWEET_JSON) Values("+str(tid)+",q'["+username.encode('utf-8').strip()+"]',:EATIT)"

                #insert into database
                c.execute(sql_str,[bvar])

            except Exception:
                sys.exc_clear()

            #watch tweets go by in console
            print((username,tweet))

            #in case you want to print response
            #print(data)
            return(True)
         except Exception:
            sys.exc_clear()
    def on_error(self, status):
        print status
#     print(data)
#     print sql_str

#log in to twitter api
auth = OAuthHandler(ckey, csecret)
auth.set_access_token(atoken, asecret)

#fire it up
twitterStream = Stream(auth, listener())

#what to search for (not case sensitive)
#comma separated for words use
# for hashtag
#phrases are words separated by spaces (like this comment)
twitterStream.filter(track=["oracle,database,dinosaur"])
```

Here's the SQL code for the view. It had an extra carraige return.
``` sql
CREATE OR REPLACE FORCE EDITIONABLE VIEW "TWITTER_USER"."LIVE_TWITTER_FEED" ("CST_DATE", "UTC_DATE", "UTC_HOUR", "UTC_MINUTE", "ID", "CREATED_ON", "SCREEN_NAME", "LOCATION", "FOLLOWERS_CNT", "FRIENDS_CNT", "LISTED_CNT", "FAVOURITES_CNT", "STATUSES_CNT", "RETWEET_CNT", "FAVOURITE_CNT", "URL", "PROFILE_IMAGE_URL", "BANNER_IMAGE_URL", "HASHTAGS", "TWEET", "EMT_TIMEWHYUPUNISHME") AS 
  SELECT cast(TO_TIMESTAMP_TZ(REPLACE(upper(b.created_on),'+0000','-00:00'),'DY MON DD HH24:MI:SS TZH:TZM YYYY')  at Time zone 'CST' as date) CST_DATE,
cast(TO_TIMESTAMP_TZ(REPLACE(upper(b.created_on),'+0000','-00:00'),'DY MON DD HH24:MI:SS TZH:TZM YYYY') as date) UTC_DATE,
cast(to_char(cast(TO_TIMESTAMP_TZ(REPLACE(upper(b.created_on),'+0000','-00:00'),'DY MON DD HH24:MI:SS TZH:TZM YYYY') as date),'HH24') as number) UTC_HOUR,
cast(to_char(cast(TO_TIMESTAMP_TZ(REPLACE(upper(b.created_on),'+0000','-00:00'),'DY MON DD HH24:MI:SS TZH:TZM YYYY') as date),'MI') as number) UTC_MINUTE,
a."ID",b."CREATED_ON",a."USERNAME",b."LOCATION",b."FOLLOWERS_CNT",b."FRIENDS_CNT",b."LISTED_CNT",b."FAVOURITES_CNT",b."STATUSES_CNT",
b."RETWEET_CNT",b."FAVOURITE_CNT",b."URL",b."PROFILE_IMAGE_URL",b."BANNER_IMAGE_URL",b."HASHTAGS",b."TWEET",
a.TIMEWHYUPUNISHME
FROM EAT_MY_TWEET a,
json_table(tweet_json,
'$' columns(
    id varchar(50) path '$.id',
    created_on varchar2(100) path '$.created_at',
    screen_name varchar2(200) path '$."user".screen_name',
    location varchar2(250) path '$."user"."location"',
    followers_cnt number path '$."user".followers_count',
    friends_cnt  number path '$."user".friends_count',
    listed_cnt  number path '$."user".listed_count',
    favourites_cnt  number path '$."user".favourites_count',
    statuses_cnt  number path '$."user".statuses_count',
    retweet_cnt number path '$.retweet_count',
    favourite_cnt number path '$.favorite_count',
     url varchar2(250) path '$."user"."url"',
    profile_image_url  varchar2(500) path '$."user".profile_image_url',
    banner_image_url varchar2(500) path '$."user".profile_banner_url',
     hashtags varchar2(500) format json with wrapper path '$.entities.hashtags[*].text',
      tweet varchar2(250) path '$.text'
    -- nested path '$.entities.hashtags[*]' columns (ind_hashtag varchar2(30) path '$.text'    )
)
) b
```

Next steps are to find out what needs to be changed for other databases. And I'd like to wrap this up in a way to quickly deploy without too many changes.