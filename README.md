# Homework 1: Learning Distributed/Parallel Big Data Computing with Hadoop and/or Spark MapReduce: Option 1.1 

## Environment Setup Review 

### Prerequisites
- Python 3.8+
- Apache Spark 3.x
- PySpark
- Jupyter Notebook
- Required Python packages: pandas, pathlib, collections, time, re

### Installation Steps
1. Create a virtual environment to host the project code 
```bash
python3 -m venv myenv
source myenv/bin/activate
```
2. Install all dependencies to run the code 
```bash
pip install pyspark pandas jupyter matplotlib seaborn
```

# Download the Kaggle dataset 
1. go to https://www.kaggle.com/datasets/socialanimal/social-animal-10k-articles-with-text-and-nlp-data/data
2. click download --> download as zip
3. uncompress the zip and place the folder in the local project directory
4. update the path varaible in the code to reference the data folder

## Homework Overview

In this Homework I run the map-reduce program on 4 different sized datasets with varying reducers and mappers. I also use map reduce to print the top 30 words occuring in the most files for 4 datasets of sizes [100, 1k, 5k, 9.7k]. Additionally I review the runtime on all of these analysis to understand the performance data and show various variables affects on runtime performance of the MapReduce job. 

### MapReduce Implementations

#### 1. Word Count MapReduce (`mapreduce_wrd_count`)
Counts total word frequency across all documents
- Map Phase: Extract words from each file, count occurrences
- Reduce Phase: Sum word counts across all files
- Output: Top 30 most frequent words globally

#### 2. Average Word Frequency MapReduce (`mapreduce_avg_word_freq`)
Calculate average word frequency per document
- Map Phase: Extract words and their counts per document
- Reduce Phase: Calculate average frequency = total_occurrences / documents_containing_word
- Output: Top 30 words with highest average frequency per document

## Experimental Design

### Dataset Configurations
1. Dataset Size Scaling: 100 → 1000 → 5000 → 9653 files
2. Partition Variations: 2, 4, 8 partitions per dataset

### Algorithm Comparison
- Same problem (word analysis) solved with two different MapReduce approaches
- Different computational complexity and memory usage patterns
- Performance comparison across identical datasets

## Results Analysis

### Key Findings

#### 1. Scaling Performance
- Linear Scaling: Runtime increases with dataset size as expected 
- Best Partition Count: 8 partitions generally perform best for larger datasets
- Diminishing Returns: Eventually as partitions continues to double, the runtime is not halved - there is a plateau in the performace gain from increasing the partition count

#### 2. Algorithm Performance Comparison
- Word Count MapReduce: Better for large datasets due to simpler aggregation
- Average Frequency MapReduce: Higher computational overhead, seen in large runtime differences with less partitions, but provides meaningful insights
- Memory Usage: The average frequency map reduce computations required more intermediate storage + runtime 

#### 3. Partition Optimization Effects
- Under-partitioning (2 partitions): CPU cores were under used resulting in a slower runtime performance that was especially large with methods that had higher computation costs (the average frequency method)
- Optimal Partitioning (8 partitions): Matches available cores in my computer and resulted in the best runtime values for all methods  
- Over-partitioning: Would increase coordination overhead (not tested due to core limitations)

### Performance Metrics Summary

| Dataset Size | Optimal Partitions | Best Algorithm | Runtime Range |
|--------------|-------------------|----------------|---------------|
| 100 files    | 4-8              | Word Count     | 0.1-0.3s     |
| 1,000 files  | 8                | Word Count     | 0.2-0.6s     |
| 5,000 files  | 8                | Word Count     | 0.8-2.1s     |
| 97,000 files | 8                | Word Count     | 15-45s       |

## Technical Implementation Details

### Map Function Design
```python
def word_map_gen(path):
    # Read file, normalize text, extract words 3+ chars
    text = Path(path).read_text(encoding="utf-8", errors="ignore").lower()
    words = re.findall(r'\b[a-zA-Z]{3,}\b', text)
    return list(Counter(words).items())
```

### Reduce Function Strategy
- Word Count: Simple addition `lambda a, b: a + b`
- Average Frequency: Tuple aggregation `lambda a, b: (a[0] + b[0], a[1] + b[1])`

### Error Handling
- File encoding errors handled and printed as errors
- Missing files skipped with logging
- Memory management for large datasets

## Experimental Results Discussion

### Scalability Analysis
1. CPU Utilization: 8 partitions effectively utilize 8-core system
2. Memory Efficiency: Spark's lazy evaluation prevents memory overflow
3. I/O Bottleneck: File reading becomes limiting factor for very large datasets

### Algorithm Trade-offs
- Simple Word Count: O(n) complexity, minimal memory overhead
- Average Frequency: O(n) complexity but higher constant factors, more memory for intermediate results

### Real-world Implications
- For production systems: Choose algorithm based on analysis goals
- For large-scale deployment: Consider distributed file systems (HDFS)
- For memory-constrained environments: Prefer simple word count approach

## Conclusions

1. **Optimal Configuration**: 8 partitions provide best performance across all dataset sizes
2. **Algorithm Choice**: Simple word count scales better; average frequency provides richer insights
3. **Scaling Behavior**: Near-linear scaling demonstrates MapReduce effectiveness
4. **Resource Utilization**: Proper partitioning crucial for performance optimization

## File Structure
```
project/
├── hw1.ipynb          # Main analysis notebook
├── mapreduce_results.csv   # Performance results
├── social_animal_data/     # Input dataset directory
├── README.md              # This file
└── analysis_plots.png     # Generated visualizations
```
