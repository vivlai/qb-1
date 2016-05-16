# QANTA

## Setup
### Dependencies
* Python 3.5 (Qanta does not support anything below Python 3.5)
* Apache Spark 1.6.1
* Vowpal Wabbit 8.1.1 or newer
* All python packages in `requirements.txt`, installable via `pip3 install -r requirements.txt`

### Installation

1. Either copy non_naqt.db to data/questions.db, simlink it, or copy your own questions.db file.

2. Run the script "python util/install_python_packages.py", which will install several python packages you'll need.  (You may need admin access.)

3. Run the script "python util/install_nltk_data.py", which will download some nltk data.  You should *not* use admin access for this script.

4. Download the Illinois Wikifier code (VERSION 2).  Place the data directory in data/wikifier/data and put the wikifier-3.0-jar-with-dependencies.jar in the lib directory http://cogcomp.cs.illinois.edu/page/software_view/Wikifier and put the config directory in data/wikifier/config

## Environment Variables
The majority of QANTA configuration is done through environment variables. Where possible, these
have been set to sensible defaults. Below are all the variables and their default settings.

```python
$ python cli.py env
Printing QANTA Environment Variables
('QB_SPARK_MASTER', 'spark://ec2-52-9-103-244.us-west-1.compute.amazonaws.com:7077')
('QB_STREAMING_CORES', 12)
('QB_QUESTION_DB', '/home/ubuntu/qb/data/questions.db')
('QB_ROOT', '/home/ubuntu/qb')
('QB_GUESS_DB', '/home/ubuntu/qb/data/guesses.db')
```

## Running QANTA
QANTA can be run in two modes: batch or streaming. Batch mode is used for training and evaluating
large batches of questions at a time. Running the batch pipeline is managed by
[Spotify Luigi](https://github.com/spotify/luigi). Luigi is a pure python make-like framework for
running data pipelines. The QANTA pipeline is specified in `qanta/pipeline.py`. Below are the
pre-requisites that need to be met before running the pipeline and how to run the pipeline itself.
Eventually any data related targets will be moved to Luigi and leave only compile-like targets in
the makefile

### Prerequisites
Execute the following commands

```bash
# Generate the makefile
$ python3 cli.py makefile

# Run pre-requisites
$ make prereqs
```

Additionally, you must have Apache Spark running at the url specified in the environment variable
`QB_SPARK_MASTER`

### Running Batch Mode
1. Start the Luigi daemon: `luigid --background --address 0.0.0.0`
2. Run the pipeline: `luigi --module qanta.pipeline AllSummaries --workers 30` (change 30 to number
of concurrent tasks to run at a time)
3. Observe pipeline progress at [http://hostname:8082](http://hostname:8082)

To rerun any part of the pipeline it is sufficient to delete the target file generated by the task
you wish to rerun.

### Running Streaming Mode
Again, Apache Spark needs to be running at the url specified in the environment variable
`QB_SPARK_MASTER`

## Output File
In the directory `data/results/` there are a number of files and folders which are the output of a quiz bowl run. Below is a description of what each file contains:

* `sentence.X.full.final`: the answer for a question given the full question text
* `sentence.X.full.buzz`: For each (question, sentence, token), the guess with whether to buzz and the weight
* `sentence.X.full.perf`: For each (question, sentence, token), performance statistics
* `sentence.X.full.pred`: Prediction weights from vowpal wabbit
