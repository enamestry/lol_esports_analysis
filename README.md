# League of Legends Esports Match Data Analysis
This is a project for the University of Michigan course EECS 398: Practical Data Science.

## Introduction
The dataset I chose to analyze for this project was League of Legends Esports Match Data for the year of 2024.

For its formatting, I noticed some important things while going through the data:

1. The data for some of the matches was incomplete. Thankfully, this was denoted through the "datacompleteness" column, which made it easy to filter out incomplete data (a relatively small portion was incomplete and even after removing I still had a lot of data to work off of).

2. There were rows for each player in a match, but there were also rows for each team's amalgamated performance in each match. To keep the type and scale of the data consistent, I would need to separate the player data from the team data to create two separate dataframes.

3. Several columns would need to be removed for one reason or another. There were a few columns that were irrelevant or completely blank, which I would need to remove from the whole dataset. Then, there were columns that either didn't have any data for or didn't make sense in the context of individual players or team statistics, so I would need to remove those in the appropriate dataframes.

The question I wanted to investigate further was about stat differences over time- would it become more pronounced or would teams be able to close the gap. Additionally, was the higher or lower gold a large factor in whether the team would win or not? I was especially curious if it would not matter as much later game after everyone was able to get fully geared up- at that point it would just be down to skill, which is fairly evenly matched among pro players.

## Data Cleaning and Exploratory Data Analysis


## Framing a Prediction Problem


## Baseline Model


## Final Model

