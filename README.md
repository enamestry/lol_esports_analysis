# League of Legends Stat Difference Implications

## Introduction
The dataset I chose to analyze for this project was League of Legends Esports Match Data for the year of 2024. All data was sourced from Oracle's Elixir (https://oracleselixir.com/about).

The dataset contains 117600 rows of player & team data corresponding to 9800 matches. For each match, it has 10 player rows, each filled with individual player performance data collected at various points throughout the game. Each match also has 2 team rows for each team participating, with amalgamated data of the whole team's performance. 

The question I wanted to investigate further was about stat differences over time- would it become more pronounced or would teams be able to close the gap as game length increased? Additionally, was the higher or lower gold a large factor in whether the team would win or not? I was especially curious if it would not matter as much in the later game after everyone was able to get fully geared up- at that point it would just be down to skill, which is fairly evenly matched among pro players.

To answer my question, I first looked in the data to find columns that I thought would be relevant to my question about stat differences. I ended up deciding to mainly focus on the following:
* "gamelength" - the length of the match, in seconds
* "result" - true/false, true if the player/team won the match
* "totalgold" - total gained gold of each player/team throughout the whole match
* "earnedgold" - totalgold but without passive gold gain over time
* "goldspent" - total gold spent by each player/team throughout the whole match
* "goldat10", "xpat10" - assorted game stats (gold, experience) measured at 10 minutes into the match
* "golddiffat10", "xpdiffat10" - differences in friendly team/enemy teams stats at 10 minutes into the match

I wanted to analyze all I could about the size of stat differences over time as well as how much they affected game outcome- and see how difficult it would be to accurately predict game result based on those stat differences over time (how early in the game could the result be predicted?).


## Data Cleaning and Exploratory Data Analysis
#### Data Cleaning
For my data cleaning, I approximately followed these steps:

1. Remove matches with incomplete data from loldata.

2. Remove columns from loldata that were irrelevant or blank. 

3. Change type of certain columns to booleans.

4. Remove notable outliers with missing late-game data that is not easily imputable.

5. Split loldata into data for individual players and data for teams.

5. Repeat step 2 with each of the smaller separated dataframes.

#### Univariate Analysis

I decided to first look at the distributions of several of the variables given. I focused on numerical variables as most categorical variables were evenly distributed given that there are the same number of each of the roles and sides in each match. First I looked at the "gamelength variable" from the teamdata, where I found it to be fairly normally distributed but with several outliers. My second analysis was of "goldat10", which is the amount of gold that each player had after the first 10 minutes of the game. I found that this one had two peaks, with one significantly lower than the other. I believed this to be due to player roles, where some roles like support typically have less gold than other positions. This inspired some of my later analysis.

 <iframe
 src="assets/fig_gl.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

 <iframe
 src="assets/fig_g.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>


#### Bivariate Analysis

I also wanted to conduct bivariate analysis to see how some of the relevant variables interacted with each other. The first relationships I wanted to look at were the gold and xp over time for winning vs losing players, which are pictured in the first and second graphs below. They were about what I expected, but it was difficult to take away more than the expected basic relationship from these graphs. Next, I wanted to reinvestigate the strange multimodal distribution for player gold at 10 minutes, so I separated it out by player role. I was able to find that supports had significantly lower gold on average than the rest of the roles, which likely accounted for the smaller curve in the univariate graph.

 <iframe
 src="assets/fig_got.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

 <iframe
 src="assets/fig_xpot.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>

 <iframe
 src="assets/fig_posvsgold.html"
 width="800"
 height="600"
 frameborder="0"
 ></iframe>


#### Interesting Aggregates

When looking at aggregates I strayed a bit from my initial question because there were so many statistics I was curious about! The three that I wanted to higlight were the following:
* Champion win rate - As we can see, some champions have almost abnormally high win rates, while others haven't managed to win even once.
<iframe src="assets/agg1.html" width="800" height="400" frameborder="0">
</iframe>

* Total number of wins of each team - "unknown team" takes the top on this one, but is really an amalgamation of all unknown teams. If we disregard this, Gen.G has the highest amount of wins in 2024 with an impressive 101 and are the only team to have higher than 100 wins.
<iframe src="assets/agg2.html" width="800" height="400" frameborder="0">
</iframe>

* Number of teams played on in 2024 by each player - I was honestly just curious about how frequently players switched teams. From what I understand, there's offseason and then competition season each year with several competitions, and you might expect 1-3 team switches, maybe one after each tournament if you're really a hot commodity. I was shocked to see Black come up with a whopping 8 teams played for in just one year.
<iframe src="assets/agg3.html" width="800" height="400" frameborder="0">
</iframe>

#### Imputation (or lack thereof)
When going through the data, I noticed that the variation in game length meant that many of the shorter games were missing data for later-game metrics such as "goldat25". Since I wanted to focus on the gold/xp/cs metrics at different points in games, I would in theory need to impute values for these games. However, I ultimately decided not to impute these values as it would be very difficult to estimate values that would still garner meaningful results from a model trained with them. Even if I took the values from previous measured points in time, those would be significantly lower than non-imputed values measured at the actual specified time. I instead did not include later measurements in my model training and focused on more universal metrics.


## Framing a Prediction Problem
The prediction problem I chose was to predict game result for teams based on their combined in-game statistics. Initally, I wanted to try classifying based on individual player data, but this garnered very poor results as League of Legends is an extremely team-based game and it is difficult to get a good understanding of the game state by looking at one player's stats isolated.

Even though many of the utilized statistics would not be available before the end of the game, where we would already know match result, I still wanted to try predicting with them. I wanted to see how well the model could do with mostly just stat differences, which would indirectly tell me how much these stat differences actually influenced game outcome. 

## Baseline Model
For my baseline model, I decided to use a simple LogisticRegression model with the following variables:
* side (nominal)
* earnedgold (quantatative)
* goldspent (quantatative)
* goldat10 (quantatative)

The "side", "earnedgold", and "goldspent" features all gave a more overarching perspective of the game, while the "goldat10" feature gave a better idea of if a game started out very biased with one team getting a big lead that was likely to snowball later in the game.

The "side" variable needed to be OneHotEncoded as it is a categorical variable, so I did this first, creating a column transformer to apply the transformation and using the argument remainder="passthrough" to ensure that all the non-transformed columns were still used for model fitting.

To quantify performance, I used the train and test accuracy rather than a more complicated metric such as F1-score, which is often needed when there is an imbalance of data. However, I know that there will be an exactly equal amount of win data vs loss data due to how the dataset is constructed, so skewed sample distribution is not a concern. Accuracy is a simple statistic which fits well for our current goals.

With an accuracy score of ~0.8 on both train and test data, I would say that my model is currently fairly decent, but nothing too special. It does have final metrics that wouldn't be accessible mid-game that could be accounting for the already fine performance. One thing that I would note as being a very good sign is that it has minimal difference between its performance on train and test data, which indicates that there is likely not too much overfitting to the training data.

## Final Model

For my final model, I added the following features in addition to the ones in my baseline model:

* xpat10 (quantative)
* golddiffat10 (quantative)
* xpdiffat10 (quantative)
* gamelength (quantative)

The "xpat10" column gives additional information about the overall team's performance and can be very influential for scaling early-game. For example, if a player hits level 6 before their opponent, they will have unlocked their ultimate ability and have significant power over their opponent until they also reach level 6 and unlock their ult. I then also decided to add "golddiffat10" and "xpdiffat10" to give the numbers more meaning as to how they compared with their opponents and if they were outscaling them or being outscaled. I additionally added the "gamelength" variable so that I would be able to do some column transformations for overall game stats, standardizing them based on game length.

I was also initially planning to include "csat10" and "csdiffat10", but then I realized that this will likely have a very strong correlation with the gold and xp columns since cs is the primary method of gaining gold and xp. This would then introduce multicollinearity within the used data and hinder performance. 

In my transformer, I added several more modifications to my data. First, I created a FunctionTransformer to standardize the "earnedgold" and "goldspent" variables according to the total gamelength, which would significantly affect these stats and put them on different scales when looking at shorter vs longer games. I also added a StandardScaler for the "goldat10" and "xpat10" variables to make them a more reasonable size and also help reduce the strain on the model being fit.

Speaking of the model, I switched from using a LogisticRegression model to using a RidgeClassifier model in the end. I also used GridSearchCV to select the optimal value of alpha for the ridge classifier, which would help minimize any overfitting. 

In the end, my accuracy went up by nearly 3% with the minimal changes I made. I'm very happy with these results and seeing an increase without adding the other extremely large amounts of data within the dataset. With a final training accuracy of ~0.831 and a final testing accuracy of 0.827, I can safely say that there is no strong overfitting and the results are predicted with a fairly high accuracy. It isn't in the 90% range of accuracy, which is expected due to several other aspects of the game being left out entirely. Even while leaving these out, I was able to achieve my final goal, which was to judge how strong of an effect just stat differences had on the output of the game, and I found that you can predict the final result reliably 80% of the time just based off of them.