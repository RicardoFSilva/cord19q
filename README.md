cord19q: COVID-19 Open Research Dataset (CORD-19) Analysis
======

![CORD19](https://pages.semanticscholar.org/hs-fs/hubfs/covid-image.png?width=300&name=covid-image.png)

COVID-19 Open Research Dataset (CORD-19) is a free resource of scholarly articles, aggregated by a coalition of leading research groups, covering COVID-19 and the coronavirus family of viruses. The dataset can be found on [Semantic Scholar](https://pages.semanticscholar.org/coronavirus-research) and there is a CORD-19 challenge on [Kaggle](https://www.kaggle.com/allen-institute-for-ai/CORD-19-research-challenge).

The cord19q project builds an index over the CORD-19 dataset to assist with analysis and data discovery. A series of COVID-19 related research topics were explored to identify relevant articles and help find answers to key scientific questions.

### Tasks
A full overview and list of Kaggle CORD-19 Challenge tasks can be found in this [Notebook](https://www.kaggle.com/davidmezzetti/cord-19-analysis-with-sentence-embeddings)

### Installation
You can use Git to clone the repository from GitHub and install it. It is recommended to do this in a Python Virtual Environment. 

    git clone https://github.com/neuml/cord19q.git
    cd cord19q
    pip install .

Python 3.6+ is supported

### Building a model
Download all the files in the Download CORD-19 section on [Semantic Scholar](https://pages.semanticscholar.org/coronavirus-research). Go the directory with the files
and run the following commands.

    cd <download_path>

For each tar.gz file run the following, where $file is the name of the file with .tar.gz removed.

    mkdir $file && tar -C $file -xvzf $file.tar.gz

Once completed, there should be a file name metadata.csv and subdirectories for each data subset with all json articles.

To build the model locally:

    # Download pre-trained study design/attribute models
    # https://www.kaggle.com/davidmezzetti/cord19-study-design/#attribute
    # https://www.kaggle.com/davidmezzetti/cord19-study-design/#design
    # Default location: ~/.cord19/models/attribute, ~/.cord19/models/design

    # Download entry-dates.csv and place in <download path>
    # https://www.kaggle.com/davidmezzetti/cord-19-article-entry-dates/output

    # Convert csv/json files to SQLite
    python -m cord19q.etl <download_path>

    # Can optionally use pre-trained vectors
    # https://www.kaggle.com/davidmezzetti/cord19-fasttext-vectors#cord19-300d.magnitude
    # Default location: ~/.cord19/vectors/cord19-300d.magnitude
    python -m cord19q.vectors

    # Build embeddings index
    python -m cord19q.index

The model will be stored in ~/.cord19

### Building a report file
A report file is simply a markdown file created from a list of queries. An example:

    python -m cord19q.report tasks/risk-factors.txt

Once complete a file named tasks/risk-factors.md will be created.

### Running queries
The fastest way to run queries is to start a cord19q shell

    cord19q

A prompt will come up. Queries can be typed directly into the console.

### Tech Overview
The tech stack is built on Python and creates a sentence embeddings index with FastText + BM25. Background on this method can be found in this [Medium article](https://towardsdatascience.com/building-a-sentence-embedding-index-with-fasttext-and-bm25-f07e7148d240) and an existing repository using this method [codequestion](https://github.com/neuml/codequestion).

The model is a combination of the sentence embeddings index and a SQLite database with the articles. Each article is parsed into sentences and stored in SQLite along with the article metadata. FastText vectors are built over the full corpus. The sentence embeddings index only uses COVID-19 related articles, which helps produce more recent and relevant results. 

Multiple entry points exist to interact with the model.

- cord19q.report - Builds a markdown report for a series of queries. For each query, the best articles are shown, top matches from those articles and a highlights section which shows the most relevant sections from the embeddings search for the query.
- cord19q.query - Runs a single query from the terminal
- cord19q.shell - Allows running multiple queries from the terminal
