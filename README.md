-- Number of olympic games held so far and a list of all games with their host cities.

SELECT COUNT(DISTINCT games) AS total_olympic_games
    FROM `Olympic_games.olympic_history`;

SELECT DISTINCT year, season, city
    FROM `Olympic_games.olympic_history`
    ORDER BY year;

-- Fetch the total no of sports played in each olympic games

WITH table_1 AS
      	(SELECT DISTINCT games, sport
      	FROM `Olympic_games.olympic_history`),
        table_2 AS
      	(SELECT games, COUNT(1) AS no_of_sports
      	FROM table_1
      	GROUP BY games)
      SELECT * FROM table_2
	  ORDER BY no_of_sports DESC;

-- sport which were played once and which Olympic games it happened

WITH t1 AS
          	(SELECT DISTINCT games, sport
          	FROM `Olympic_games.olympic_history`),
          t2 as
          	(SELECT sport, COUNT(1) AS no_of_games
          	FROM t1
          	GROUP BY sport)
      SELECT t2.*, t1.games
      FROM t2
      JOIN t1 ON t1.sport = t2.sport
      WHERE t2.no_of_games = 1
      ORDER BY t1.sport;

-- Top 5 athletes who have won the most gold medals.

WITH t1 AS
            (SELECT name, team, COUNT(1) AS total_gold_medals
            FROM `Olympic_games.olympic_history`
            WHERE medal = 'Gold'
            GROUP BY name, team
            ORDER BY total_gold_medals DESC),
        t2 AS
            (SELECT *, DENSE_RANK() OVER (ORDER BY total_gold_medals DESC) AS rnk
            FROM t1)
    SELECT name, team, total_gold_medals
    FROM t2
    WHERE rnk <= 5;

-- oldest athletes to win a gold medal

WITH TEMP AS
            (SELECT name,sex,CAST(CASE WHEN age = 'NA' THEN '0' ELSE age END AS int) AS age
              ,team,games,city,sport, event, medal
            FROM `Olympic_games.olympic_history`),
        ranking AS
            (SELECT *, RANK() OVER(ORDER BY age desc) AS rnk
            FROM TEMP
            WHERE medal='Gold')
    SELECT *
    FROM ranking
    WHERE rnk = 1;

-- Which nation has participated in most of the olympic games

SELECT COUNT(DISTINCT games) AS no_of_years, team
FROM `Olympic_games.olympic_history`
GROUP BY team
ORDER BY no_of_years DESC;


-- list of all sports which have been part of every olympics.

WITH t1 AS
          	(SELECT COUNT(DISTINCT games) AS total_games
          	FROM `Olympic_games.olympic_history` WHERE season = 'Summer'),
          t2 as
          	(SELECT DISTINCT games, sport
          	FROM `Olympic_games.olympic_history` WHERE season = 'Summer'),
          t3 AS
          	(SELECT sport, COUNT(1) AS no_of_games
          	FROM t2
          	GROUP BY sport)
      SELECT *
      FROM t3
      JOIN t1 ON t1.total_games = t3.no_of_games;

--Top athletes who have won most medals (Medals include gold, silver and bronze).

WITH t1 AS
            (SELECT name, team, COUNT(1) AS total_medals
            FROM `Olympic_games.olympic_history`
            WHERE medal IN ('Gold', 'Silver', 'Bronze')
            GROUP BY name, team
            ORDER BY total_medals DESC),
        t2 AS
            (SELECT *, DENSE_RANK() OVER (ORDER BY total_medals DESC) AS rnk
            FROM t1)
    SELECT name, team, total_medals
    FROM t2;
  
  -- In which Sport/event, has Jamaica won the most medals

WITH t1 AS
        	(SELECT sport, COUNT(1) AS total_medals
        	FROM `Olympic_games.olympic_history`
        	WHERE medal <> 'NA'
        	AND team = 'Jamaica'
        	GROUP BY sport
        	ORDER BY total_medals DESC),
        t2 AS
        	(SELECT *, RANK() OVER(ORDER BY total_medals DESC) AS rnk
        	FROM t1)
    SELECT sport, total_medals
    FROM t2
    WHERE rnk = 1;


-- Breakdown of all olympic games where Finland won a medal in Ice Hockey and how many medals in each olympic games

SELECT team, sport, games, COUNT(1) AS total_medals
    FROM `Olympic_games.olympic_history`
    WHERE medal <> 'NA'
    AND team = 'Finland' AND sport = 'Ice Hockey'
    GROUP BY team, sport, games
    ORDER BY total_medals DESC;


WITH t1 AS
            (SELECT team, COUNT(1) AS total_medals
            FROM `Olympic_games.olympic_history`
            JOIN `Olympic_games.countries` ON NOC =NOC
            WHERE medal <> 'NA'
            GROUP BY team
            ORDER BY total_medals DESC),
        t2 as
            (SELECT *, DENSE_RANK() over(ORDER BY total_medals DESC) AS rnk
            FROM t1)
    SELECT *
    FROM t2
   WHERE rnk <= 5 ;


-- A list of total gold, silver and broze medals won by each country corresponding to each olympic games.

SELECT team,games,SUM(CASE WHEN medal='Gold' THEN 1 ELSE 0 END) Gold,
SUM(CASE WHEN medal='Silver' THEN 1 ELSE 0 END) Silver,
Sum(CASE WHEN medal='Bronze' THEN 1 ELSE 0 END) Bronze,
COUNT(team) AS total_medals
FROM `Olympic_games.olympic_history` GROUP BY team,games ORDER BY total_medals DESC;

-- Identify which sport has been played in all summer olympics.
with t1 as
          	(select count(distinct games) as total_games
          	from `Olympic_games.olympic_history` where season = 'Summer'),
          t2 as
          	(select distinct games, sport
          	from `Olympic_games.olympic_history` where season = 'Summer'),
          t3 as
          	(select sport, count(1) as no_of_games
          	from t2
          	group by sport)
      select *
      from t3
      join t1 on t1.total_games = t3.no_of_games;
