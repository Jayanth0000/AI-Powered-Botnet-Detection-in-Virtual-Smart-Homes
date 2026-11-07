# Lightweight & Explainable IoT Botnet Detection using Regularised Random Forests

A botnet detector for smart-home IoT devices. Each model is a shallow Random Forest, kept small enough to run on an IoT gateway. The model is trained on network traffic features to separate normal device traffic from Mirai and Bashlite (Gafgyt) attacks. The project report is `IoT_Project_Report_23BCE8392.pdf`.

## Folder contents

| Path | Purpose |
|---|---|
| `IoT_Botnet_Detection-1.ipynb` | Per-device training and evaluation on the `data1/` folders |
| `IoT_Botnet_Detection-2.ipynb` | Per-device training and evaluation on the `data2/` folders, which include extra devices. Notebook 1 has the cross-malware test and the timing benchmark |
| `IoT_base_code.txt` | Template notebook code. Replace the placeholder folder and device names before running it |
| `botnet_detector.pkl` | Saved Random Forest trained on Mirai traffic |
| `bashlite_detector.pkl` | Saved Random Forest trained on Bashlite traffic |
| `data/`, `data1/`, `data2/` | Per-device traffic CSVs. Each device folder holds `benign_traffic.csv` and `udpplain.csv` |
| `Iot_Project.pptx` | Presentation |
| `IoT_conference_paper_23BCE8392_T.Jayanth.pdf` | Conference paper |
| `IoT_Project_Report_23BCE8392.pdf` | Project report |
| `IoT_Project.zip` | Archive of the project folder (about 3.5 GB uncompressed). Keep it only as a backup |

## Data

- Each device folder contains `benign_traffic.csv` (normal traffic) and `udpplain.csv` (UDP flood traffic, labelled as botnet).
- Each CSV has 115 statistical features. The feature names combine a window and a statistic, for example `H_L1_mean`, `MI_dir_L5_weight` and `H_L0.01_weight`. The windows are L0.01, L0.1, L1, L3 and L5.
- The notebooks read the first 5,000 rows of each file, so each device gets 5,000 benign and 5,000 attack samples.
- Devices in `data1/`: Danmini Doorbell, Ecobee Thermostat, Philips B120N10 Baby Monitor, Provision PT-737E and PT-838 Security Cameras, SimpleHome XCS7 1002 and 1003 Security Cameras.
- Devices in `data2/`: the same devices plus Ennio Doorbell and Samsung SNH-1011-N Webcam.

## Method

1. Load the first 5,000 benign rows and 5,000 attack rows for one device.
2. Label benign traffic `0` and attack traffic `1`, then merge and shuffle.
3. Split with `train_test_split(test_size=0.9, random_state=42)`. The model trains on 10% of the data and is tested on the other 90%.
4. Train `RandomForestClassifier(n_estimators=10, max_depth=2)`.
5. Evaluate with a classification report, accuracy and a confusion matrix.
6. Rank features by importance to find the strongest botnet indicators.
7. Run a cross-malware test: a Mirai-trained model on Bashlite data, and the reverse.
8. Save the model with `joblib.dump`.

## Results

Accuracy on the 90% test split for each device, as reported in the notebooks:

| Device | Botnet type | Accuracy (%) |
|---|---|---|
| Danmini Doorbell | Mirai | 99.97 |
| Ecobee Thermostat | Mirai | 99.98 |

The report gives an overall figure of about 99.9% across nine hardware profiles.

Cross-malware test (from the report):

| Model | Tested on Bashlite | Tested on Mirai |
|---|---|---|
| Mirai-trained | 49.83% | 99.97% |
| Bashlite-trained | 99.93% | 50.06% |

Each model scores close to chance on the other botnet family. The report reads this as the two families having different traffic signatures.

Inference timing (from the report):

- Mirai model: 3.45 µs per packet
- Bashlite model: 5.56 µs per packet

## Setup

```bash
pip install pandas numpy scikit-learn matplotlib seaborn joblib
```

Open the notebooks in Jupyter from this folder, so the relative paths `data1/` and `data2/` resolve.

## Using a saved model

```python
import joblib
model = joblib.load("botnet_detector.pkl")
label = model.predict(features)  # features: a DataFrame with the 115 columns; 1 = botnet, 0 = normal
```

## Known issues in the notebooks

- In Step 9 of both notebooks, `loaded_model = joblib.dump(...)` should be `joblib.load(...)`. The line currently saves the model again and does not load it.
- The cross-malware cells reuse `X_test` and `y_test` from the current device, so the transfer score depends on which device's data is loaded.
- The notebooks contain the author's reported results. They were not re-run as part of this README.
