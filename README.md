# Industrial anomaly monitoring and predictive maintenance

This repository presents end-to-end predictive maintenance pipelines for two distinct classes of industrial assets: continuous thermodynamic systems and cyclic electromechanical machinery.

---

## Case study 1: Combined cycle power plant digital twin

A combined cycle power plant pairs gas and steam turbines to generate up to 680 MW of base-load electricity. Power output fluctuates continuously with ambient weather conditions (temperature, pressure, and relative humidity), making it difficult to distinguish weather-driven power swings from genuine mechanical fouling.

* We train an empirical digital twin using a Random Forest ensemble, evaluated with Median Absolute Error (1.84 MW MedAE, 0.9558 R²) to measure baseline prediction accuracy without distortion from rare plant trip outliers.

* Historical plant trips down to -50 MW severely contaminate conventional metrics; using Median Absolute Deviation (MAD) recovers a 23% tighter operational baseline (2.80 MW vs. 3.63 MW) because scaled MAD replicates theoretical Gaussian variance under normal conditions while ignoring heavy-tailed outage spikes.

* An exponential moving average (EWMA) suppresses high-frequency sensor noise, narrowing normal residual fluctuations down to ±0.93 MW. In a 680 MW facility, keeping normal monitoring error below 1 MW allows operators to detect subtle mechanical degradation early.

* In gradual cooling condenser fouling simulations, the robust alerting engine flags critical power loss at hour 20 compared to hour 27 for conventional thresholds, providing an entire 7-hour operational shift of advance warning.

* To avoid false dispatches from temporary grid fluctuations, the system enforces a 2-hour persistence rule: an operational fault is confirmed only when the power deficit persists continuously for two consecutive hours, achieving a 0.00% false alarm rate across 200 hours of continuous baseline validation.

---

## Case study 2: Rolling-stock air production unit monitoring

Auxiliary air production units (APUs) supply compressed air (8.5 to 10.0 bar) for passenger train friction brakes, air suspension, and automated doors. Unlike continuous turbines, compressors cycle intermittently between active pumping and atmospheric rest, creating brief motor inrush currents and multimodal pressure regimes that mislead standard anomaly models.

* Multi-sensor triangulation across intake valve commands (`COMP`), electrical current, and discharge pressure (`TP2`) isolates operational regimes without data leakage, identifying that active compression occupies 15.5% of total revenue service.

* Telemetry screening across variance and physical coupling criteria prunes 15 raw channels down to 10 candidate features, removing unmonitored drain lines and redundant pressure reflections that duplicate existing information.

* A Spearman rank correlation audit on healthy baseline data verifies that all ten candidate features remain orthogonal (|r| < 0.85).

* We train a one-class Isolation Forest on the healthy baseline (February–March 2020: 73,638 minutes). Features are normalized using median and interquartile range (`RobustScaler`) rather than standard mean and variance, preventing brief 10-second motor inrush surges and high breakdown temperatures from compressing the normal operating range.

* Anomaly scores are smoothed using a 15-minute rolling persistence filter (`window=15`, `min_periods=5`) matching the compressor's thermodynamic cycle (3 minutes pumping, 12 minutes venting). This sliding window dilutes isolated 1-minute startup spikes into surrounding idle rest, eliminating nuisance alarms during normal operations while locking onto continuous, sustained breakdowns.

* Benchmarked against official maintenance logs, the persistence engine achieves a 100% event capture rate across all documented failure incidents, providing up to 2.0 hours of advance lead time before severe air leaks and 14.5 hours of precursor warning during intake valve chattering.

* In an extended benchmark against all 21 failure events recorded in the MetroPT research literature, the model achieves a 100.0% event detection recall and exposes uncatalogued multi-day air leaks in the raw fleet telemetry.

---

## Executive results matrix

| Performance metric | Case 1: Power plant (CCPP) | Case 2: Train compressor (APU) | Operational impact |
| :--- | :---: | :---: | :--- |
| Primary asset type | 680 MW turbine plant | 5.5 kW reciprocating compressor | Continuous vs. cyclic machinery |
| Baseline model | Random Forest regressor | One-class Isolation Forest | Supervised twin vs. one-class novelty boundary |
| Modeling accuracy | 1.84 MW MedAE (0.9558 R²) | 99.29% failure recall | High baseline tracking fidelity |
| Noise suppression technique | EWMA memory filter | 15-minute rolling persistence | Smooths high-frequency sensor jitter |
| Nominal false alarm rate | 0.00% (2-hour persistence) | 0.00% on healthy service day | Completely eliminates alarm fatigue |
| Breakdown event capture | 100% of simulated fouling | 100% of official failures (4/4) | Zero missed catastrophic breakdowns |
| Advance warning lead time | +7.0 hours advance notice | +2.0h (air leak), +14.5h (valve hunting) | Shifts emergency halts to planned repairs |
| Asset life protection | Prevents condenser scaling | Stops motor burnout at 88.8°C | Replaces full overhauls with routine seals |

---

## Repository organization

```text
├── Power_Plant.ipynb          # Case study 1: Combined cycle power plant digital twin
├── MetroPT-3.ipynb            # Case study 2: Train air compressor anomaly detection
├── docs/
│   ├── ccpp_thermodynamics.md # Equipment specifications and thermodynamic process flow for CCPP
│   └── apu_thermodynamics.md  # APU specifications, telemetry dictionary, and 21 failure logs
├── images/
│   ├── ccpp_diagram.png       # Process diagram for combined cycle generation
│   └── apu_architecture_diagram.png # Mechanical schematic for train pneumatic system
└── README.md                  # Project overview and executive summary
```

---

## Environment setup and reproduction

### Prerequisites
* Python 3.10 or higher
* Recommended: Virtual environment (`venv` or `conda`)

### Installation

1. Clone the repository:
```bash
git clone https://github.com/otrebor-solrac/industrial-anomaly-monitoring.git
cd industrial-anomaly-monitoring
```

2. Create and activate a virtual environment:
```bash
python3 -m venv venv
source venv/bin/activate
```

3. Install required packages:
```bash
pip install pandas numpy scipy scikit-learn matplotlib seaborn jupyter
```

4. Launch Jupyter to run the notebooks:
```bash
jupyter notebook
```
Open [Power_Plant.ipynb](Power_Plant.ipynb) or [MetroPT-3.ipynb](MetroPT-3.ipynb) to execute the complete end-to-end pipelines.
