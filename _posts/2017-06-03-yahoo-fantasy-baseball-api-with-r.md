---
layout: post
title: "Yahoo Fantasy Baseball API with R"
date: 2017-06-03
tags: [R]
splash_img_source: /assets/images/blogPosts/yahooFantasy_oauth.png
splash_img_caption: Not the final picture 
permalink: "blog/yahoo-fantasy-baseball-api-with-r"
---


Connecting to the Yahoo Fantasy Baseball API is tricky business. If you’ve found this post, you may have also found [this post](http://ryankuhn.net/blog/Fantasy-Football-in-R-Part-I) by Ryan Kuhn, and [this post](http://blog.corynissen.com/2013/12/using-r-to-analyze-yahoo-fantasy.html) by Cory Nissen, both of whom have successfully implemented the API.

I want to build on their work a little bit, by adding some extra functionality and making some generalized functions for connecting to the API. Those functions can be found in Github [here](https://github.com/nequals30/baseball_closer/blob/master/R_func_yahooAPI.R), and an example of using them is [here](https://github.com/nequals30/baseball_closer/blob/master/R_main_baseballCloser.R).

Additionally, these links might be useful:

* [Fantasy API Documentation Page](https://developer.yahoo.com/fantasysports/guide/) (Yahoo)
* [Javascript Node Module for Fantasy API](http://yfantasysandbox.herokuapp.com/) (Luke DeWitt)

So first, a complete example, start to finish. This script will connect to the API and pull down a list of all the relievers in your league:

```R
library(httr)
library(httpuv)
library(XML)

## Enter your information here --------------------------------------------------------
  league_ID <- 12345;
  league_sport <- 'mlb';
  league_year <- 2017;

  # Get Yahoo Credentials:
  # NOTE: You can just paste yours in here instead of storing them in a file like me.
  cKey     <- readLines("myPrivateKey.txt")[1];
  cSecret  <- readLines("myPrivateKey.txt")[2];
  
## Authenticate Yahoo API credentials -------------------------------------------------

  # If our token is already saved, use it. Otherwise, get it again.
  yahoo_token <- tryCatch({
    load("yahoo_token.Rdata");
    yahoo_token;
    
  }, error <- function(e){
    yahoo_token <- yahooFantasy_get_oauth_token(cKey,cSecret);
    return(yahoo_token);
  })
  
## Figure out the correct Game_ID and league Key ---------------------------------------
  
  game_ID <- yahooFantasy_get_gameID(league_sport,league_year,yahoo_token);
  leagueKey <- paste0(game_ID,'.l.',league_ID);
  
  allRelievers <- yahooFantasy_get_allRelievers(leagueKey,yahoo_token);

```

The final product is a data frame which contains all the relievers in your league, and their associated player data in Yahoo:

![Data Frame with all RPs](/assets/images/blogPosts/yahooFantasy_rpDf.png)

### 1. Obtaining Credentials

This part is really easy. Go [here](https://developer.yahoo.com/apps/create/) and register a new application. Pick “Installed Application” at the top, and check “Fantasy Sports” at the bottom.

It will give you a consumer key and a secret key. I saved mine to a txt file but you can hardcode them into your R script.

### 2. Connecting with Oauth

This is where things get a bit trickier. Ryan Kuhn’s [version](http://ryankuhn.net/blog/Fantasy-Football-in-R-Part-I) tells you to use Oauth2.0, but then you will quickly end up in [this conundrum](https://stackoverflow.com/questions/36893652/integrating-yahoo-sports-api-data) where both Yahoo and R are asking for a 7-digit code.

Instead, we will use Oauth1.0, found in the httr library:

```R
yahooFantasy_get_oauth_token <- function(cKey,cSecret) {
  # Creates Oauth Token for connecting to Yahoo API and saves it.
  #
  # Inputs:
  #   cKey       Credential Key (register here: https://developer.yahoo.com/apps/create)
  #   cSecret    Credential Secret
  #
  # Outputs:
  #   yahoo_token

    yahoo <- oauth_endpoints("yahoo");
    
    myapp <- oauth_app("yahoo", key=cKey, secret=cSecret);
    yahoo_token<- oauth1.0_token(yahoo, myapp, cache=FALSE);
    
    save(yahoo_token,file="yahoo_token.Rdata");
    return(yahoo_token);
}
```

If you run this function with the keys yahoo provided you, you should end up at a screen that looks like this:

![Yahoo Fantasy Oauth](/assets/images/blogPosts/yahooFantasy_oauth.png)

Just click through; everything should be pretty simple. Note: I am also saving off a copy of the token, so that you don’t need to do this every time.

### 3. Figuring out your League Key

To identify your league, you need a _League Key_, like: 370.l.12345. There are three parts to this:

1. __Game ID__: A number that represents game and season. So, MLB 2017 is 370.
2. The lowercase letter “L”, for “league”.
3. __League ID__: The number that identifies your league. You can find that at the top of your league settings or in the URL

To get your game ID from Yahoo, you can use this function:

```R
yahooFantasy_get_gameID <- function(sport,year,yahoo_token){
  # Ask Yahoo for the 3-digit game_ID based on the sport and season.
  #
  # Inputs:
  #   sport: For example, 'mlb' or 'nfl'
  #   year:  For example, 2017
  #   yahoo_token:  yahoo token obtained from yahooFantasy_get_oauth_token
  #
  # Outputs:
  #   game_ID: A string with a 3 digit number
  
  thisUrl <- paste0("http://fantasysports.yahooapis.com/fantasy/v2/game/",sport,'?season=',year);
  thisXml <- yahooFantasy_query(thisUrl,yahoo_token);
  game_ID <- xmlValue(thisXml[["game"]][["game_key"]]);
  return(game_ID)
}
```

### 4. Querying the API

You’ll notice that I abstracted a function to do the queries. It queries the API, brings in the XML, and returns the root XML node which can be manipulated (such as with the `xmlToDataFrame` function).

```R
yahooFantasy_query <- function(inUrl,yahoo_token){
  # Internal helper function to query Yahoo for the data.
  #
  # Inputs:
  #   inUrl         String URL
  #   yahoo_token   yahoo_token produced by yahooFantasy_get_oauth_token
  #
  # Outputs: 
  #   outNode       xml node object which contains query results
  
  page <-GET(inUrl,config(token=yahoo_token));
  out <- content(page, as="text", encoding="utf-8");
  
  outXml<-xmlTreeParse(out,useInternal=TRUE);
  outNode <- xmlRoot(outXml);
  
  return(outNode)
}
```

Note: it wasn’t immediately obvious to me how to append more than one filter to the end of the URL. The [Yahoo Documentation](https://developer.yahoo.com/fantasysports/guide/#players-collection) seems to imply that the URL is `/players;position=RP;status=A` to get a list of all the active Relief Pitchers.

In our case, though, the right way to construct the URL is: `/players?&position=RP&status=A`.

Obviously, there is a lot more that this API can do, and I will spend some time in the future trying to build out some better tools for connecting with the API. For now, I hope that this little guide provides the basic building blocks for someone trying to implement it.

I’ll keep working on this; if you are interested, follow me on [twitter](https://twitter.com/nequals30).

