# Text Deduplication using LSH

This project implements a configurable text deduplication system using Locality-Sensitive Hashing (LSH) techniques. Designed for parameter optimization, it supports systematic benchmarking of different parameter combinations (band size, threshold, n-gram ranges, etc.) to identify optimal deduplication settings. The system processes Wiki40B dataset to identify duplicates using three complementary approaches:

1. **MinHash** - Efficient Jaccard similarity estimation  
2. **SimHash** - Cosine similarity via compact binary fingerprints  
3. **Bit-Sampling** - Tunable similarity thresholds for precision/recall balance  

Key features:  
✅ Automated experiment tracking with parameter hashing  
✅ Performance comparison across 12+ parameter dimensions  
✅ Comprehensive metrics (duplicate rates, time usage, memory usage)  

## Prerequisites

- Python 3.7 or higher
- required packages
  ```bash
  pip install pandas numpy scikit-learn datasets tqdm
  ```

## Dataset Preparation and Expected Output

1. **Download Wiki-40B/en Dataset**
Use Hugging Face's datasets library to download and prepare the data:

   ```python
   from datasets import load_dataset

   # Download test split
   test_ds = load_dataset("wiki40b/en", split="test", trust_remote_code=True)
   test_ds.save_to_disk("raw_data/test")  # Save in Arrow format

   # Download validation split
   val_ds = load_dataset("wiki40b/en", split="validation", trust_remote_code=True) 
   val_ds.save_to_disk("raw_data/validation")
   ```

2. **Verify Directory Structure**
Your raw data directory should match this structure:

   ```
   raw_data/
   ├── test/
   │   ├── data-00000-of-00002.arrow
   │   ├── data-00001-of-00002.arrow
   │   ├── dataset_info.json
   │   └── state.json
   └── validation/
       ├── data-00000-of-00002.arrow  
       ├── data-00001-of-00002.arrow
       ├── dataset_info.json
       └── state.json
   ```

3. **Configure Paths**
Modify these paths in ExperimentConfig:

   ```python
   class ExperimentConfig:
       raw_data_root: str = "path/to/your/raw_data"  # Point to parent "raw_data" folder
       processed_dir: str = "path/to/your/processed_data" 
   ```


4. **Expected Output**
   Run these files by clicking the `Run All` bar.
   The expected output file structure is
   ```
   path/to/your/processed_data/
   ├── preprocessed.csv               # Cleaned text
   ├── experiments/  
       ├── exp_<parameter_hash>/          # Experiment-specific
       │   ├── signatures_<hash>.parquet  # SimHash signatures
       |   ├── candidates_<hash>.csv       # Candidate pairs
       │   └── performance.json           # Resource usage logs
       └── experiment_summary.csv  # summary
   ```


## Implementation Details

### MinHash


### SimHash

1. **Preprocessing**
  ​​Process​​:
     - Remove structural markers (`_START_xxx`)
     - Convert to lowercase
     - Remove punctuation
     - Normalize whitespace
    ​​Output​​: `path/to/your/processed_data/preprocessed.csv`

1. **SimHash Signature Generation**
  ​​Process​​:
    - Employs `HashingVectorizer` with n-grams to avoid vocabulary memory bottlenecks
    - Applies dynamic TF-IDF weighting across text chunks
    - Parallel processing in chunks
    ​Output​​: `path/to/your/processed_data/exp_<hash>/signatures_<hash>.parquet`

1. **LSH Candidate Generation**
​   ​Process​​:
      - Banded signature hashing
      - Inverted index construction
      - Similarity threshold filtering
​     ​Output​​: `path/to/your/processed_data/exp_<hash>/candidates_<hash>.csv`

### Bit-Sampling

## Evaluation Metrics & Analysis

### Experiment Summary Schema

The `experiment_summary.csv` file contains these key columns
Method-specific metrics


| Column | Type | Description | Calculation Source | 
| --- | --- | --- | --- | 
| experiment_id | str | Unique parameter configuration hash | config.param_hash | 
| timestamp | datetime | Experiment execution timestamp | System clock | 
| num_bands | int | Number of LSH bands used | ExperimentConfig.num_bands | 
| band_size | int | Bits per LSH band | ExperimentConfig.band_size | 
| threshold | float | Similarity cutoff threshold (0.7-1.0) | ExperimentConfig.threshold | 
| simhash_time_sec | float | Time spent generating SimHash signatures | Stage performance monitoring | 
| lsh_time_sec | float | Time spent generating candidate pairs | Stage performance monitoring | 
| peak_memory_mb | int | Maximum RAM usage during experiment | psutil memory tracking | 
| total_candidates | int | Primary generated candidates by banding only | generate candidates |
| valid_candidates | int | Candidates verified by Jaccard/Hamming Distance | genertae candidates |
| total_duplicate_ratio | float | Percentage of duplicated documents | Union-Find component analysis | 
| cross_set_ratio | float | Documents duplicated across val/test sets | Component source classification | 
| intra_val_ratio | float | Duplicated documents within validation set | Component source classification | 
| intra_test_ratio | float | Duplicated documents within test set | Component source classification | 

### Key Features:

1. ​​Complete Experiment Overview​​

   - Combines configuration parameters with runtime metrics and quality indicators
​​
2. Traceability​​

   - Explicit data source mapping:
   Config parameters → `ExperimentConfig`
   Timing metrics → Performance monitors
   Duplication metrics → Union-Find analysis
