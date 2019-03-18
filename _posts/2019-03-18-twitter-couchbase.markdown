---
layout: post
title:  "Live Stream Tweets into Couchbase"
date:   2019-03-18 20:38:03 +0000
permalink: /tweetcouchbase
---

Getting the Twitter streaming into an Oracle database was kind of cool. So why not use a different database and try to learn a bit more?

I'm lucky enough to have access to on-demand VMs with Couchbase already installed. If you're here to learn about a python script to stream tweets into Couchbase you probably already have a Couchbase database. If you don't, take a look at Couchbase's website - it's pretty easy to get set up on a machine or even in a Docker container. You'll obviously have to update your host details in anything below for your Couchbase server.

#### Bucket

I created a new bucket in my Couchbase database. To aid my learning of Couchbase I decided to do this using the rest API rather than the GUI.

Instructions [here](https://docs.couchbase.com/server/6.0/rest-api/rest-bucket-create.html)

```
curl -X POST -u Administrator:password http://cbhost:8091/pools/default/buckets \
-d name=tweeter -d bucketType=couchbase -d ramQuotaMB=100 -d authType=none \
-d proxyPort=11216
```

#### SDK

Now we need to prepare the python SDK for Couchbase.

Docs [here](https://docs.couchbase.com/python-sdk/2.5/start-using-sdk.html).

First step is to install the libcouchbase library. "It depends on the C SDK, libcouchbase, which it uses for performance and reliability." Instructions [here](https://developer.couchbase.com/server/other-products/release-notes-archives/c-sdk).

```
wget http://packages.couchbase.com/releases/couchbase-release/couchbase-release-1.0-4-x86_64.rpm
sudo rpm -iv couchbase-release-1.0-4-x86_64.rpm
yum install libcouchbase-devel libcouchbase2-bin gcc gcc-c++

yum install gcc gcc-c++ python-devel python-pip
pip install couchbase
pip install tweepy
```

#### Test

Let's test with the sample code from the Couchbase website.

We have to create a user for python access.
I created the user in the web interface as the API looks a bit excessive for this bit of work.
username: twitter_user
password: secret

Privileges:
Bucket Full Access for tweeter bucket only.
Data Reader for tweeter bucket only.
Data Writer for tweeter bucket only.

Using the code from the docs we can test. Remember to update the bucket name in the create index and select queries (I learned the hard way).
``` python
from couchbase.cluster import Cluster
from couchbase.cluster import PasswordAuthenticator
cluster = Cluster('couchbase://cbhost')
authenticator = PasswordAuthenticator('username', 'password')
cluster.authenticate(authenticator)
cb = cluster.open_bucket('tweeter')
cb.upsert('u:king_arthur', {'name': 'Arthur', 'email': 'kingarthur@couchbase.com', 'interests': ['Holy Grail', 'African Swallows']})
# OperationResult<RC=0x0, Key=u'u:king_arthur', CAS=0xb1da029b0000>

cb.get('u:king_arthur').value
# {u'interests': [u'Holy Grail', u'African Swallows'], u'name': u'Arthur', u'email': u'kingarthur@couchbase.com'}

## The CREATE PRIMARY INDEX step is only needed the first time you run this script
cb.n1ql_query('CREATE PRIMARY INDEX ON tweeter').execute()
from couchbase.n1ql import N1QLQuery
row_iter = cb.n1ql_query(N1QLQuery('SELECT name FROM tweeter WHERE ' +\
'$1 IN interests', 'African Swallows'))
for row in row_iter: print(row)
# {u'name': u'Arthur'}
```

After running this the data is displayed in the console and you can see the entry in the database using the GUI.

#### Twitter streaming code

Now let's update our twitter streaming script. This took me ages to get right. In the end I simplified a lot of the python code (because I didn't understand it) and here's the working script:

``` python
from tweepy import Stream
from tweepy import OAuthHandler
from tweepy.streaming import StreamListener
import datetime
import json
import argparse
from couchbase.cluster import Cluster
from couchbase.cluster import PasswordAuthenticator

#parse arguments
parser = argparse.ArgumentParser(description='Import live tweets to Couchbase database')
parser.add_argument('-p', action='store', dest='password', help="Password for database user", default="secret")
parser.add_argument('-u', action='store', dest='username', help="Username for database user", default="twitter_user")
parser.add_argument('-b', action='store', dest='bucket', help="Bucket Name", default="tweeter")
parser.add_argument('-m', action='store', dest='host', help="Database host", default="localhost")
parser.add_argument('-w', action='store', dest='words', help="Twitter search words", default="dinosaur")
inputs = parser.parse_args()



cluster = Cluster('couchbase://' + inputs.host)
authenticator = PasswordAuthenticator(inputs.username, inputs.password)
cluster.authenticate(authenticator)
cb = cluster.open_bucket(inputs.bucket)

#twitter api application keys
#consumer key, consumer secret, access token, access secret.
ckey='Dont'
csecret='Tell'
atoken='Anybody'
asecret='The Password'



#listen to the stream
class myListener(StreamListener):

#get some
    def on_status(self, status):
        print status.id,status.text
        cb.upsert(status.id_str,{'user': status.user.screen_name, 'text': status.text})

#log in to twitter api
auth = OAuthHandler(ckey, csecret)
auth.set_access_token(atoken, asecret)

#fire it up
twitterStream = Stream(auth, myListener())

#what to search for (not case sensitive)
#comma separated for words use
# for hashtag
#phrases are words separated by spaces (like this comment)
twitterStream.filter(track=[inputs.words])
```

#### Docker

Now let's put this into a Docker container, cause that's cool, right?


The Couchbase client requires some wget and yum install so we still can't use the python base image. Here's the working Dockerfile.

``` docker
# Pull Oracle CentOS image from Docker hub
FROM centos

# Install OS packages
RUN yum -y install curl epel-release 
RUN yum -y install python-pip python-devel

# Add couchbase client
RUN wget http://packages.couchbase.com/releases/couchbase-release/couchbase-release-1.0-4-x86_64.rpm
RUN rpm -iv couchbase-release-1.0-4-x86_64.rpm
RUN yum -y install libcouchbase-devel libcouchbase2-bin gcc gcc-c++

WORKDIR /usr/src/app

RUN pip install  --no-cache-dir tweepy
RUN pip install  --no-cache-dir couchbase

COPY . .

ENTRYPOINT [ "python", "./couchstream.py" ]

CMD [ "-p", "secret", "-u", "twitter_user", "-b", "tweeter", "-m", "ut011542", "-w", "dinosaur" ]
```

I had to remove ```print status.id,status.text``` from the python script that's copied into docker and update the cb.upsert to the following to handle UTF8 characters:
```cb.upsert(status.id_str,{'user': status.user.screen_name.encode("utf-8"), 'text': status.text.encode("utf-8")})```


Then build the image:
```
docker build -t stream-to-cb .
```

And run it.
```
docker run -it --rm --name stream-cb-app stream-to-cb
```

And it worked! This took me far longer than it should have. And I did so much hacking small changes that I'm not even sure I learned much. I think that'll do for playing with streaming tweets into databases for now. Time to play with something else. I'll almost certainly use these docker images in the future though.
