# SQLportfolio

<b>SCENARIO 1 - Min, Max, and Average Closing Price</b>


SELECT
    security,
    MIN(close_price)||' $' AS min_price,
    MAX(close_price)||' $' AS max_price,
    ROUND(AVG(close_price),2)||'%' AS avg_price
FROM 
    price_data
GROUP BY 
    security;


<b>SCENARIO 2 - Most Significant Positive Spike</b>


WITH price_spikes AS (
    SELECT 
        security, 
        date, 
        close_price,
        close_price - LAG(close_price) OVER (PARTITION BY security ORDER BY date) AS price_spike
    FROM 
        price_data
),
ranked_spikes AS (
    SELECT 
        security, 
        date, 
        close_price, 
        price_spike,
        ROW_NUMBER() OVER (PARTITION BY security ORDER BY price_spike DESC) AS rank
    FROM 
        price_spikes
    WHERE 
        price_spike IS NOT NULL
)
SELECT 
    security, 
    date, 
    close_price, 
    price_spike
FROM 
    ranked_spikes
WHERE 
    rank = 1;


<b>SCENARIO 3 - ROI using the most significant spike's date</b>


WITH base_data AS (
    SELECT 
		security,
        MIN(close_price) AS buy_price,
        MAX(close_price) AS spike_price
    FROM 
        price_data
    WHERE 
        date <= CURRENT_DATE
    GROUP BY 
        security
)
SELECT 
    security,
    1000 * (spike_price - buy_price) AS roi
FROM 
    base_data;
