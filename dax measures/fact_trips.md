Total Trips = SUM(fact_trips[total_trips])

Total Revenue = SUM(fact_trips[total_revenue])

Avg Fare (Weighted) = SUMX(fact_trips, fact_trips[avg_fare] * fact_trips[total_trips]) / SUM(fact_trips[total_trips])

Avg Duration (Weighted) = SUMX(fact_trips, fact_trips[avg_duration_min] * fact_trips[total_trips]) / SUM(fact_trips[total_trips])