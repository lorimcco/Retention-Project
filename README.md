# Mobile Game Player Retention Project

### Introduction

This project analyzes data from a mobile game company’s first year. The data set is a relational database that includes four tables:
- Player information: player id, location, age, system, and day joined
- Match information: player id, match id, opponent id, outcome, and day played
- Item information: item id, price
- Purchase information: player id, item id, day purchased

We used these tables to answer two questions about the data:
1. What is the rolling 30-day retention rate by day?
2. How does spending on in-game items of retained players differ from non-retained players?

We used Google BigQuery to run SQL queries and Google Sheets to create charts to display the results.

### Process & Challenges

The metric used to determine if a player was retained was whether they played a match 30 or more days after joining. To create a query to give this information, we joined the Player Info and Match Info tables on player id. We subtracted the day the player joined from their `MAX(match_day)` to get the number of days the player was retained. Then we used `CASE WHEN` to assign a 1 to the players that were retained 30 days or more and a 0 to those who weren’t. This allowed us to use a `SUM` function and aggregate by day joined to find the number of players retained per day, as well as the fractional retention by day: `SUM(retention) / COUNT(player_id)`. We did this in two ways: with a nested query and with a `WITH` clause (see Queries 1a and 1b in the SQLQueries.txt file). The result is a table with four columns: day joined, number of players joined, how many of those were retained, and the fractional retention.

To answer the second question, we wanted a very simple table for our final solution: For retained players vs non-retained players, how much did each group spend on in-game items in total and how much was spent per player on average? To get this table, I used a simplified version of the subquery in the retention table described above that gave the player id and whether they were retained (1) or not (0). I also joined the Item Info and Purchase Info tables on item id to get the total amount each player spent. Then I joined these two newly created tables on player id and found the sum of the amount spent and the average amount spent per player: `SUM(total_spent) / COUNT(player_id)`, which I aggregated by retention to get the overall numbers for the retained vs non-retained players.

Figuring out how to join all the necessary tables in the query to find in-game spending habits was the biggest challenge. We quickly realized that we couldn’t join them all directly because we got many duplicated records and needed to get it down to one row for each player. The key was creating the two separate tables and then joining them. Once again, I challenged myself to do this in two different ways: using two subqueries and using two temporary tables created in a `WITH` clause (see Queries 2a and 2b in the SQLQueries.txt file).

We also noticed an issue when we visualized the data from our rolling 30-day retention table and noticed that the retention rate dropped to zero for the final 30 days of the year. Since we only had data for 365 days, any player who joined after day 335 would automatically be considered non-retained according to our metric, since we didn’t have any match data for 30 days after they joined. This inflated our results for in-game spending by non-retained players. We decided to implement filtering using a `WHERE` clause in all our queries to remove these players from the results entirely.

### Solution and Analysis

While the rolling 30-day retention rate varied between 60% and 84%, the overall retention rate remained stable over the year, averaging around 71% (see table and chart [here](https://docs.google.com/spreadsheets/d/1TcEmIfuYU5jsOtb8fx3eGwZGEiqP4JTSrO84cKI1vY0/edit#gid=183732744)). The chart shows a drop in retention rate for the last few days due to the fact that we don’t have data past day 365. Some of the players who joined on days 333-335 may end up being retained if they play a match in the next few days beyond the end of the first year. The trend of the number of players joining each day is also stable overall, not showing an increasing or decreasing pattern throughout the year.

The total amount spent on in-game items by retained players ($36.5 million) is over twice as much as non-retained players ($16.9 million) (see table and chart [here](https://docs.google.com/spreadsheets/d/1Ym-TaIv49G9PLVmOk8ogKUQLzN2XMrDc6wdZKNkizHk/edit#gid=1727563496)). This was expected, since a majority of players are retained. The most interesting result is that non-retained players spend about $185 more per player than their retained counterparts. This could suggest that there is a pay-to-win mechanic at work in this game: non-retained players may be purchasing many in-game items that make the game too easy, causing them to become quickly bored with the game and quit playing. 

My suggestions for the company depend on their goals. If the company wants to increase retention, they may want to evaluate the in-game items to see if there are some powerful items that are making the game less engaging. Perhaps there are ways to modify some of these items to make a pay-to-win game strategy less viable. However, if the company is primarily concerned with increasing profit, it would be more effective to pursue strategies to attract more new players rather than retaining them for a longer period, since retained players actually spend less than non-retained ones.

[Fractional Retention table and charts](https://docs.google.com/spreadsheets/d/1TcEmIfuYU5jsOtb8fx3eGwZGEiqP4JTSrO84cKI1vY0/edit#gid=183732744)

[Spending by Retention table and charts](https://docs.google.com/spreadsheets/d/1Ym-TaIv49G9PLVmOk8ogKUQLzN2XMrDc6wdZKNkizHk/edit#gid=1727563496)
