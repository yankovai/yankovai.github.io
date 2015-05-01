---
layout: post
title: Network Graphs and NCAA Basketball
---

The 2015 NCAA basketball tournament is long over but who cares? Let's face it, us data scientists are more interested in the numbers surrounding the tournament than the actual game itself. In this sense, the tournament is just a venue for applying our sweet tools and algorithms. In this post I'm going to demonstrate the power of network graphs for data mining. Networks graphs are awesome because they can be used to model so many things. Anything that can be described using nodes and edges can be beaten into something insightful using the suite of network graph tools, metrics, and algorithms. Usually, network graph stuff is demonstrated on problems like the [Karate Club](https://networkdata.ics.uci.edu/data.php?id=105). Personally, I stopped caring about karate after Bruce Lee died. I reckon analyzing some college basketball data will be much more exciting. In case you were wondering, the reason I decided to write this post is because I'm on the bus from Seattle to Vancouver in hopes of making the [SIAM Conference on Data Mining](http://www.siam.org/meetings/sdm15/) and so I have a few hours to kill.  

Let's assume all you know about college basketball is that it consists of these things called games where two different teams play each other. Also, you know there is a college basketball season consisting of a whole schedule of these games. We need some data. Fortunately some guy has [already compiled](https://www.spreadsheet-sports.com/2015-ncaa-basketball-game-data) all the games during the 2015 college basketball season in an easily digestible format. I am forever grateful to the person who owns that page, and would very much like to high-five him, since now I don't have to build a scraper to extract the schedule from [Sports Reference](http://www.sports-reference.com/cbb/). We're going to mine the college basketball schedule to see if we can learn anything about college basketball that we didn't already know. To achieve this monumental task we'll use Python of course, and the [NetworkX](https://networkx.github.io) library, which in my opinion is a brilliant piece of work for network graph analysis. 

To start our analysis let's import everything we'll need and go ahead and download the [basketball schedule](https://www.spreadsheet-sports.com/2015-ncaa-basketball-game-data):
{% highlight python %}
import pandas as pd
import numpy as np
import networkx as nx
import matplotlib.pylab as plt
from scipy.stats import gaussian_kde
{% endhighlight %}

Take a peak inside the schedule data, which is formatted as an excel spreadsheet, using Pandas. I have stored the excel file in a directory called 'file_name'. 
{% highlight python %}
df = pd.read_excel(file_path)
df = df.loc[df['Game Type'] == 'Division 1']
df.head()
{% endhighlight %}
I'm not going to show you the results of the code above because it outputs an ugly looking table that can't fit inside this blog horizontally. I'm only interested in games between two divsion I teams so I query the data for games matching this criterion. As a good data scientist, you always inspect the data first, right? Well if you inspect the data you'll find that there are duplicate entries for each game. Keep this in mind. The way we're going to analyze this schedule using network graphs is by treating teams as nodes. If team A plays team B then we'll draw an edge between team A and team B. If team A plays team B more than once, we'll take this information into account by adding weight to the edge. Splendid.

Now, we have to get this raw schedule data into a format NetworkX can understand. I like to use the from_dict_of_dicts methods described [here](http://networkx.lanl.gov/archive/networkx-1.5/reference/generated/networkx.convert.from_dict_of_dicts.html#networkx.convert.from_dict_of_dicts). We've got to convert the raw schedule spreadsheet into this dictionary of dictionaries object. Here's one way to do it:

{% highlight python %}
# Mold data into dictionary of dictionary format accepted by networkx
dod = dict()
for team in df.Team.unique():
    dod.setdefault(team, {})
    opponents = df.loc[df.Team == team, 'Opponent'].values
    
    # Filter some opponents so they are not double counted
    opponents = [opponent for opponent in opponents if opponent < team]
    
    for opponent in opponents:    
        try:
            dod[team][opponent]['weight'] += 1
        except KeyError:
            dod[team][opponent] = {'weight': 1}
{% endhighlight %}

The weight key between two teams in this case corresponds to the number of times the two teams played each other. However, in a lot of network graph problems, like finding the shortest distance between any two nodes, a larger weight implies a longer distance to traverse. Let's stop and think what it might mean to travel from node "Michigan" to node "Princeton". Perhaps Michigan is going to face off against Princeton and wants to get the inside scoop on how to beat the mighty Tigers by watching game tape. Well, Michigan played Syracuse once who played Cornell once who played Princeton twice. That's a total of 4 matches to watch. We also know that Michigan played Illinois 3 times who played Brown once who played Princeton twice for a total of 6 matches to watch. We want the "shortest" path from Team A to team B to be the one that has the most matches played. The inverse of edge weight will give us the desired behavior. We add the inverse weight as an edge attribute to the dictionary of dictionaries object:

{% highlight python %}            
# Calculate inverse weight for distance measures
for team, opponents in dod.iteritems():
    for opponent in opponents:
        dod[team][opponent]['inv_weight'] = 1./dod[team][opponent]['weight']
{% endhighlight %}

The dictionary of dictionaries object is now complete. Let's create an undirected graph in NetworkX!
{% highlight python %}   
g = nx.from_dict_of_dicts(dod)
{% endhighlight %}

Your first instinct as a data scientist may be to plot the network graph we created. I would strongly advise you to restrain yourself from doing this initially for network graph analysis unless you enjoy looking at something resembling a ball of spaghetti. Rather, you should look at some other plots to give you a sense of the type of beast you'll be dealing with. First, go ahead and plot the degree distribution of the nodes. This should give you a sense of whether or not some edges may be filtered out. Often times in network graph problems there are a lot of spurious edges that can safely be removed. In this case the degree of a node is the number of unique teams played throughout the regular season.  
![Degree distribution of NCAA basketball teams.](/assets/bball_schedule_degree_distribution.png)
Looks like most teams play some 21 unique opponents on average. The degree distribution is neither exponentially distributed nor is it normally distributed. For an epic read on degree distributions in huge networks you should check out the book Linked by Albert-laszlo Barabasi. Don't be intimidated by the author's double-pronged name, it's a popular science book like Stephen Hawking's The Universe in a Nutshell!  


<!-- ![Ego-centric graphs of 10 random NCAA basketball teams.](/assets/ego_centric_network_sample.png) -->
