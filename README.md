# Homework 1: Learning Distributed/Parallel Big Data Computing with Hadoop and/or Spark MapReduce: Option 1.1 

Report in README

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

#### 3. Top 30 words occuring most in a document (`mapreduce_doc_frequency`)
Calculate the top 30 words occuring in most documents / files 
- Map Phase: Processes each document to build a set of unique words across all documents 
- Reduce Phase: Calculates how many documents each word appears in 
- Output: A sorted list that is 30 words long containing the most common words across all documents

### Dataset Configurations
1. Dataset Size Scaling: 100 → 1000 → 5000 → 9653 files
2. Partition Variations: 2, 8, 32 partitions per dataset

### Algorithm Comparison
- Same problem (word document frequency) solved with two different MapReduce approaches ( one using 1 mapreduce job and one using 3)
- Additionally I also used map reduce to solve two other problems being printing the word count and average word count per document problems.
- Between all 4 mapreduce algorithm I saw strong differences between memory usage patterns and overall runtime. 
- I also completed performance comparison across identical datasets with varying partitions

Document Frequency Analysis: 
I compared two mapreduce algorithms both solving the same problem, and varied how many mapreduce jobs they both compute. One had 1 job while hte other had 3. Very interestingly the 3-job map-reduce algorithm consistently yeiled lower runtimes than the 1-job algorithm. In my single job algo i use a python set in order to gather unqiue words, and convert it to a list. I beleive that this is causing increased memory overhead that outweights the benefit of using less map reduce jobs. Additionally in the 3-job algo I use spark's optimized functions which could also help in the improved runtime. The runtime outputs (in seconds) can be seen below: 

| Approach  / DS size            | 100   | 1000  | 5000  | 9653  |
|:----------------------|------:|------:|------:|------:|
| Single-Job MapReduce  | 0.089 | 0.125 | 0.338 | 0.551 |
| Three-Job MapReduce   | 0.134 | 0.194 | 0.538 | 0.924 |

I additionally completed an analysis of my other two methods that I made in order to understand how partitions / dataset size affect runtime: 
## Word Count / Average Frequency Algorithms Runtime Analysis
The following graph depicts the realaitonship between the number of partitions used and the final run-time based on the dataset size:
![](/graph1.png)

Across all four datasets, the runtime appears lowest around 8 partitions with 2 closly following in smaller datasets and looking under-parallelized in larger ones. Additionally across the board we can see that 32 partitions results in a heavy increase to runtime. I hypothesize that moving from 2→8 partitions improves core utilization for the datasets greater than or equal to 1k in size. For the 100, super small dataset, I believe that using too many cores splits up data too much, when just 2 would suffice / be optimal it seems. This graph also shows that jumping to 32 partitions introduces higher scheduling and shuffle overhead with many tiny tasks. The “Avg Frequency” line is consistently a bit higher than “Word Count,” which I suspect comes from an extra computations in the algorithm. The effect seems more pronounced on the larger datasets, perhaps because overhead amortizes differently as data volume scales. Overall, it seems that 8 partitions is the most optimal. 

Next I reviewed how the dataset size as a whole affects the runtime: 
![](/graph2.png)

For each fixed partition count, the curves look roughly linear, in some cases exponential, in dataset size. This seems consistent with O(n)-like behavior for both algorithms. A clear trend we can see in these graphs is that as the dataset size increases the runtime also increases, no matter what partition is being used. This graph also shows that 8 partitions seems to be the most optimal, potentially using available cores better and minimizing overhead.  With only 2 partitions, large datasets seem to suffer. With 32 partitioins, small datasets may pay a disproportionate scheduling/serialization cost. The small gap where “Avg Frequency” runs slower than “Word Count” can also be seen again. 

I then competed a complete runtime analysis that compares the runtime for each dataset at all defined partition amounts: 
![](/graph3.png)

This bar chart further exhibits the pattern: it becomes clear that the 8-partition map reduce configuration is the most efficient. Most certainly this is the case for the datasets tested that have over or equal to 1k files. In the 100 file dataset case, we see that 2 partitions slightly outperforms 8, this is likely due to increased overhead / splitting of tasks that leveraging all 8 cores to process such a small dataset does. It seems that overall the 2-partitions cases seem to struggle on bigger datasets, when it comes to comparing it to the best performing, 8 partitions. Additionally across the board it seems that 32 partitions drastically performs the wrost and slows the runtime performance. As my computer has 8 cores I beleive that 32 partitoins may introduce extra shuffle and task-setup costs that don’t pay off for datasets of the sizes tests. 

Additionally we can see that the word count algorithm has a consistently better runtime than the average frequency algorithm. I’d attribute this to differences in constant factors such as aggregation steps or object creation. Taken together, the results seem consistent with near-linear scaling in input size, with runtime primarily determined by the partition number and dataset size.

#### 1. Scaling Performance
- Linear Scaling: Runtime increases with dataset size as expected 
- Best Partition Count: 8 partitions generally perform best for larger datasets
- Diminishing Returns: Eventually as partitions continues to double, the runtime is not halved - there is a plateau in the performace gain from increasing the partition count. In an extreme case, which is seen using 32 partitions, we see increasing partitions significantly increase runtime as well. 

#### 2. Algorithm Performance Comparison
- Word Count MapReduce: Better for large datasets in terms of running time likley due to simpler aggregation
- Average Frequency MapReduce: Seems to have a higher computational overhead, seen in large runtime differences with less partitions, but provides meaningful insights

#### 3. Partition Optimization Effects
- Under-partitioning (2 partitions): CPU cores were under used resulting in a slower runtime performance that was especially slow with larger datasets
- Optimal Partitioning (8 partitions): Matches available cores in my computer and resulted in the best runtime values for all methods  
- Over-partitioning (32 partitions): Seemed to increase overhead by splitting up calls too much - significant runtime increas. 

### Performance Metrics Summary

| Dataset Size | Optimal Partitions | Best Algorithm | Runtime Range |
|--------------|-------------------|----------------|---------------|
| 100 files    | 2-8              | Word Count     | 0.1-0.3s     |
| 1,000 files  | 8                | Word Count     | 0.2-0.6s     |
| 5,000 files  | 8                | Word Count     | 0.8-2.1s     |
| 97,000 files | 8                | Word Count     | 15-45s       |

## Map Reduce to solve printing top 30 words occuring in the most files problem: 

This was implemented two times (discuessed above) 
Here are the results: 
100 files
   1. 'the         'appears in   97 of files 100)
   2. 'and         'appears in   93 of files 100)
   3. 'for         'appears in   87 of files 100)
   4. 'with        'appears in   81 of files 100)
   5. 'that        'appears in   78 of files 100)
   6. 'this        'appears in   76 of files 100)
   7. 'from        'appears in   74 of files 100)
   8. 'have        'appears in   70 of files 100)
   9. 'are         'appears in   69 of files 100)
  10. 'will        'appears in   67 of files 100)
  11. 'was         'appears in   65 of files 100)
  12. 'has         'appears in   61 of files 100)
  13. 'not         'appears in   54 of files 100)
  14. 'all         'appears in   54 of files 100)
  15. 'their       'appears in   53 of files 100)
  16. 'but         'appears in   53 of files 100)
  17. 'one         'appears in   53 of files 100)
  18. 'which       'appears in   53 of files 100)
  19. 'also        'appears in   51 of files 100)
  20. 'can         'appears in   48 of files 100)
  21. 'you         'appears in   47 of files 100)
  22. 'time        'appears in   47 of files 100)
  23. 'they        'appears in   46 of files 100)
  24. 'there       'appears in   45 of files 100)
  25. 'more        'appears in   44 of files 100)
  26. 'other       'appears in   44 of files 100)
  27. 'first       'appears in   43 of files 100)
  28. 'new         'appears in   43 of files 100)
  29. 'out         'appears in   43 of files 100)
  30. 'over        'appears in   43 of files 100)

1000 files
   1. 'the         'appears in  972 of files 1000)
   2. 'and         'appears in  951 of files 1000)
   3. 'for         'appears in  868 of files 1000)
   4. 'with        'appears in  826 of files 1000)
   5. 'that        'appears in  756 of files 1000)
   6. 'from        'appears in  688 of files 1000)
   7. 'this        'appears in  685 of files 1000)
   8. 'are         'appears in  631 of files 1000)
   9. 'has         'appears in  626 of files 1000)
  10. 'have        'appears in  619 of files 1000)
  11. 'was         'appears in  582 of files 1000)
  12. 'will        'appears in  571 of files 1000)
  13. 'their       'appears in  532 of files 1000)
  14. 'also        'appears in  501 of files 1000)
  15. 'all         'appears in  492 of files 1000)
  16. 'but         'appears in  483 of files 1000)
  17. 'not         'appears in  482 of files 1000)
  18. 'can         'appears in  481 of files 1000)
  19. 'one         'appears in  475 of files 1000)
  20. 'more        'appears in  471 of files 1000)
  21. 'which       'appears in  470 of files 1000)
  22. 'they        'appears in  449 of files 1000)
  23. 'you         'appears in  428 of files 1000)
  24. 'about       'appears in  418 of files 1000)
  25. 'who         'appears in  409 of files 1000)
  26. 'other       'appears in  404 of files 1000)
  27. 'after       'appears in  402 of files 1000)
  28. 'out         'appears in  400 of files 1000)
  29. 'said        'appears in  392 of files 1000)
  30. 'time        'appears in  391 of files 1000)

5000 files
   1. 'the         'appears in 4872 of files 5000)
   2. 'and         'appears in 4758 of files 5000)
   3. 'for         'appears in 4407 of files 5000)
   4. 'with        'appears in 4112 of files 5000)
   5. 'that        'appears in 3892 of files 5000)
   6. 'this        'appears in 3480 of files 5000)
   7. 'from        'appears in 3477 of files 5000)
   8. 'are         'appears in 3214 of files 5000)
   9. 'has         'appears in 3193 of files 5000)
  10. 'have        'appears in 3135 of files 5000)
  11. 'was         'appears in 2938 of files 5000)
  12. 'will        'appears in 2802 of files 5000)
  13. 'their       'appears in 2619 of files 5000)
  14. 'also        'appears in 2592 of files 5000)
  15. 'more        'appears in 2467 of files 5000)
  16. 'not         'appears in 2426 of files 5000)
  17. 'one         'appears in 2422 of files 5000)
  18. 'but         'appears in 2395 of files 5000)
  19. 'which       'appears in 2386 of files 5000)
  20. 'all         'appears in 2377 of files 5000)
  21. 'can         'appears in 2348 of files 5000)
  22. 'they        'appears in 2296 of files 5000)
  23. 'about       'appears in 2108 of files 5000)
  24. 'new         'appears in 2099 of files 5000)
  25. 'been        'appears in 2065 of files 5000)
  26. 'you         'appears in 2063 of files 5000)
  27. 'other       'appears in 2059 of files 5000)
  28. 'after       'appears in 2052 of files 5000)
  29. 'who         'appears in 2041 of files 5000)
  30. 'its         'appears in 2020 of files 5000)

9653 files
   1. 'the         'appears in 9423 of files 9653)
   2. 'and         'appears in 9155 of files 9653)
   3. 'for         'appears in 8505 of files 9653)
   4. 'with        'appears in 7977 of files 9653)
   5. 'that        'appears in 7538 of files 9653)
   6. 'from        'appears in 6778 of files 9653)
   7. 'this        'appears in 6687 of files 9653)
   8. 'has         'appears in 6227 of files 9653)
   9. 'are         'appears in 6132 of files 9653)
  10. 'have        'appears in 6103 of files 9653)
  11. 'was         'appears in 5645 of files 9653)
  12. 'will        'appears in 5439 of files 9653)
  13. 'their       'appears in 5081 of files 9653)
  14. 'also        'appears in 5061 of files 9653)
  15. 'more        'appears in 4724 of files 9653)
  16. 'not         'appears in 4687 of files 9653)
  17. 'one         'appears in 4683 of files 9653)
  18. 'which       'appears in 4636 of files 9653)
  19. 'but         'appears in 4632 of files 9653)
  20. 'can         'appears in 4560 of files 9653)
  21. 'all         'appears in 4531 of files 9653)
  22. 'they        'appears in 4365 of files 9653)
  23. 'new         'appears in 4120 of files 9653)
  24. 'about       'appears in 4065 of files 9653)
  25. 'been        'appears in 4058 of files 9653)
  26. 'after       'appears in 4042 of files 9653)
  27. 'who         'appears in 3990 of files 9653)
  28. 'other       'appears in 3984 of files 9653)
  29. 'you         'appears in 3957 of files 9653)
  30. 'its         'appears in 3924 of files 9653)

## Conclusions

1. **Optimal Configuration**: 8 partitions provide best performance across all dataset sizes
2. **Algorithm Choice**: Simple word count scales better; average frequency provides richer insights
3. **Scaling Behavior**: Near-linear scaling demonstrates MapReduce effectiveness
4. **Resource Utilization**: Proper partitioning crucial for performance optimization

# Resources & Citations

* Apache Spark. (n.d.). *RDD Programming Guide*. Retrieved from [https://spark.apache.org/docs/latest/rdd-programming-guide.html](https://spark.apache.org/docs/latest/rdd-programming-guide.html)

* Apache Spark. (n.d.). *PySpark RDD API Documentation*. Retrieved from [https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.RDD.html](https://spark.apache.org/docs/latest/api/python/reference/api/pyspark.RDD.html)

* Apache Spark. (n.d.). *Spark Programming Guide – Word Count Example*. Retrieved from [https://spark.apache.org/examples.html](https://spark.apache.org/examples.html)

* Stack Overflow. (2014). *PySpark Word Count Implementation*. Retrieved from [https://stackoverflow.com/questions/22350722/word-count-program-in-spark](https://stackoverflow.com/questions/22350722/word-count-program-in-spark)

* Apache Spark. (n.d.). *Spark Configuration Documentation*. Retrieved from [https://spark.apache.org/docs/latest/configuration.html](https://spark.apache.org/docs/latest/configuration.html)

* Stack Overflow. (2016). *Spark Performance Tuning – Number of partitions in RDD and performance*. Retrieved from [https://stackoverflow.com/questions/35800795/number-of-partitions-in-rdd-and-performance-in-spark](https://stackoverflow.com/questions/35800795/number-of-partitions-in-rdd-and-performance-in-spark)

* Dean, J., & Ghemawat, S. (2008). *MapReduce: Simplified data processing on large clusters*. Communications of the ACM, 51(1), 107–113. [https://doi.org/10.1145/1327452.1327492](https://doi.org/10.1145/1327452.1327492)

* Apache Spark. (n.d.). *Cluster Mode Overview*. Retrieved from [https://spark.apache.org/docs/latest/cluster-overview.html](https://spark.apache.org/docs/latest/cluster-overview.html)

* Zaharia, M., Chowdhury, M., Das, T., Dave, A., Ma, J., McCauley, M., ... & Stoica, I. (2016). *Apache Spark: A unified engine for big data processing*. Communications of the ACM, 59(11), 56–65. [https://doi.org/10.1145/2934664](https://doi.org/10.1145/2934664)
