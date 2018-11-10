---
id: 202
title: Twitter Sentiment Analysis in less than 100 lines of code!
author: rahular
layout: post
excerpt: An extremely simple sentiment analysis engine for Twitter, written in Java with Stanford's NLP library
guid: http://rahular.com/?p=202
permalink: /twitter-sentiment-analysis/
categories:
  - ML
---
When I started learning about Artificial Intelligence, the hottest topic was to analyse the sentiment of unstructured data like blogs and tweets. I tried implementing a module for it and failed miserably due to the lack of ability, time and more importantly libraries! Now that I know a bit of coding and there are libraries lying around on <a href="https://github.com/" target="_blank">GitHub</a>, I planned to give it another shot. I couldn&#8217;t believe how easy it is! So here goes..

You will need to set up a couple of things before we can get started with coding.

## Creating a Twitter App

  * Go to [Twitter&#8217;s developer site][1] and click on **My Applications** which will pop up when you hover on the top-right part of the screen where your profile picture is
  * Click on **Create new app** and fill out the basic information
  * Now go to the **API Keys **section in your app, scroll down and click on **Create my access token**. Refresh the page till your Access token information is reflected on screen
  * Next, create a new Java project in Eclipse and inside the root of the project create a file named **twitter4j.properties**
  * Finally add the following information to it in the same format as below

```java
debug=true
oauth.consumerKey=<api-key-for-your-app>
oauth.consumerSecret=<api-secret-for-your-app>
oauth.accessToken=<access-token>
oauth.accessTokenSecret=<access-token-secret>
```

## Setting up Twitter4J

  * Create a new folder named **lib** in the Eclipse project you created
  * Download the latest stable version of Twitter4J <a href="http://twitter4j.org/en/index.html#download" target="_blank">here</a>
  * Extract files, go to the **libs** folder which was just extracted, copy a file named **twitter4j-core-xxx.jar** into the **lib** folder of your project
  * Add this **jar** file to the **Java Build Path** (Right click on project -> Properties -> Java Build Path -> Libraries tab -> Add Jars)
  * Now it&#8217;s time to pull out some tweets. Create a class named **TweetManager** and copy the following code into it (I hope the code is self-explanatory)

```java
import java.util.ArrayList;
import java.util.List;

import twitter4j.Query;
import twitter4j.QueryResult;
import twitter4j.Status;
import twitter4j.Twitter;
import twitter4j.TwitterException;
import twitter4j.TwitterFactory;

public class TweetManager {

	public static ArrayList<String> getTweets(String topic) {

		Twitter twitter = new TwitterFactory().getInstance();
		ArrayList<String> tweetList = new ArrayList<String>();
		try {
			Query query = new Query(topic);
			QueryResult result;
			do {
				result = twitter.search(query);
				List<Status> tweets = result.getTweets();
				for (Status tweet : tweets) {
					tweetList.add(tweet.getText());
				}
			} while ((query = result.nextQuery()) != null);
		} catch (TwitterException te) {
			te.printStackTrace();
			System.out.println("Failed to search tweets: " + te.getMessage());
		}
		return tweetList;
	}
}
```

  *  If Twitter4J packages are not imported properly, then check if you have indeed added the library to the Java Build Path
  * This class contains a single method called **getTweets** which takes in a topic as parameter, queries and returns a list of tweets

## Setting up Stanford&#8217;s Core NLP

  * Download the latest release of Core NLP library <a href="http://nlp.stanford.edu/software/corenlp.shtml" target="_blank">here</a>
  * Extract files and copy the following files into your project&#8217;s **lib** folder and add it to Java Build Path 
      * ejml-xxx.jar
      * stanford-core-nlp-xxx.jar
      * stanford-core-nlp-xxx-models.jar
  * Now create another file in the project&#8217;s root folder and name it **MyPropFile.properties **(during runtime if there is an error saying cannot find MyPropFile.properties, then move it to the **src** folder) and copy this line into the file

```java
annotators = tokenize, ssplit, parse, sentiment
```

  *  For more information regarding the meaning of each term, see the **Usage **section of the download page
  * Create a class named **NLP **and copy the following code into it

```java
import edu.stanford.nlp.ling.CoreAnnotations;
import edu.stanford.nlp.neural.rnn.RNNCoreAnnotations;
import edu.stanford.nlp.pipeline.Annotation;
import edu.stanford.nlp.pipeline.StanfordCoreNLP;
import edu.stanford.nlp.sentiment.SentimentCoreAnnotations;
import edu.stanford.nlp.trees.Tree;
import edu.stanford.nlp.util.CoreMap;

public class NLP {
	static StanfordCoreNLP pipeline;

	public static void init() {
		pipeline = new StanfordCoreNLP("MyPropFile.properties");
	}

	public static int findSentiment(String tweet) {

		int mainSentiment = 0;
		if (tweet != null && tweet.length() > 0) {
			int longest = 0;
			Annotation annotation = pipeline.process(tweet);
			for (CoreMap sentence : annotation
					.get(CoreAnnotations.SentencesAnnotation.class)) {
				Tree tree = sentence
						.get(SentimentCoreAnnotations.AnnotatedTree.class);
				int sentiment = RNNCoreAnnotations.getPredictedClass(tree);
				String partText = sentence.toString();
				if (partText.length() > longest) {
					mainSentiment = sentiment;
					longest = partText.length();
				}

			}
		}
		return mainSentiment;
	}
}
```

  *  This will initialize the NLP pipeline using the properties file and do some other good stuff, more about which you can read <a href="http://nlp.stanford.edu/~socherr/EMNLP2013_RNTN.pdf" target="_blank">here</a>
  * This class contains two functions namely, **init** which initializes the pipeline and **findSentiment **which takes in a tweet as input and returns it&#8217;s **sentiment score** (Higher the score, happier the sentiment)

## Putting it all together

  * Now that we have everything, it&#8217;s just a matter of integration. Create a class named **What2Think** (random and lame, I know) and copy the following code into it

```java
import java.util.ArrayList;

public class WhatToThink {

	public static void main(String[] args) {
		String topic = "ICCT20WC";
		ArrayList<String> tweets = TweetManager.getTweets(topic);
		NLP.init();
		for(String tweet : tweets) {
			System.out.println(tweet + " : " + NLP.findSentiment(tweet));
		}
	}
}
```

  *  All this does is, get tweets based on the **topic, **initialize the pipeline and feed tweets into the sentiment analyzer, one at a time

And that&#8217;s it. If you run this, you will initially see some garbage output from Twitter4J while it queries for tweets. After that, you can see a list of tweets and their sentiment scores in this format

```
<Tweet> : <Sentiment-score>
```

There is a way to get much better results than what we get now by cleaning up the tweets before sending it the the sentiment analyzer as most of the tweets inherently contains useless data such as usernames, links, hashtags, etc. One way to clean the tweets is by using [this][2] awesome library. You can try it out if you want.

By the way, I am not uploading my project onto GitHub because it is around 400 MB which is quite a lot and also I don&#8217;t want someone stealing my keys :)

Hope this helped you!

 [1]: https://dev.twitter.com/
 [2]: https://github.com/twitter/twitter-text-java
