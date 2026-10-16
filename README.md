# enterprise-user-engagement-analytics
WITH monthly_account_activity AS (
    SELECT
        org_id,
        DATE_TRUNC('month', event_timestamp) AS activity_month,
        COUNT(DISTINCT user_id) AS active_users,
        COUNT(DISTINCT session_id) AS total_sessions
    FROM 
        prod.telemetry.user_session_logs
    WHERE 
        event_timestamp >= DATEADD('month', -6, CURRENT_DATE())
    GROUP BY 
        org_id, 
        DATE_TRUNC('month', event_timestamp)
),
historical_baseline AS (
    SELECT
        org_id,
        activity_month,
        active_users,
        -- Calculate rolling 6-month average and standard deviation of active users
        AVG(active_users) OVER (
            PARTITION BY org_id 
            ORDER BY activity_month 
            ROWS BETWEEN 6 PRECEDING AND 1 PRECEDING
        ) AS baseline_avg_users,
        LAG(active_users, 1) OVER (
            PARTITION BY org_id 
            ORDER BY activity_month
        ) AS prev_month_users
    FROM 
        monthly_account_activity
)
SELECT
    org_id,
    activity_month,
    active_users,
    prev_month_users,
    ROUND(SAFE_DIVIDE(active_users - prev_month_users, prev_month_users) * 100, 2) AS mom_user_change_pct,
    CASE 
        WHEN active_users < (baseline_avg_users * 0.80) THEN 'High Risk - Usage Drop'
        ELSE 'Stable'
    END AS account_health_status
FROM 
    historical_baseline
WHERE 
    prev_month_users IS NOT NULL
ORDER BY 
    mom_user_change_pct ASC;
