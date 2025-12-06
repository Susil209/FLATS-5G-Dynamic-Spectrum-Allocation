# FLATS-5G: Federated Learning-Augmented Transformer-LSTM Spectrum Allocation

# FLATS-5G: Machine Learning–Based Dynamic Spectrum Allocation in 5G

This repository contains the implementation of **FLATS-5G**, a machine learning–driven framework for **traffic prediction** and **access-aware dynamic bandwidth allocation** in 5G networks. It is inspired by and extends the methodology of:

> R. I. Rony et al., “Dynamic Spectrum Allocation Following Machine Learning-Based Traffic Predictions in 5G,” *IEEE Access*, 2021.

The current stage of the project focuses on:
- Cleaning and unifying real/simulated 5G KPI datasets,
- Implementing and tuning a **NARNET baseline** for traffic prediction,
- Designing a **dynamic bandwidth allocation** module, and
- Building a **three-phase results framework** for fair comparison across models.

---

## 1. Datasets

This project uses publicly available 5G KPI datasets:

- **5G Network Metrics for High-Traffic Event** (IEEE DataPort)  
- **5G Network Data** (Kaggle, vinothkannaece)

These datasets provide cell-level measurements such as:
- Traffic load and UE demand,
- Radio KPIs (RSRP, RSRQ, CQI, MCS),
- Bandwidth utilization and latency,
- Network slice information, and
- Resource block allocation.

All dataset copyrights remain with their respective owners. This repository only uses the data for research and academic purposes.

---

## 2. Preprocessing and Clean Dataset

The preprocessing pipeline is implemented in `FLATS5G_Preprocessor` and performs:

- Duplicate removal and missing-value handling,
- Categorical encoding (Mobility, Frequency_Layer, UE_Status, Backhaul_Status, NFV_Status, Congestion_Level),
- Feature engineering (SINR_approx, Spectral_Efficiency, Load_Ratio, Util_Efficiency, etc.),
- MinMax scaling for numeric KPIs, and
- 5G traffic scaling using a **Scaling Factor (SF)** (e.g., SF=25) to emulate 5G-level loads.

For modeling, a **clean minimal dataset** is created with only essential features:

- `Physical_Cell_ID`, `Timestamp`
- `Traffic_Load_SF25` (target), `UE_Demand_SF25`
- `RSRP`, `RSRQ`, `CQI`, `MCS`
- `BW_Utilization (%)`, `Latency (ms)`
- `Network_Slice`, `RB_Allocation`

This clean dataset is saved as:


---

## 3. NARNET Baseline (Traffic Prediction)

A per-cell **autoregressive neural network (NARNET)** is implemented as the main baseline.

### 3.1. Sequence Generation

- For each cell, the target series is `Traffic_Load_SF25`.
- The series is MinMax scaled to [0, 1].
- Autoregressive sequences are created:

\[
X_t = [y_{t-d}, \dots, y_{t-1}], \quad y_t = \text{Traffic\_Load\_SF25}(t)
\]

- Temporal split: 70% train, 15% validation, 15% test.

### 3.2. Model Architecture

- Input: delay `d` past samples (tested d = 1, 4, 8).
- Hidden layer: 8 neurons, `tanh` activation, L2 regularization (≈ Bayesian Regularization behaviour).
- Output: 1 neuron, linear activation.
- Optimizer: Adam (lr=0.001), loss: MSE.
- Early stopping and learning rate reduction on plateau.

### 3.3. Performance

A systematic delay search was performed (d = 1, 4, 8) on the augmented dataset:

- **d = 1**: mean RE ≈ 0.316 (insufficient)
- **d = 4**: mean RE ≈ 0.052
- **d = 8**: **mean RE ≈ 0.012–0.015, R² ≈ 0.99–0.997**

The tuned NARNET (delay=8) is used as the **main prediction baseline** for subsequent models (LSTM, Transformer, FLATS-5G).

---

## 4. Dynamic Bandwidth Allocation and Metrics

A **DynamicBandwidthAllocator** is implemented to emulate the access-aware spectrum allocation from the base paper, using a shared spectrum pool (e.g., 100 MHz total, 5 MHz chunks / RB abstraction).

### 4.1. Allocation Approaches

For each time step and scaling factor (SF):

- **Static-1**: Max 20 MHz per cell (or equal division if over-subscribed),
- **Static-2**: Entire bandwidth equally divided among cells,
- **Dynamic (NARNET-based)**: RBs allocated proportionally to predicted traffic, with min/max RB constraints.

### 4.2. Allocation Metrics

For each approach, the following metrics are computed:

- **Unsatisfied cells**  
  Number of cells where achievable throughput fails to meet a minimum fraction (e.g., 80%) of their demand.

- **Utilization Factor (UF)**  
  \[
  UF = \frac{\sum \min(\text{Demand}, \text{Throughput})}{\sum \text{Demand}}
  \]  
  Measures how efficiently bandwidth is used (target: close to 1).

- **Bandwidth savings**  
  \[
  \text{BW\_Savings} = \left(1 - \frac{\sum \text{BW}_{\text{dynamic}}}{\sum \text{BW}_{\text{static}}}\right) \times 100\%
  \]

- **Inference time**  
  Average milliseconds per prediction for NARNET (and later, other models).

These metrics are exported for comparison and plotting.

---

## 5. Three-Phase Results Framework

To mirror the base paper’s structure, a three-phase analysis is implemented:

1. **Peak-Hour Analysis**
   - Identify the time slot with the highest average load (e.g., around 23:00–23:15).
   - For multiple scaling factors (SF = 10–120), compare:
     - Unsatisfied cells (Static-1, Static-2, Dynamic-NARNET)
     - UF across approaches

2. **24-Hour Analysis**
   - Aggregate all 7 days into a single 24-hour profile.
   - Plot:
     - Average traffic per hour (with variance)
     - Hourly UF for Static-1, Static-2, Dynamic-NARNET
     - Hourly unsatisfied cells

3. **ML-Based Prediction and BW Allocation**
   - Compare **Oracle** (allocation with real traffic) vs **ML** (allocation with NARNET-predicted traffic) vs **Static**.
   - Evaluate impact of prediction error on UF, unsatisfied cells, and BW usage.
   - Generate:
     - Predicted vs actual scatter plots,
     - UF vs SF (Oracle vs ML vs Static),
     - Unsatisfied cells vs SF.

This framework will be reused for LSTM, Transformer, and FLATS-5G, enabling **like-for-like comparison**.

---

## 6. Planned Next Steps

The repository is structured to extend beyond NARNET:

1. **LSTM-only model**
   - Sequence-to-one or sequence-to-sequence architectures.
   - Same per-cell setup and metrics framework.

2. **Transformer-only model**
   - Attention-based temporal modeling for traffic prediction.

3. **FLATS-5G Hybrid**
   - Combine LSTM and Transformer components for improved robustness.
   - Use the same allocation module to evaluate:
     - RE, unsatisfied cells, UF, BW savings, inference time.

4. **Final comparison table**
   - Static-1, Static-2, NARNET, LSTM, Transformer, FLATS
