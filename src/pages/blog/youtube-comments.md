---
layout: "../../layouts/BlogPost.astro"
title: "Pulling YouTube Comments With Python"
description: "YouTube Python API"
pubDate: "Oct 24 2021"
heroImage: "/python-hero.jpg"
previewText: "YouTube comments are a treasure trove of information. Wouldn't it be nice if you could pull YouTube comments and analyze them? With the YouTube Python API, you can."
---

## Overview

YouTube comments are a treasure trove of information. Wouldn't it be nice if you could pull YouTube comments and analyze them? With the YouTube Python API, you can.

In this article, I discuss how to get a YouTube API key, how to get all YouTube comments from a video, filtering out unwanted HTML characters, and even touch on how to make a word cloud using the WordCloud python package.

This article also exists in an interactive [Jupyter Notebook](https://colab.research.google.com/gist/Timothy-Pulliam/f5e512e4b7d71734877878fc39701322/pullyoutubecomments.ipynb#scrollTo=WwGcr_jsKvDp&line=1&uniqifier=1).

## Creating a YouTube API Key

First thing we must do is create an API key. You can do this by logging in to [Google Developer Console](https://console.cloud.google.com/apis).

I have also made a video showing how to do this.

<iframe width="560" height="315" src="https://www.youtube.com/embed/kD7Lu2ORqpo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

## Required Packages

We need to install the necessary Python Packages.

`pip install google-api-python-client matplotlib wordcloud`

At the top of your python file, import the following packages

```python
from googleapiclient.discovery import build
from googleapiclient.errors import HttpError
from wordcloud import WordCloud
import html
import matplotlib.pyplot as plt
import re
```

## Top Level Comments

First, we will see how to get Top Level Comments, or threads. These are comments which themselves contain other comments. For a reference of the API, see the below docs, which include an Interactive API Explorer. It even generates Python code for you.

https://developers.google.com/youtube/v3/docs/commentThreads/list

![API Reference](/youtube-comments/api-reference.webp)

```python
# don't share with anyone ever
YOUTUBE_API_KEY = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX'


def get_comment_threads(videoId, maxResults=20, pageToken=None):
    """Returns top level comments. Only between 1-100 comments can be returned
    in a single request. For more requests, you must specify the next page of
    comments using the pageToken to specify the page you wish to return.

    maxResults: number of threads per page (between 1 and 100 inclusive)
    pageToken: page to retrieve"""
    with build('youtube', 'v3', developerKey=YOUTUBE_API_KEY) as service:
        request = service.commentThreads().list(part='snippet,replies', videoId=videoId, maxResults=maxResults, pageToken=pageToken)
        try:
            response = request.execute()
        except HttpError as e:
            print('Error response status code : {0}, reason : {1}'.format(e.status_code, e.error_details))
            return {}
        return response
```

We need to go three layers deep to get to the meaty comment data within. The reponse that is returned back will look like this.

Sample `commentThreadListResponse` resource

```
  "kind": "youtube#commentThreadListResponse",
  "etag": etag,
  "nextPageToken": string,
  "pageInfo": {
    "totalResults": integer,
    "resultsPerPage": integer
  },
  "items": [
    commentThread Resource
  ]
}
```

It is of type `youtube#commentThreadListResponse`. We can see there is some metadata, along with the `items` attribute, which is a list of `commentThread` resource types. The `nextPageToken` allows us to page through the returned results. We are interested in the `items` attribute, which contains the top-level comments.

A Comment Thread Resource has the following JSON structure. It is what is contained in the items list. Some more metadata, but ultimately we are interested in `snippet.topLevelComment` and `replies.comments`. `snippet.topLevelComment` is a `youtube#comment` type, and `replies.comments` is a list of `youtube#comment` types.

Sample `commentThread` Resource

```
  "kind": "youtube#commentThread",
  "etag": etag,
  "id": string,
  "snippet": {
    "channelId": string,
    "videoId": string,
    "topLevelComment": comments Resource,
    "canReply": boolean,
    "totalReplyCount": unsigned integer,
    "isPublic": boolean
  },
  "replies": {
    "comments": [
      comments Resource
    ]
  }
}
```

And then finally, the `youtube#comment` type. This is the text of the data that we are actually after.

Sample `comment` resource

```
  "kind": "youtube#comment",
  "etag": etag,
  "id": string,
  "snippet": {
    "authorDisplayName": string,
    "authorProfileImageUrl": string,
    "authorChannelUrl": string,
    "authorChannelId": {
      "value": string
    },
    "channelId": string,
    "videoId": string,
    "textDisplay": string,
    "textOriginal": string,
    "parentId": string,
    "canRate": boolean,
    "viewerRating": string,
    "likeCount": unsigned integer,
    "moderationStatus": string,
    "publishedAt": datetime,
    "updatedAt": datetime
  }
}
```

If we add the following code, we can see these resource types returned.

```python
# The response
response = get_comment_threads('VZun3DhJvpE')
print("Response")
print(thread.keys())

# A single comment thread
threads = response['items']
print("\nA Single Comment Thread Resource")
print(threads[0].keys())
# The snippet object contains basic details about the comment thread.
# It also contains the thread's top-level comment, which is a comment resource.
print(threads[0]['snippet'].keys())

# Getting the actual top level comment from a thread
print("\nTop Level Comment")
print(threads[0]['snippet']['topLevelComment']['snippet'].keys())
print(threads[0]['snippet']['topLevelComment']['snippet']['textDisplay'])
```

Output

```python
Response
dict_keys(['kind', 'etag', 'nextPageToken', 'pageInfo', 'items'])

A Single Comment Thread Resource
dict_keys(['kind', 'etag', 'id', 'snippet'])
dict_keys(['videoId', 'topLevelComment', 'canReply', 'totalReplyCount', 'isPublic'])

Top Level Comment
dict_keys(['videoId', 'textDisplay', 'textOriginal', 'authorDisplayName', 'authorProfileImageUrl', 'authorChannelUrl', 'authorChannelId', 'canRate', 'viewerRating', 'likeCount', 'publishedAt', 'updatedAt'])
“What happens after a drought or the end of a pandemic”. This episode is more relevant today than ever before.
```

The `maxResults` parameter specifies the maximum number of items that should be returned per request. Acceptable values are 1 to 100, inclusive. The default value is 20. This means we can only get up to 100 comments at a time. In order to get more comments, you must pass `nextPageToken` as an argument to get the next page of comments.

```python
pages = []
nextPageToken = None
# get the first 500 comments
for i in range(5):
    print("getting page " + str(i + 1))
    response = get_comment_threads('R_wscUcbynk', maxResults=100, pageToken=nextPageToken)
    pages.append(response)
    nextPageToken = response['nextPageToken']
with open('comments.txt', 'w') as f:
    for page in pages:
        for thread in page['items']:
            f.write(thread['snippet']['topLevelComment']['snippet']['authorDisplayName'] + ' Said ')
            f.write(thread['snippet']['topLevelComment']['snippet']['textDisplay'] + '\n')
```

## Getting All the Comments

Break out of a while loop once there is no longer a `nextPageToken` to retrieve.

```python
pages = []
response = get_comment_threads('j4xCb_OU_lM', maxResults=100, pageToken=None)
pages.append(response)
response.keys()
nextPageToken = response['nextPageToken']
while True:
    print("getting page " + str(len(pages) + 1))
    response = get_comment_threads('j4xCb_OU_lM', maxResults=100, pageToken=nextPageToken)
    pages.append(response)
    try:
        nextPageToken = response['nextPageToken']
    except KeyError:
        break
```

## Escaping HTML Characters

You may get strange characters like

```
TheWagelessMage Said I&#39;ll be honest. I&#39;m 90% here for the music
```

We can turn these into regular HTML characters using `html.unescape` from the html module. For example

```python
import html

html.unescape("TheWagelessMage Said I&#39;ll be honest. I&#39;m 90% here for the music")
# TheWagelessMage Said I'll be honest. I'm 90% here for the music
```

We can also remove HTML tags using regular expressions

```python
# Get rid of HTML tags in text
# as per recommendation from @freylis, compile once only
text = '<br>(<a href="https://www.youtube.com/watch?v=R_wscUcbynk&amp;t=0m00s">0:00</a>) HOME - We’re Finally Landing'

CLEANR = re.compile('<.*?>')
def removeHtmlTags(raw_html):
  cleantext = re.sub(CLEANR, '', raw_html)
  return cleantext

text = removeHtmlTags(text)
print(text)
# (0:00) HOME - We’re Finally Landing
```

## Simple Sentiment Analysis

From here, we could do some simple sentiment analysis. What are people saying in the comments? Do they like the video? Let's see if the word "love" is used. Looking at the results, it sounds like people like instructinoal content.

```python
threads = get_comment_threads('-JMYaBZIesI')['items']
for thread in threads:
    if 'love' in thread['snippet']['topLevelComment']['snippet']['textDisplay']:
        print(thread['snippet']['topLevelComment']['snippet']['textDisplay']+"\n")
```

Output

```
So wholesome thank you i love it

As someone who watches to learn, this video is nice and would love to see more!<br>Any time you can get a top players genuine insight is a gem.

I loved hearing Marss actually teach
```

## Replies to Top Level Comments

```python
threads = get_comment_threads('-JMYaBZIesI')['items']
for thread in threads:
    # only get threads with replies
    if thread['snippet']['totalReplyCount'] > 0:
        parent_comment = thread['snippet']['topLevelComment']['snippet']
        # original poster
        op = parent_comment['authorDisplayName']
        time = parent_comment['publishedAt']
        print("{} said at {}".format(op, time))
        print(parent_comment['textDisplay'] + "\n")
        # sort replies by date posted
        for reply in sorted(thread['replies']['comments'], key=lambda x: x['snippet']['publishedAt']):
            responder = reply['snippet']['authorDisplayName']
            time = reply['snippet']['publishedAt']
            print("{} replied to {} at {}".format(responder, op, time))
            print(reply['snippet']['textDisplay'] + "\n")
```

Output

```
mirrorzedge523 said at 2021-09-07T01:18:07Z
Does anyone know the name of that song at the end of the video?

John Eckhardt replied to mirrorzedge523 at 2021-09-07T04:06:47Z
Perturbation from persona 5 strikers

mirrorzedge523 replied to mirrorzedge523 at 2021-09-07T08:16:22Z
@John Eckhardt Thanks man
```

Looks pretty good, except for one issue. Anyone who replies will always show as replying to the original commenter. In the last comment OP is replying the themselves, which is obviously not the case. This is a tricky issue which I have yet to work out.

## Word Cloud

Let's create a word cloud of all the words from all of the comments to see what everyone is saying.

```python
CLEANR = re.compile('<.*?>')
def removeHtmlTags(raw_html):
  cleantext = re.sub(CLEANR, '', raw_html)
  return cleantext

threads = get_comment_threads('R_wscUcbynk', 100)['items']

with open('word_cloud.txt', 'w') as f:
    for thread in threads:
        f.write(removeHtmlTags(html.unescape(thread['snippet']['topLevelComment']['snippet']['textDisplay'])) + '\n')
        if thread['snippet']['totalReplyCount'] > 0:
            for reply in thread['replies']['comments']:
                f.write(removeHtmlTags(html.unescape(reply['snippet']['textDisplay'])) + '\n')

text = open('word_cloud.txt').read()

# Add to list of stop words
STOPWORDS.update(['https'])
wordcloud = WordCloud(stopwords=STOPWORDS).generate(text)


# Display the generated image:
plt.figure(figsize=(10,10))
plt.imshow(wordcloud, interpolation='bilinear')
plt.axis("off")

# lower max_font_size
wordcloud = WordCloud(max_font_size=40).generate(text)
plt.figure(figsize=(10,10))
plt.imshow(wordcloud, interpolation="bilinear")
plt.axis("off")
plt.show()
```

Here is the resulting word cloud
![Word Cloud](/youtube-comments/wordcloud1.png)

Here is the same word cloud with smaller font size
![Word Cloud](/youtube-comments/wordcloud2.png)

## Conclusion

Of course this is just the tip of the iceberg. How will we structure our data? How will we store it? What sort of analysis could we perform? What questions do we hope to answer? These are questions that we will dive into later, as they they depend upon the project at hand. However we have taken the first step; You can now get comments from YouTube videos.
