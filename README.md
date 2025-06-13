# A Distributed Machine Learning Approach for Network Intrusion Detection Using HIKARI-2021 Dataset

This project implements a machine learning pipeline for network traffic classification using Apache Spark and HDFS, running in Docker containers. The system classifies network traffic into different categories using the HIKARI 2021 dataset.

## Project Overview

- **Dataset**: HIKARI 2021 network traffic dataset
- **ML Algorithm**: Random Forest Classifier with class weighting
- **Infrastructure**: Dockerized HDFS + Spark cluster
- **Interface**: Jupyter Notebook with PySpark

## Architecture

```
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│   Jupyter       │  │   Spark Master  │  │   HDFS NameNode │
│   Notebook      │──│   + Worker      │──│   + DataNode    │
│   (Port 8888)   │  │   (Port 8080)   │  │   (Port 9870)   │
└─────────────────┘  └─────────────────┘  └─────────────────┘
```

## Prerequisites

- Docker Desktop installed and running
- PowerShell (Windows) or Terminal (Mac/Linux)
- At least 8GB RAM and 10GB free disk space
- HIKARI 2021 dataset (`hikari2021.csv`)

## Project Structure

```
dmlnid/
├── docker-compose.yml          # Docker services configuration
├── data/
│   └── hikari2021.csv         # Dataset file
├── notebooks/
│   └── hikari.ipynb           # Main analysis notebook
└── README.md                  # This file
```

## Step-by-Step Setup Guide

### 1. Clone/Download the Project

```powershell
# Navigate to your project directory
cd d:\dev\project\notebook\dmlnid
```

### 2. Prepare the Dataset

**Download the dataset:**
- Get the HIKARI 2021 dataset from: [[Dataset Link](https://www.kaggle.com/datasets/kk0105/allflowmeter-hikari2021?resource=download)]

Place your `hikari2021.csv` file in the `data/` directory:
```
dmlnid/
├── data/
│   └── hikari2021.csv    # ← Your dataset goes here
```

### 3. Start Docker Services

```powershell
# Start all services in the background
docker-compose up -d

# Check if all containers are running
docker ps
```

You should see 5 containers running:
- `namenode` (HDFS NameNode)
- `datanode` (HDFS DataNode)
- `spark-master` (Spark Master)
- `spark-worker` (Spark Worker)
- `notebook` (Jupyter Notebook)

### 4. Wait for Services to Initialize

Wait 2-3 minutes for all services to fully start, then verify:

```powershell
# Check HDFS cluster health
docker exec namenode hdfs dfsadmin -report
```

You should see "Live datanodes: 1" in the output.

### 5. Upload Dataset to HDFS

```powershell
# Create directory structure in HDFS
docker exec namenode hdfs dfs -mkdir -p /user/jovyan/input

# Upload the dataset to HDFS
Get-Content .\data\hikari2021.csv -Raw | docker exec -i namenode hdfs dfs -D dfs.replication=1 -put - /user/jovyan/input/hikari2021.csv

# Verify upload
docker exec namenode hdfs dfs -ls /user/jovyan/input/
```

### 6. Access Jupyter Notebook

**Get the Jupyter access token:**
```powershell
# View Jupyter container logs to find the access token
docker logs notebook
```

Look for a line like:
```
    http://127.0.0.1:8888/lab?token=abc123...
```

**Access Jupyter:**
1. Open your web browser
2. Navigate to: `http://localhost:8888`
3. If prompted for a token, use the token from the logs above
4. Go to the `work` folder
5. Open `hikari.ipynb`

### 7. Run the Analysis

Execute the notebook cells in order:

1. **Cell 1**: Import libraries
2. **Cell 2**: Initialize Spark session (connects to Docker Spark cluster)
3. **Cell 3**: Check Spark version
4. **Cell 4**: Load dataset from HDFS
5. **Continue with remaining cells** for data preprocessing, model training, and evaluation

## Web Interfaces

Once running, you can access:

- **Jupyter Notebook**: http://localhost:8888
- **Spark Master UI**: http://localhost:8080
- **HDFS NameNode UI**: http://localhost:9870

## Expected Results

The notebook will:

1. Load and explore the HIKARI 2021 dataset
2. Preprocess the data (handle missing values, encode categorical features)
3. Train a Random Forest classifier with class balancing
4. Evaluate performance using F1-score and confusion matrix
5. Calculate per-class precision, recall, and F1-score

## Stopping the Environment

```powershell
# Stop all services
docker-compose down

# Stop and remove all data (clean slate)
docker-compose down -v
```

## Data Pipeline

1. **Data Ingestion**: CSV file uploaded to HDFS
2. **Data Processing**: PySpark reads from HDFS, handles missing values
3. **Feature Engineering**: String indexing, vector assembly
4. **Model Training**: Random Forest with class weighting
5. **Evaluation**: Multi-class F1-score, confusion matrix analysis

## Machine Learning Details

- **Algorithm**: Random Forest Classifier
- **Features**: All numeric network flow features
- **Target**: Traffic category classification
- **Evaluation**: Weighted F1-score for imbalanced classes
- **Cross-validation**: 80/20 train-test split

## Team

| Name | GitHub |
|------|--------|
| Rochmanu Purnomohadi Erfitra | [@nrcst](https://github.com/nrcst) |
| I Kadek Surya Satya Dharma | [@Suryy16](https://github.com/Suryy16) |
| Muhammad Rajendra Alkautsar Dikna | [@alkadikna](https://github.com/alkadikna) |
| Abiyyu Kumara Nayottama | [@kumara221](https://github.com/kumara221) |


## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test with the Docker environment
5. Submit a pull request

## Citation

If you use the HIKARI-2021 dataset, please cite:

```bibtex
@Article{app11177868,
AUTHOR = {Ferriyan, Andrey and Thamrin, Achmad Husni and Takeda, Keiji and Murai, Jun},
TITLE = {Generating Network Intrusion Detection Dataset Based on Real and Encrypted Synthetic Attack Traffic},
JOURNAL = {Applied Sciences},
VOLUME = {11},
YEAR = {2021},
NUMBER = {17},
ARTICLE-NUMBER = {7868},
URL = {https://www.mdpi.com/2076-3417/11/17/7868},
ISSN = {2076-3417},
ABSTRACT = {The lack of publicly available up-to-date datasets contributes to the difficulty in evaluating intrusion detection systems. This paper introduces HIKARI-2021, a dataset that contains encrypted synthetic attacks and benign traffic. This dataset conforms to two requirements: the content requirements, which focus on the produced dataset, and the process requirements, which focus on how the dataset is built. We compile these requirements to enable future dataset developments and we make the HIKARI-2021 dataset, along with the procedures to build it, available for the public.},
DOI = {10.3390/app11177868}
}
```

**Paper**: Ferriyan, A.; Thamrin, A.H.; Takeda, K.; Murai, J. Generating Network Intrusion Detection Dataset Based on Real and Encrypted Synthetic Attack Traffic. *Applied Sciences* **2021**, *11*, 7868. https://doi.org/10.3390/app11177868

## License

This project is for educational and research purposes.

## Trivia

For those who wonder why the jupyter username is "jovyan" you can rever to this : 
https://github.com/jupyter/docker-stacks/issues/358
