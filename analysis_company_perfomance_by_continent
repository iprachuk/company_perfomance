-- CTE 1: Calculate revenue metrics by continent
WITH revenue_result AS(
SELECT
  sp.continent,
  SUM(p.price) AS revenue,  -- Total revenue per continent
  -- Revenue generated from mobile devices
  SUM(CASE WHEN sp.device = 'mobile' THEN p.price ELSE 0 END) AS revenue_from_mobile,
  -- Revenue generated from desktop devices
  SUM(CASE WHEN sp.device = 'desktop' THEN p.price ELSE 0 END) AS revenue_from_desktop
FROM `DA.order` o
JOIN `DA.product` p
  ON o.item_id = p.item_id
JOIN `DA.session_params` sp
  ON o.ga_session_id = sp.ga_session_id
-- Grouping by continent to aggregate revenue metrics
GROUP BY sp.continent
),

-- CTE 2: Collect user and session metrics by continent
data_result AS(
SELECT
  sp.continent AS continent,
  COUNT(acc.id) AS account_count,  -- Total number of accounts
  -- Count only verified accounts
  COUNT(CASE WHEN acc.is_verified = 1 THEN acc.id END) AS verified_account,
  COUNT(s.ga_session_id) AS session_count  -- Total number of sessions
FROM `DA.session` s
LEFT JOIN `DA.session_params` sp
  ON s.ga_session_id = sp.ga_session_id
LEFT JOIN `DA.account_session` acs
  ON s.ga_session_id = acs.ga_session_id
LEFT JOIN `DA.account` acc
  ON acs.account_id = acc.id
-- Grouping by continent for aggregation
GROUP BY continent
)

-- Final SELECT: Combine revenue metrics with user/session metrics
SELECT
  dr.continent,
  rr.revenue,
  rr.revenue_from_mobile,
  rr.revenue_from_desktop,
  -- Percentage of total revenue across all continents
  rr.revenue / SUM(revenue) OVER() * 100 AS percent_revenue_from_total,
  dr.account_count,
  dr.verified_account,
  dr.session_count

FROM data_result dr
LEFT JOIN revenue_result rr
  ON dr.continent = rr.continent

-- Exclude empty or undefined continent values
WHERE rr.continent NOT IN ('', '(not set)')
