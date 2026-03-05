# SIGMOD 2026 Round 4 - SEMA

This repository contains the technical report and experimental scripts for reproducible research on SEMA.

## Table of Contents

1. [Appendix Documentation](#appendix-documentation)
2. [Experimental Scripts](#experimental-scripts)

## Appendix Documentation

The `SemaAppendix.pdf` file contains supplementary materials that could not be included in the main paper due to space constraints. This appendix includes:

- **20 Designed Queries and Variants**: A comprehensive collection of 20 SQL queries (Q1-Q20) designed to evaluate semantic analytics capabilities across different query patterns:
  - **Single-Operators**: Queries Q1-Q5 that demonstrate single semantic filter/map operator.
  - **Multiple-Filters**: Queries Q6-Q10 that demonstrate consective semantic filters.
  - **Map-Filter**: Queries Q11-Q13 that perform semantic mapping followed by filtering.
  - **Map-Map**: Queries Q14-Q15 that chain multiple semantic mapping operations.
  - **Filter-Map**: Queries Q16-Q18 that apply semantic filtering followed by mapping operations.
  - **Filter-Aggregate**: Queries Q19-Q20 that combine filtering with semantic aggregation.



- **Algorithm Pseudocode**: Detailed pseudocode for the SEMA algorithms, providing implementation-level insights into the semantic analytics framework.

## Experimental Scripts

### Prerequisites

Before running the experiments, you need to extract the binary executables:

```bash
# Extract SEMA binary
cd build/
unzip sema.zip

# Extract Flock binary  
unzip flock.zip

# Install Lotus
conda create -n lotus python=3.10 -y
conda activate lotus
pip install lotus-ai
```

### Datasets
We use datasets from bird benchmark **Train Set**, which you can get from https://bird-bench.github.io/.
We transform sqlite format of datasets to duckdb format, which make it more compatible with Sema and Flock.
For lotus, we transform them into parquet.

### Running Experiments

After extraction, you can directly execute the binary files to run the experiments:

```bash
# Run SEMA experiments
./sema

# Run Flock experiments
./flock

# Run Lotus experiments
conda activate lotus
python3 q1.py
```

### SEMA
#### Configuration
For sema, you can set different configurations to enable/disable different optimizations.

Sema supports OPENAI-compatible API formats.
```bash
# Set SEMA llm models
set llm_model = 'google/gemma-3-12b-it';
set llm_url = 'https://openrouter.ai/api/v1/chat/completions';
set llm_api_key = 'Your api key here';
```

To see details, you can enable logging and set different logging levels to show different details.
```bash
# Enable logging
set enable_logging=true;
set logging_storage=stdout;
set logging_level=trace; # (Trace: most detailed, DEBUG, INFO)
```

#### An example
To test your llm api connectivity, here's a simple example:
```bash
# A running example
create table test (cap varchar, country varchar);
insert into test values ('Moscow', 'Russia'), ('Beijing', 'Russia'), ('Shanghai', 'China'), ('London', 'UK');
select * from test where s'Is {cap} the capital of {country}?';
```

To see token costs and profile information, you can use profile insturction
```bash
explain analyze select * from test where s'Is {cap} the capital of {country}?';
```


#### Batch Prompt
To enable batch prompt, you can set:
```bash
set semantic_batch_size = 4;
set semantic_voting_rounds = 1;
```
You can use explain to see logical plan where batch_size and voting rounds will be shown.

### AQE
To enable AQE, you can set:
```bash
# Only work for Q6-Q10
set enable_semantic_filter_multiplexer = true; # open AQE
set semantic_filter_accuracy_threshold = 0.9; # acc threshold
set semantic_filter_latency_first = true; # false for cost first
set semantic_filter_batch_size=4; # batch for AQE (not same as semantic_batch_size which is for non-AQE)
set semantic_filter_voting_rounds; # voting rounds for AQE
```

To check details of AQE(expression exploration, path exploration, pareto-frontier, path selection), you can profile select query like:
```bash
explain analyze SELECT id AS id
FROM (SELECT id, Translated_Review
      FROM user_reviews ur JOIN playstore p
      ON ur.App = p.App AND p.Type = 'Free'
      AND ur.Sentiment_Subjectivity IS NOT NULL
      AND ur.Translated_Review != 'nan'
      AND length(ur.Translated_Review) < 255
      AND CAST(ur.Sentiment_Subjectivity AS REAL) < 0.3)
WHERE s'{Translated_Review} is a valid user review (which is a complete sentence and has informative content)'
AND s'{Translated_Review} is a positive or neutral user review'
AND s'{Translated_Review} shows that this app is very creative and innovative';
```

### Flock

We have modified Flock to support vLLM, providing better model compatibility and performance.
To check more docs, please visit https://dais-polymtl.github.io/flock/docs/what-is-flock

```bash
# View available models
get models
```

### Results Analysis Scripts

The experimental results are organized in a consistent structure across all experiment categories in `sema_experiment_scripts/`. Each experiment folder follows the same organizational pattern:

#### Directory Structure
- **`labels/`**: Contains ground truth labels generated using 27B parameter models
  - **`all/`**: Stores the complete dataset without semantic filtering
  - **`label/`**: Contains filtered labels generated by 27B models for evaluation
- **`results/`**: Contains experimental results obtained using 12B parameter models
- **`calculate_metrics.py`**: Automated script for computing performance metrics

#### Experimental Methodology
- **SEMA & Lotus**: Results are averaged across 3 independent runs to ensure statistical reliability
- **Flock**: Single run results (due to computational constraints)
- **Metrics**: The `calculate_metrics.py` script automatically computes accuracy, precision, recall, F1-score, and other relevant performance indicators

#### Supported Experiment Categories
1. **Single Operators** (Q1-Q5): Basic semantic filter/map operations
2. **Multiple Filters** (Q6-Q10): Consecutive semantic filtering operations  
3. **Map-Filter** (Q11-Q13): Semantic mapping followed by filtering
4. **Map-Map** (Q14-Q15): Chained semantic mapping operations
5. **Filter-Map** (Q16-Q18): Semantic filtering followed by mapping
6. **Filter-Aggregate** (Q19-Q20): Combined filtering with semantic aggregation

Each category includes corresponding query files for SEMA (SQL), Lotus (Python), and Flock (SQL) implementations, along with their respective result analysis scripts.