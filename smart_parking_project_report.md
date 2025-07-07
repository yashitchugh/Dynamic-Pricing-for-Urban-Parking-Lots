
# 🅿️ Smart Parking Pricing System – Final Report
**Project by:** *[Yashit Chugh]*  
**Institution:** *IIT Guwahati *  
**Technology Stack:** Python, Pandas, Bokeh, Pathway  

---

## 📌 Objective

To build a **real-time smart parking pricing system** that dynamically adjusts parking prices based on demand, competition, and other influencing features. The solution is structured into three models with increasing complexity and intelligence.

---

## 📁 Phase 1: Data Preprocessing

### ✅ Tasks:
- Combined 'LastUpdatedDate' and 'LastUpdatedTime' into a new Timestamp column:
  ```python
  df['Timestamp'] = pd.to_datetime(df['LastUpdatedDate'] + ' ' + df['LastUpdatedTime'], format='%d-%m-%Y %H:%M:%S')
  ```
- Sorted the DataFrame by time and reset the index:
  ```python
  df = df.sort_values('Timestamp').reset_index(drop=True)
  ```
  ✅ **Why:** Ensures time-series order for all real-time simulation and windowing.

- Converted categorical and numerical columns as needed.
- Removed or filtered out unwanted vehicle types (like 'cycle').

---

## 🔁 Phase 2: Dynamic Pricing Models

We developed and implemented **three pricing models**, each adding a layer of intelligence.

---

## 🧮 Model 1: **Baseline Dynamic Pricing**

**Goal:**  
Adjust price based on current occupancy rate using a demand multiplier.

### Formula:
```python
new_price = previous_price + ALPHA * (occupancy / capacity)
```

### Constraints:
- MIN_PRICE = 5, MAX_PRICE = 20, BASE_PRICE = 10
- Bounded using:
  ```python
  new_price = max(MIN_PRICE, min(MAX_PRICE, new_price))
  ```

### ✅ Justification:
This model reacts to usage level directly and helps scale prices based on demand in a simple and interpretable way.

---

## 📊 Model 2: **Multi-Factor Demand-Based Pricing**

**Goal:**  
Predict price based on multiple features like queue length, traffic, vehicle type, and special day flags.

### Formula:
```python
raw_demand = (occupancy/capacity) + 0.5*queue_length - 0.3*traffic + 2*special_day + weight(vehicle_type)
norm_demand = raw_demand / max_possible_value
demand_price = BASE_PRICE * (1 + 0.5 * norm_demand)
```

### Bounded and Clipped:
```python
df['demand_price'] = df['demand_price'].clip(lower=5, upper=20)
```

### ✅ Justification:
This model simulates real-world demand fluctuations from various sources and gives more accurate, behaviorally-aware pricing.

---

## 🔄 Model 3: **Competitive Pricing Adjustment**

**Goal:**  
Align each lot’s price with the average price of nearby lots (simulated using location_id).

### Logic:
```python
df['competitive_price'] = df['demand_price'] + 0.5 * (avg_location_price - df['demand_price'])
df['competitive_price'] = df['competitive_price'].clip(5, 20)
```

### ✅ Justification:
Introduces **market-awareness**, discourages overpricing or undercutting, and reflects smart competition strategies.

---

## 🕒 Real-Time Simulation (Pathway)

**Goal:**  
Stream data in real-time and update pricing on the fly.

### Key Pathway Steps:
- Defined schema using either pw.Schema or column_names
- Simulated stream with:
  ```python
  mode="streaming", delay_ms=500
  ```
- Applied pricing functions as @pw.udf
- Windowed aggregation with:
  ```python
  pw.temporal.tumbling(datetime.timedelta(days=1))
  ```
- Final daily price output with:
  ```python
  price = 10 + (occ_max - occ_min) / cap
  ```

✅ **Why:** Needed for real-time, reactive system to simulate deployment on smart city infrastructure.

---

## 📈 Visualization (Bokeh)

### Plot: Real-Time Pricing Comparison
- Plotted both demand_price and competitive_price over time.
- Used solid lines for demand and dashed for competitive pricing.
- Filtered by selected lot IDs to avoid clutter.

### Code Sample:
```python
p.line(df['timestamp'], df['demand_price'], ...)
p.line(df['timestamp'], df['competitive_price'], line_dash='dashed', ...)
```

✅ **Justification:** Demonstrates the **evolution of pricing** clearly and justifies behavior visually.

---

## 🧠 Insights & Results

| Model     | Strength                     | Limitation                     |
|-----------|------------------------------|--------------------------------|
| Model 1   | Simple, fast, interpretable  | Ignores other features         |
| Model 2   | Richer, demand-aware         | No market-awareness            |
| Model 3   | Market-aware, fair pricing   | Needs group clustering         |

---

## ✅ Final Thoughts

This project successfully:
- Processed and streamed real-time parking data
- Designed 3 tiered dynamic pricing models
- Justified all pricing with temporal and competitive logic
- Visualized results using interactive dashboards

The framework is scalable to include machine learning models, real GPS-based clustering, or IoT-based real-time integration in future extensions.
