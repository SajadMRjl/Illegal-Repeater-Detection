# Wireless Repeater Detection Project

Course project for detecting unauthorized repeaters in a mobile network using drive test data analysis.

## Project Overview

This project simulates a mobile network in Tehran, Iran, with Base Transceiver Stations (BTS) and unauthorized repeaters. It generates synthetic drive test measurements using the Friis transmission equation and detects repeater locations through statistical anomaly detection.

### Key Features

- **Synthetic Network Generation**: Generates realistic BTS locations and unauthorized repeaters
- **Friis Equation Propagation**: Models signal propagation using free-space path loss
- **Dual-Path Signal Model**: Correctly combines direct BTS signals and repeater-amplified signals
- **Statistical Anomaly Detection**: Uses z-score based detection to identify signal anomalies
- **DBSCAN Spatial Clustering**: Filters noise and identifies repeater clusters
- **Interactive Visualization**: Creates interactive maps with Folium showing BTS, repeaters, and detection results

## Technology Stack

- **Python 3.10+**
- **Core**: NumPy, Pandas, SciPy
- **Geospatial**: GeoPy, Shapely
- **Visualization**: Folium, Plotly, Matplotlib
- **Machine Learning**: scikit-learn (DBSCAN clustering)
- **Development**: Jupyter Notebook

## Project Structure

```
Wireless/
├── data/                           # Generated data files
│   ├── bts_stations.csv           # BTS locations
│   ├── repeaters.csv              # Repeater locations
│   └── drive_test_measurements.csv # Measurement data
├── output/                         # Visualization output files
│   ├── tehran_network_map.html    # Main interactive map
│   └── detection_comparison.html  # Side-by-side comparison maps
├── pkg/                            # Package source code
│   ├── config.py                  # Configuration parameters
│   ├── bts_generator.py           # BTS and repeater generation
│   ├── propagation.py             # Friis equation implementation
│   ├── drive_test_simulator.py    # Measurement simulation
│   ├── detection.py               # Detection algorithm
│   └── visualization.py           # Interactive maps and plots
├── simulation_and_detection.ipynb  # Main Jupyter notebook
├── requirements.txt               # Python dependencies
└── README.md                      # This file
```

## Installation

1. **Clone or download the project**:
   ```bash
   cd Wireless
   ```

2. **Install dependencies**:
   ```bash
   pip install -r requirements.txt
   ```

## Usage

### Recommended: Use Jupyter Notebook

For interactive analysis and visualization:

```bash
# Start Jupyter
jupyter notebook

# Open and run:
# - simulation_and_detection.ipynb
```

Or use JupyterLab:
```bash
jupyter lab
```

### Alternative: Use as Python Package

The modules can be imported and used programmatically:

```python
from pkg import bts_generator, drive_test_simulator, detection, visualization
from pkg import config

# Generate BTS and repeaters
bts_list = bts_generator.generate_bts_stations()
repeater_list = bts_generator.generate_repeaters(bts_list)
bts_generator.save_bts_to_csv(bts_list)
bts_generator.save_repeaters_to_csv(repeater_list)

# Simulate drive test
measurements = drive_test_simulator.simulate_drive_test(bts_list, repeater_list)

# Detect repeaters
predictions = drive_test_simulator.simulate_result_without_repeaters(bts_list, ...)
residuals = detection.calculate_residuals(measurements, predictions, bts_list)
anomalies = detection.detect_anomalies_statistical(residuals, bts_list)
detected_repeaters = detection.cluster_and_localize(anomalies)

# Visualize results
map_obj = visualization.create_main_detection_map(
    bts_list, repeater_list, detected_repeaters, measurements
)
```

## How It Works

### 1. Network Generation

- Generates BTS stations in a hexagonal grid pattern within Tehran boundaries
- BTS count depends on geographic bounds and separation distance (default: 1 km)
- Places 3 unauthorized repeaters (configurable) within 0.33-0.66 km of a reference BTS
- Ensures minimum separation between BTS to avoid overlap
- Each BTS has realistic parameter variations (power, gain, height)

### 2. Signal Propagation (Friis Equation)

The Friis transmission equation calculates received signal strength:

```
Pr(d) = Pt - PL(d)
PL(d) = 20*log10(d) + 20*log10(f) + 32.45 - Gt - Gr
```

Where:
- `Pr(d)`: Received power at distance d (dBm)
- `Pt`: Transmit power (dBm)
- `PL(d)`: Path loss (dB)
- `d`: Distance (km)
- `f`: Frequency (MHz)
- `Gt, Gr`: Antenna gains (dBi)

### 3. Dual-Path Propagation Model

**Critical Implementation**: Signals from direct BTS path and repeater path are combined logarithmically:

```python
# Convert dBm to linear power
P_linear = sum(10^(P_i/10) for each path)

# Convert back to dBm
P_total = 10 * log10(P_linear)
```

This correctly models the physics of signal combination.

### 4. Drive Test Simulation

- Creates grid of measurement points (100m spacing, configurable)
- At each point, calculates RSSI from all BTS
- Includes both direct and repeater-amplified paths
- Adds log-normal shadowing noise (σ = 4 dB, configurable)
- Applies receiver sensitivity floor (-110 dBm)

### 5. Repeater Detection Algorithm

**Step 1**: Build expected coverage map (without repeaters)

**Step 2**: Calculate residuals: `residual = actual_RSSI - predicted_RSSI`

**Step 3**: Statistical anomaly detection:
```python
z_score = (residual - mean) / std
if z_score > 3.5: mark as anomaly  # Configurable threshold
```

**Step 4**: Spatial clustering with DBSCAN (eps=300m, min_samples=3)

**Step 5**: Localize repeater using weighted centroid of cluster

### 6. Validation

Compares detected repeater locations with actual planted locations:
- Calculates detection error (distance in meters)
- Reports detection rate and confidence metrics

## Configuration

Edit `pkg/config.py` to adjust simulation parameters:

```python
# Example: Change BTS separation (affects BTS count)
BTS_CONFIG['separation_km'] = 0.8  # Closer spacing = more BTS

# Example: Change number of repeaters
REPEATER_CONFIG['count'] = 5

# Example: Adjust detection sensitivity
DETECTION_CONFIG['z_score_threshold'] = 3.0

# Example: Change noise level
NOISE_CONFIG['log_normal_sigma_db'] = 6

# Example: Change grid spacing for drive test
DRIVE_TEST_CONFIG['grid_spacing_m'] = 50  # Higher resolution
```

## Output Files

The simulation generates:

1. **Data Files** (CSV):
   - `data/bts_stations.csv`: BTS locations and parameters
   - `data/repeaters.csv`: Repeater locations and parameters
   - `data/drive_test_measurements.csv`: All measurement data

2. **Visualization Files**

## Key Results

Typical detection performance:
- **Detection Rate**: 100% (all repeaters detected)
- **Mean Error**: 200-500 meters
- **Confidence**: High (z-score > 5.0 in cluster centers)

The algorithm successfully identifies unauthorized repeaters without requiring:
- Time-delay measurements (TOA/TDOA)
- Specialized RF equipment
- Prior knowledge of repeater parameters

## Algorithm Tuning

If detection fails or has false positives:

1. **No detections**:
   - Lower `z_score_threshold` (try 2.5 or 3.0)
   - Verify repeater gain is high enough (default: 60 dB)
   - Check repeater placement (default: 0.33-0.66 km from reference BTS)
   - Increase grid density for better coverage

2. **Too many false positives**:
   - Raise `z_score_threshold` (try 4.0 or higher)
   - Increase `dbscan_min_samples` (try 5 or 10)
   - Reduce noise level (`NOISE_CONFIG['log_normal_sigma_db']`)
   - Increase `dbscan_eps_m` to merge nearby clusters

3. **Low accuracy**:
   - Decrease grid spacing (try 50m or 30m)
   - Increase `dbscan_min_samples` for more stable clusters
   - Adjust `dbscan_eps_m` to better match repeater coverage area
   - Verify repeater gain settings are realistic

## References

### Signal Propagation
- Friis Transmission Equation
- Log-Normal Shadowing Model

### Detection Methods
- Statistical Anomaly Detection (Z-Score)
- DBSCAN Spatial Clustering

### Libraries
- [Folium](https://python-visualization.github.io/folium/) - Interactive maps
- [GeoPy](https://geopy.readthedocs.io/) - Geospatial calculations
- [scikit-learn](https://scikit-learn.org/) - DBSCAN clustering

## License

Academic course project - free to use and modify.

## Authors

Course: Wireless Networks
Project: Unauthorized Repeater Detection
