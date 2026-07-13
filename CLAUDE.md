# EV Spare Parts Inventory Analytics — Project Context

## What this project is
We are analysts for an EV spare-parts warehouse supplying several EV service centers.
Using weekly historical data, we analyze demand, inventory performance, and costs,
then compare demand forecasting methods in two ways: statistical error (MSE, MAE, MAPE)
and business cost (holding cost from over-forecasting, stockout cost from under-forecasting).

## Central question (everything must connect back to this)
**Can a forecasting method with low statistical error still create high inventory cost?**
Every analysis, chart, and conclusion in the notebook should help answer this question.

## Data (in data/)
All data is weekly. `week_start` marks the beginning of each weekly period.
Join demand and inventory on (`week_start`, `part_id`).

### parts.csv — one row per spare part
- part_id: unique part identifier
- part_name: part name
- category: e.g. Battery System, Charging System, Brake System, Thermal Management,
  Power Electronics, Electric Motor, Sensors, Cables and Connectors, Control Units
- unit_cost: purchasing cost per unit
- unit_price: selling / internal transfer price per unit
- supplier: supplier name
- lead_time_days: average supplier lead time in days
- demand_trend: synthetic label — Increasing, Decreasing, Stable, or Seasonal
- holding_cost_per_unit_week: weekly cost of keeping one unit in stock
- stockout_cost_per_unit: cost of one unit of unmet demand

### demand.csv — weekly demand per part
- week_start, part_id
- units_demanded: units requested by service centers that week
- service_center_region: region of the requesting center
- vehicle_type: EV model group
- service_campaign: whether a maintenance/recall campaign was active

### inventory.csv — weekly inventory BEFORE demand is fulfilled
- week_start, part_id
- stock_on_hand: inventory at the beginning of the week
- replenishment: units received from suppliers during the week

## Required derived variables (use these EXACT formulas, do not improvise)
After merging demand.csv and inventory.csv:
- available_inventory = stock_on_hand + replenishment
- units_fulfilled = min(available_inventory, units_demanded)
- unmet_demand = max(units_demanded - available_inventory, 0)
- ending_stock = max(available_inventory - units_demanded, 0)
- stockout = 1 if unmet_demand > 0 else 0

## Deliverables
1. notebooks/ev_spare_parts_inventory_forecasting.ipynb
   - A NARRATIVE notebook: every section starts with a markdown cell explaining
     what we are doing and why, and ends with a short interpretation of results.
   - Readable code with clear comments. No dead code, no unexplained magic numbers.
2. outputs/reorder_recommendations.csv with at least these columns:
   part_id, part_name, category, current_stock, forecasted_weekly_demand,
   lead_time_days, safety_stock, reorder_point, recommended_order_quantity,
   recommendation
   - The recommendation column must distinguish: needs immediate replenishment /
     excessive inventory / high stockout risk / expensive low-demand part
     (do not overstock).

## Analysis plan (phases)
1. Data audit: shapes, dtypes, missing values, duplicate (part_id, week_start),
   date gaps, negative values.
2. Merge + derived variables, with sanity check:
   total demand == total fulfilled + total unmet, no negatives.
3. EDA: demand by part/category/region, stockout rate, fill rate, unmet demand,
   ending stock, high-cost vs high-criticality parts.
4. Forecasting: naive, moving average, exponential smoothing; Holt-Winters for
   seasonal parts; consider Croston's method if demand is intermittent.
5. Dual evaluation: error metrics vs cost metrics; rank methods both ways and
   highlight where the rankings disagree.
6. Reorder logic: safety stock, reorder point, recommended order quantity,
   then produce reorder_recommendations.csv.

## Conventions and rules
- Forecasts and order quantities must be rounded to INTEGER units before any
  inventory calculation.
- Time-based train/test split only. NEVER shuffle time series data.
- Exclude weeks with zero demand when computing MAPE (division by zero).
- Overage cost assumption: units over-forecast x holding_cost_per_unit_week.
  Underage cost assumption: units under-forecast x stockout_cost_per_unit.
  State all cost assumptions explicitly in markdown where they are used.
- Convert lead_time_days to weeks (divide by 7) when combining with weekly demand.
- Label all plot axes and titles. Every plot needs one sentence of interpretation.
- Prefer simple, readable pandas over clever one-liners.
- Commit to git after each completed, working step.
