### Query 1a: Retention Table using nested queries

```sql
--Finds number of players joined, number of players retained for 30 days, and fractional retention per day, using nested queries.

SELECT
    day_joined,
    COUNT(player_id) AS players_joined, --total players joined per day
    SUM(retention) AS players_retained, --number of players retained per day
    (SUM(retention) / COUNT(player_id)) AS fractional_retention --percentage of players retained
FROM 
    --Subquery: Finds the day joined, the day of the last match played, and whether the player was retained (1) or not (0)
    (SELECT 
        p.player_id AS player_id,
        p.joined AS day_joined,
        MAX(m.day) AS most_recent_match, --gives day of each player's latest match
        CASE 
            WHEN MAX(m.day) - p.joined >= 30 THEN 1 --If the most recent match is 30 or more days after the day the player joined, player is retained
            ELSE 0 --If the most recent match is less than 30 days after the player joined, player is not retained
            END AS retention
    FROM `first-project-329514.game_company_data.player_info` p
    JOIN `first-project-329514.game_company_data.matches_info` m
        ON p.player_id = m.player_id
    WHERE 
        p.joined <= 335 --No data exists for more than 30 days after day 335, so can't determine whether a player is retained or not if they joined after day 335 
    GROUP BY 
        p.player_id,
        p.joined)
GROUP BY 
    day_joined --aggregate by day joined
ORDER BY 
    day_joined ASC
```

### Query 1b: Retention Table using WITH clause

```sql
--Finds number of players joined, number of players retained for 30 days, and fractional retention per day, using a WITH clause.

WITH retention AS
    --Finds the day joined, the day of the last match played, and whether the player was retained (1) or not (0)
    (SELECT 
        p.player_id,
        p.joined,
        MAX(m.day) AS most_recent_match, --gives day of each player's latest match
        CASE 
            WHEN MAX(m.day) - p.joined >= 30 THEN 1 --If the most recent match is 30 or more days after the day the player joined, player is retained
            ELSE 0 --If the most recent match is less than 30 days after the player joined, player is not retained
            END AS player_retention
    FROM `first-project-329514.game_company_data.player_info` p
    JOIN `first-project-329514.game_company_data.matches_info` m
        ON p.player_id = m.player_id
    WHERE 
        p.joined <= 335 --No data exists for more than 30 days after day 335, so can't determine whether a player is retained or not if they joined after day 335 
    GROUP BY 
        p.player_id,
        p.joined)

SELECT
    retention.joined AS day_joined,
    COUNT(p.player_id) AS players_joined, --total players joined per day
    SUM(retention.player_retention) AS players_retained, --number of players retained per day
    SUM(retention.player_retention) / COUNT(p.player_id) AS fractional_retention --percentage of players retained
FROM retention 
JOIN `first-project-329514.game_company_data.player_info` p
    ON retention.player_id = p.player_id
GROUP BY 
    day_joined --aggregate by day joined
ORDER BY 
    day_joined ASC
```

### Query 2a: In-Game Spending by Retention using nested queries

```sql
--Finds the total spent and average amount spent per player for retained and non-retained players, using nested queries

SELECT 
    r.retention,
    ROUND(SUM(s.total_spent) / 1000000, 1) AS total_spent_millions, --total spent by all players
    ROUND(SUM(s.total_spent) / COUNT(s.player_id), 2) AS avg_spent_per_player --average spent by each player
FROM 
    --Subquery: Gives retention (0 or 1) for each player
    (SELECT 
        p.player_id,
        CASE 
            WHEN MAX(m.day) - p.joined >= 30 THEN 1 --If the most recent match is 30 or more days after the day the player joined, player is retained
            ELSE 0 --If the most recent match is less than 30 days after the player joined, player is not retained
        END AS retention
    FROM `first-project-329514.game_company_data.player_info` p
    JOIN `first-project-329514.game_company_data.matches_info` m
        ON p.player_id = m.player_id
    WHERE p.joined <= 335 --No data exists for more than 30 days after day 335, so can't determine whether a player is retained or not if they joined after day 335 
    GROUP BY 
        p.player_id,
        p.joined
    ) r
JOIN --join the two subqueries
    --Subquery: Gives the total amount spent by each player
    (SELECT 
        pu.player_id,
        SUM(i.price) AS total_spent --sum of all items bought by player
    FROM `first-project-329514.game_company_data.item_info` i
    JOIN `first-project-329514.game_company_data.purchase_info` pu
        ON i.item_id = pu.item_id
    GROUP BY 
        pu.player_id
    ) s
ON r.player_id = s.player_id
GROUP BY 
    r.retention --aggregate by whether the player was retained (1) or not (0)
ORDER BY 
    r.retention DESC
```

### Query 2b: In-Game Spending by Retention using WITH clause

```sql
--Finds the total spent and average amount spent per player for retained and non-retained players, using two temporary tables in a WITH clause

WITH retention AS 
    --Gives retention (0 or 1) for each player
    (SELECT
        p.player_id,
        CASE 
            WHEN MAX(m.day) - p.joined >= 30 THEN 1 --If the most recent match is 30 or more days after the day the player joined, player is retained
            ELSE 0 --If the most recent match is less than 30 days after the player joined, player is not retained
        END AS player_retention,
    FROM `first-project-329514.game_company_data.player_info` p
    JOIN `first-project-329514.game_company_data.matches_info` m 
        ON p.player_id = m.player_id
    WHERE p.joined <= 335 --No data exists for more than 30 days after day 335, so can't determine whether a player is retained or not if they joined after day 335 
    GROUP BY
        p.player_id,
        p.joined
     ),

    player_spending AS 
    --Gives the total amount spent by each player
    (SELECT
        pu.player_id,
        SUM(i.price) AS total_spent, --sum of all items bought by player
    FROM `first-project-329514.game_company_data.item_info` i
    JOIN `first-project-329514.game_company_data.purchase_info` pu
        ON i.item_id = pu.item_id
    GROUP BY
        pu.player_id
     )

SELECT
    retention.player_retention,
    ROUND(SUM(player_spending.total_spent)/1000000,1) AS grand_total_millions, --total spent by all players
    ROUND(SUM(player_spending.total_spent) / COUNT(player_spending.player_id), 2) AS avg_spent_per_player --average spent by each player
FROM retention --first table in WITH clause
JOIN player_spending --second table in WITH clause
ON retention.player_id = player_spending.player_id
GROUP BY 
    player_retention --aggregate by whether the player was retained (1) or not (0)
ORDER BY 
    player_retention DESC
```
