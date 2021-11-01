```SQL
SELECT -- counting rolling fractional retention
    first_day,
    total_number_of_players,
    number_of_retained,
    fractional_retention,
    ROUND(SAFE_DIVIDE(previous_day_retention-fractional_retention,previous_day_retention)* 100,2) AS rolling_retention_rate
FROM(    
    SELECT -- creating previous day retention rate
        first_day,
        total_number_of_players,
        number_of_retained,
        fractional_retention,
        Lag(fractional_retention,1) OVER (ORDER BY first_day) AS previous_day_retention         
    FROM(
SELECT --counting fractional retention, total number of players, number of retained players
  first_day,
  COUNT(player_id) AS total_number_of_players,
  SUM(retaintion) AS number_of_retained,
  ROUND(SUM(retaintion)/COUNT(player_id) * 100, 2) AS fractional_retention
FROM (
    SELECT --define if each player played more or less than 30days
      player_id,
      first_day,             
    CASE WHEN match_day - first_day > 30 THEN 1
      ELSE 0
   END AS retaintion  
  FROM (
    SELECT --count what was the day players joined the game and when was the last match played
      p.player_id,
      MIN(p.joined) AS first_day,
      MAX(m.day) AS match_day
    FROM
      `my-project-99345-329514.gamecompanydata.matches_info` AS m
    INNER JOIN
      `my-project-99345-329514.gamecompanydata.player_info` AS p
    ON
      m.player_id = p.player_id
    GROUP BY 1) 
GROUP BY  1,  2, 3)
GROUP BY 1)
GROUP BY 1,2,3,4)
GROUP BY 1,2,3,4,5
ORDER BY 1
```

-- Do players with rolling 30-day retention spend more?
-- Do players with rolling 30-day retention come from specific regions?

```SQL
WITH total_s AS -- creating a temporary table to count how much each player spent in the game
        (SELECT 
                prc.player_id AS player_id,                 
                ROUND(SUM(i.price), 2) AS total_spent
            FROM `my-project-99345-329514.gamecompanydata.purchase_info` AS prc
                JOIN `my-project-99345-329514.gamecompanydata.item_info` AS i
                ON prc.item_id = i.item_id            
    GROUP BY 1),

    retention_rate_by_player AS --creating a temporary table with retention rate for each player, select location and age for each player
        (SELECT  --define if each player played more or less than 30days              
        player_id,
        location,
        age,
	(match_day-first_day) AS days_played,            
        CASE 
            WHEN (match_day - first_day) > 30 THEN '1'
            ELSE '0'
            END AS retained
        FROM(
            SELECT --count what was the day they joined the game and when was the last match played     
                p.player_id AS player_id,
                p.location,  
                p.age,            
                MIN(p.joined) AS first_day,
                MAX(m.day) AS match_day
            FROM `my-project-99345-329514.gamecompanydata.matches_info` AS m
                JOIN `my-project-99345-329514.gamecompanydata.player_info` AS p
                ON m.player_id = p.player_id
            GROUP BY 1,2,3)
            GROUP BY 1,2,3,4,5)

SELECT player_id, location, age, days_played, retained, total_spent  --displaying the column from each temporary table
FROM retention_rate_by_player
LEFT JOIN total_s USING (player_id)
GROUP BY 1,2,3,4,5
ORDER BY 5 DESC
```
