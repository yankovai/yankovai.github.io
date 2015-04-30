---
layout: post
title: Network Graphs and NCAA Basketball
---

The 2015 NCAA basketball tournament is long over but who cares? Let's face it, us data scientists are more interested in the numbers surrounding the tournament than the actual game itself. In this sense, the tournament is just a venue for applying our sweet tools and algorithms. In this post I'm going to demonstrate the power of network graphs for data mining. Networks graphs are awesome because they can be used to model so many things. Anything that can be described using nodes and edges can be beaten into something insightful using the suite of network graph tools, metrics, and algorithms. Usually, network graph stuff is demonstrated on problems like the [Karate Club](https://networkdata.ics.uci.edu/data.php?id=105). Personally, I stopped caring about karate after Bruce Lee died. I reckon analyzing some college basketball data will be much more exciting. In case you were wondering, the reason I decided to write this post is because I'm on the bus from Seattle to Vancouver in hopes of making the [SIAM Conference on Data Mining](http://www.siam.org/meetings/sdm15/) and I need time to move a little faster.  

Let's assume all you know about college basketball is that it consists of these things called games where two different teams play each other. Also, you know there is a college basketball season consisting of a whole schedule of these games. We need some data. Fortunately some guy has [already compiled](https://www.spreadsheet-sports.com/2015-ncaa-basketball-game-data) all the games during the 2015 college basketball season in an easily digestible format. I am forever grateful to the person who owns that page, and would very much like to high-five him, since now I don't have to build a scraper to extract the schedule from [Sports Reference](http://www.sports-reference.com/cbb/). We're going to mine the college basketball schedule to see if we can learn anything about college basketball that we didn't already know. To achieve this monumental task we'll use Python of course, and the [NetworkX](https://networkx.github.io) library, which in my opinion is a brilliant piece of work for network graph analysis. 

To start our analysis let's import everything we'll need:
```python
import pandas as pd
import numpy as np
import networkx as nx
import matplotlib.pylab as plt
from scipy.stats import gaussian_kde
```



what we're modeling (teams as nodes and if team A plays team B then we'll draw an edge between team A and team B. If team A plays team B more than once, we'll take this information into account by adding weight to the edge.)
the data
 
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Date</th>
      <th>Team</th>
      <th>Team Location</th>
      <th>Team Score</th>
      <th>Opponent</th>
      <th>Opponent Score</th>
      <th>Opponent Location</th>
      <th>Neutral Site</th>
      <th>Team Result</th>
      <th>Team Differential</th>
      <th>Opponent Differential</th>
      <th>Game Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>2</th>
      <td>2014-11-14</td>
      <td> Valparaiso</td>
      <td> Home</td>
      <td> 90</td>
      <td>          ETSU</td>
      <td> 76</td>
      <td> Away</td>
      <td> NaN</td>
      <td>  Win</td>
      <td> 10.545455</td>
      <td>  2.033333</td>
      <td> Division 1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2014-11-14</td>
      <td>       ETSU</td>
      <td> Away</td>
      <td> 76</td>
      <td>    Valparaiso</td>
      <td> 90</td>
      <td> Home</td>
      <td> NaN</td>
      <td> Loss</td>
      <td>  2.033333</td>
      <td> 10.545455</td>
      <td> Division 1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2014-11-14</td>
      <td> Texas Tech</td>
      <td> Home</td>
      <td> 71</td>
      <td>     Loyola MD</td>
      <td> 59</td>
      <td> Away</td>
      <td> NaN</td>
      <td>  Win</td>
      <td> -3.437500</td>
      <td> -7.366667</td>
      <td> Division 1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2014-11-14</td>
      <td>  Loyola MD</td>
      <td> Away</td>
      <td> 59</td>
      <td>    Texas Tech</td>
      <td> 71</td>
      <td> Home</td>
      <td> NaN</td>
      <td> Loss</td>
      <td> -7.366667</td>
      <td> -3.437500</td>
      <td> Division 1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2014-11-14</td>
      <td>  Kansas St</td>
      <td> Home</td>
      <td> 98</td>
      <td> Southern Utah</td>
      <td> 68</td>
      <td> Away</td>
      <td> NaN</td>
      <td>  Win</td>
      <td> -0.812500</td>
      <td> -4.655172</td>
      <td> Division 1</td>
    </tr>
  </tbody>
</table>

I would recommend not starting out by plotting all the nodes and edges in your data set because most often you'll just get a ball of spaghetti to look at.  

![Ego-centric graphs of 10 random NCAA basketball teams.](/assets/ego_centric_network_sample.png)
