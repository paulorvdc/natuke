# ImpNatUKE: Improving Natural Product Knowledge Extraction from Academic Literature on NatUKE Benchmark

Welcome to ImpNatUKE! Here we present usability explanations based on the NatUKE benchmark as well as the additions we made and how to use them. 

- [Usability](https://github.com/aksw/ImpNatUKE#usability)
- [Submodules](https://github.com/aksw/ImpNatUKE#submodules)
- [Benchmark](https://github.com/aksw/ImpNatUKE#benchmark)
- [Data](https://github.com/aksw/ImpNatUKE#data)
- [Models](https://github.com/aksw/ImpNatUKE#models)
- [Results](https://github.com/aksw/ImpNatUKE#results)
- [License](https://github.com/aksw/ImpNatUKE#license)
- [Wiki](https://github.com/aksw/ImpNatUKE#wiki)

## Usability

Original NatUKE source code explanation for understanding, running and evaluating experiments.

### Source code breakdown

**(this source code breakdown is also the regular flow of execution from natuke)**

Here we explain all the source code in the repository and the order in which to execute them:
1. ```clean_pdfs.ipynb```: load pdfs considering database and prepare two dataframes to be used further;
2. ```phrases_flow.py```: load texts dataframe and separate the texts into 512 tokens phrases;
3. ```topic_generation.ipynb```: load phrases dataframe and create a topic cluster using BERTopic [4];
4. ```topic_distribution.ipynb```: load BERTopic model the phrases dataframe and distributes the topics filtering according to an upper limit of the proportion and outputs the dataframe;
5. ```hin_generation.ipynb```: load the filtered topics dataset and paper information to generate the usable knowledge graph;
6. ```knn_dynamic_benchmark.py```: runs the experiments using the generated knowledge graph, considering the parametrers set on the main portion of the code; 
**(option for BiKE challenge (https://aksw.org/bike/))** ```knn_dynamic_benchmark_splits.py```: runs the experiments using the splits tailored for the BiKE (First International Biochemical Knowledge Extraction Challenge: http://aksw.org/bike/) challenge;
7. ```dynamic_benchmark_evaluation.py```: generates hits@k and mrr metrics for the experiments, allowing different parameters to be set for the algorithms used as well as the metrics;
8. ```execution_time_processer.py```: processes the dynamically ```.txt``` generated by ```knn_dynamic_benchmark.py``` experiments into a dataframe of execution times;
9. ```metric_graphs.py```: with the metric results and execution times allows the generation of personalized graphs;
* ```natuke_utils.py```: contains the source for the: methods; split algorithms; similar entity prediction; and metrics.
* ```exploration.ipynb```: used to explore data, as for the quantities of each property.

### ImpNatUKE source code breakdown

Here we explain the source code changes from NatUKE and where they apply:
1. ```clean_grobid.py```: cleans the processed ```.xml``` files outputted from GROBID (https://github.com/kermitt2/grobid) into ```.txt``` files that will be used to generate the embeddings. It substitutes ```clean_pdfs.ipynb``` and ```phrases_flow.py``` for the GROBID experiments;
2. ```clean_nougat.py```: cleans the processed ```.md``` files outputted from Nougat (https://github.com/facebookresearch/nougat) into ```.txt``` files that will be used to generate the embeddings. It substitutes ```clean_pdfs.ipynb``` and ```phrases_flow.py``` for the Nougat experiments;
3. ```old_processed_pdfs.py```: saves the PyMuPdf extractions into ```.txt``` files for usage in LLM embedding generation;
4. ```hin_save_splits.py```: loads the embeddings file and connects it with the topics and other data to generate the networkx Graph and saves the train/test split files  on the BiKE challenge format. It substitutes ```hin_generation.ipynb``` for LLM experiments;
5. ```hin_bert_splits.py```: loads the processed ```.txt``` files from either GROBID or Nougat and uses the original DistilBERT model to generate the embeddings before connecting it to the topics and other data to generate the networkx Graph and saving the train/test split files on the BiKE challenge format. It substitutes ```hin_generation.ipynb``` for BERT experiments with the new PDF file processors;
6. ```tsne_plot.ipynb```: generates 2D tsne plots for the embedding models for a visual comparation of the reduced embeddings.

### Instalation and running

All experiments were tested with a conda virtual environment of Python 3.8. With conda installed the virtual envs should be created with:
```
conda create --name [name] python=3.8
```
Install the requirements:
```
cd natuke
conda activate [name]
pip install -r requirements.txt
```

**GraphEmbeddings**

GraphEmbeddings submodule based on https://github.com/shenweichen/GraphEmbedding but the used algorithms works with tf 2.x

To install this version of GraphEmbeddings run:
```
cd GraphEmbeddings
python setup.py install
```

To run the benchmark execute ```knn_dynamic_benchmark.py``` after adding the repository with the data and adding the kg name in the code. Other parameters can also be changed within the code.
You can access a full KG and splits at: https://drive.google.com/drive/folders/1NXLQQsIXe0hz32KSOeSG1PCAzFLHoSGh?usp=sharing

**Metapath2Vec**

metapath2vec submodule based on: https://stellargraph.readthedocs.io/en/stable/demos/link-prediction/metapath2vec-link-prediction.html

### Enviroments compatibility

For a better user experience we recommend setting up two virtual environments for running biologist: 
* ```requirements.txt``` for all the codes, except ```topic_distribution.ipynb```; ```topic_generation.ipynb```; and ```hin_generation.ipynb```;
* ```requirements_topic.txt``` for ```topic_distribution.ipynb```; ```topic_generation.ipynb```; and ```hin_generation.ipynb``` (BERTopic requires a different numpy version for numba).

### Preparing new files for performance evaluation

The files must follow a naming convention, for example `knn_results_deep_walk_0.8_doi_bioActivity_0_2nd.csv`: 
* `knn_results` is the name of the experiments batch used in natuke;
* `deep_walk` is the name of the algorithm;
* `0.8` represents the maximum train/test splits percentage in the evaluation stages;
* `doi_bioActivity` is the edge_type restored for evaluation;
* `0` is the random sampling for the splits;
* and `2nd` is the evaluation_stage.

All these parameters can be changed as needed in this portion of the `dynamic_benchmark_evaluation.py`
```
path = 'path-to-data-repository'
file_name = "knn_results"
splits = [0.8]
#edge_groups = ['doi_name', 'doi_bioActivity', 'doi_collectionSpecie', 'doi_collectionSite', 'doi_collectionType']
edge_group = 'doi_collectionType'
#algorithms = ['bert', 'deep_walk', 'node2vec', 'metapath2vec', 'regularization']
algorithms = ['deep_walk', 'node2vec', 'metapath2vec', 'regularization']
k_at = [1, 5, 10, 15, 20, 25, 30, 35, 40, 45, 50]
dynamic_stages = ['1st', '2nd', '3rd', '4th']
```

## Benchmark

More information about the NatUKE benchmark is available at https://github.com/aksw/natuke#benchmark.

### Data

More information about the dataset used for evaluation is available at https://github.com/aksw/natuke#data.

### Models

More information about the models evaluated are available at https://github.com/aksw/natuke#models.

## Results

### Original NatUKE

Results from the original NatUKE benchmark are available at https://github.com/aksw/natuke#results.

### ImpNatUKE

Results with the PDF file processors PyMuPdf, GROBID and Nougat for the language model DistilBERT:

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-c3ow{border-color:inherit;text-align:center;vertical-align:top}
.tg .tg-7btt{border-color:inherit;font-weight:bold;text-align:center;vertical-align:top}
.tg .tg-0pky{border-color:inherit;text-align:left;vertical-align:top}
.tg .tg-amwm{font-weight:bold;text-align:center;vertical-align:top}
.tg .tg-uzvj{border-color:inherit;font-weight:bold;text-align:center;vertical-align:middle}
.tg .tg-0lax{text-align:left;vertical-align:top}
.tg .tg-73oq{border-color:#000000;text-align:left;vertical-align:top}
</style>
<table class="tg"><thead>
  <tr>
    <th class="tg-7btt" colspan="14">DistilBERT</th>
  </tr></thead>
<tbody>
  <tr>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-7btt" colspan="4">PyMuPdf</td>
    <td class="tg-c3ow" colspan="4"><span style="font-weight:bold">GROBID</span></td>
    <td class="tg-amwm" colspan="4">Nougat</td>
  </tr>
  <tr>
    <td class="tg-uzvj"></td>
    <td class="tg-7btt">k</td>
    <td class="tg-7btt">1st</td>
    <td class="tg-7btt">2nd</td>
    <td class="tg-7btt">3rd</td>
    <td class="tg-7btt">4th</td>
    <td class="tg-7btt">1st</td>
    <td class="tg-7btt">2nd</td>
    <td class="tg-7btt">3rd</td>
    <td class="tg-amwm">4th</td>
    <td class="tg-amwm">1st</td>
    <td class="tg-amwm">2nd</td>
    <td class="tg-amwm">3rd</td>
    <td class="tg-amwm">4th</td>
  </tr>
  <tr>
    <td class="tg-7btt">C</td>
    <td class="tg-c3ow">50</td>
    <td class="tg-0pky">0.09 ± 0.01</td>
    <td class="tg-0pky">0.02 ± 0.01</td>
    <td class="tg-0pky">0.03 ± 0.02</td>
    <td class="tg-0pky">0.04 ± 0.05</td>
    <td class="tg-0pky">0.09 ± 0.01</td>
    <td class="tg-0pky">0.00 ± 0.00</td>
    <td class="tg-0pky">0.00 ± 0.00</td>
    <td class="tg-0lax">0.00 ± 0.00</td>
    <td class="tg-0lax">0.09 ± 0.01</td>
    <td class="tg-0lax">0.00 ± 0.00</td>
    <td class="tg-0lax">0.00 ± 0.00</td>
    <td class="tg-0lax">0.00 ± 0.00</td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">B</td>
    <td class="tg-c3ow">5</td>
    <td class="tg-73oq">0.55 ± 0.06</td>
    <td class="tg-0pky">0.57 ± 0.07</td>
    <td class="tg-0pky">0.60 ± 0.08</td>
    <td class="tg-0pky">0.64 ± 0.07</td>
    <td class="tg-0pky">0.58 ± 0.05</td>
    <td class="tg-0pky">0.64 ± 0.03</td>
    <td class="tg-0pky">0.69 ± 0.06</td>
    <td class="tg-0lax">0.73 ± 0.08</td>
    <td class="tg-0lax">0.59 ± 0.06</td>
    <td class="tg-0lax">0.66 ± 0.05</td>
    <td class="tg-0lax">0.69 ± 0.05</td>
    <td class="tg-0lax">0.71 ± 0.11</td>
  </tr>
  <tr>
    <td class="tg-c3ow">1</td>
    <td class="tg-0pky">0.17 ± 0.05</td>
    <td class="tg-0pky">0.19 ± 0.05</td>
    <td class="tg-0pky">0.24 ± 0.06</td>
    <td class="tg-0pky">0.25 ± 0.06</td>
    <td class="tg-0pky">0.19 ± 0.02</td>
    <td class="tg-0pky">0.23 ± 0.03</td>
    <td class="tg-0pky">0.28 ± 0.06</td>
    <td class="tg-0lax">0.35 ± 0.10</td>
    <td class="tg-0lax">0.19 ± 0.02</td>
    <td class="tg-0lax">0.25 ± 0.04</td>
    <td class="tg-0lax">0.30 ± 0.06</td>
    <td class="tg-0lax">0.33 ± 0.10</td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">S</td>
    <td class="tg-c3ow">50</td>
    <td class="tg-0pky">0.36 ± 0.04</td>
    <td class="tg-0pky">0.24 ± 0.03</td>
    <td class="tg-0pky">0.29 ± 0.07</td>
    <td class="tg-0pky">0.30 ± 0.06</td>
    <td class="tg-0pky">0.34 ± 0.03</td>
    <td class="tg-0pky">0.24 ± 0.03</td>
    <td class="tg-0pky">0.29 ± 0.06</td>
    <td class="tg-0lax">0.34 ± 0.10</td>
    <td class="tg-0lax">0.34 ± 0.03</td>
    <td class="tg-0lax">0.23 ± 0.03</td>
    <td class="tg-0lax">0.29 ± 0.05</td>
    <td class="tg-0lax">0.30 ± 0.10</td>
  </tr>
  <tr>
    <td class="tg-c3ow">20</td>
    <td class="tg-0pky">0.10 ± 0.02</td>
    <td class="tg-0pky">0.15 ± 0.03</td>
    <td class="tg-0pky">0.19 ± 0.05</td>
    <td class="tg-0pky">0.22 ± 0.07</td>
    <td class="tg-0pky">0.10 ± 0.03</td>
    <td class="tg-0pky">0.17 ± 0.03</td>
    <td class="tg-0pky">0.22 ± 0.04</td>
    <td class="tg-0lax">0.28 ± 0.07</td>
    <td class="tg-0lax">0.11 ± 0.03</td>
    <td class="tg-0lax">0.18 ± 0.03</td>
    <td class="tg-0lax">0.21 ± 0.03</td>
    <td class="tg-0lax">0.25 ± 0.09</td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">L</td>
    <td class="tg-c3ow">20</td>
    <td class="tg-0pky">0.53 ± 0.03</td>
    <td class="tg-0pky">0.52 ± 0.06</td>
    <td class="tg-0pky">0.55 ± 0.04</td>
    <td class="tg-0pky">0.55 ± 0.06</td>
    <td class="tg-0pky">0.56 ± 0.04</td>
    <td class="tg-0pky">0.62 ± 0.03</td>
    <td class="tg-0pky">0.62 ± 0.05</td>
    <td class="tg-0lax">0.62 ± 0.08</td>
    <td class="tg-0lax">0.56 ± 0.04</td>
    <td class="tg-0lax">0.62 ± 0.03</td>
    <td class="tg-0lax">0.63 ± 0.05</td>
    <td class="tg-0lax">0.65 ± 0.08</td>
  </tr>
  <tr>
    <td class="tg-c3ow">5</td>
    <td class="tg-0pky">0.26 ± 0.04</td>
    <td class="tg-0pky">0.29 ± 0.05</td>
    <td class="tg-0pky">0.30 ± 0.07</td>
    <td class="tg-0pky">0.27 ± 0.07</td>
    <td class="tg-0pky">0.28 ± 0.04</td>
    <td class="tg-0pky">0.35 ± 0.05</td>
    <td class="tg-0pky">0.36 ± 0.04</td>
    <td class="tg-0lax">0.35 ± 0.08</td>
    <td class="tg-0lax">0.27 ± 0.05</td>
    <td class="tg-0lax">0.31 ± 0.04</td>
    <td class="tg-0lax">0.35 ± 0.08</td>
    <td class="tg-0lax">0.38 ± 0.09</td>
  </tr>
  <tr>
    <td class="tg-7btt">T</td>
    <td class="tg-c3ow">1</td>
    <td class="tg-0pky">0.71 ± 0.04</td>
    <td class="tg-0pky">0.66 ± 0.10</td>
    <td class="tg-0pky">0.75 ± 0.10</td>
    <td class="tg-0pky">0.75 ± 0.11</td>
    <td class="tg-0pky">0.77 ± 0.02</td>
    <td class="tg-0pky">0.75 ± 0.04</td>
    <td class="tg-0pky">0.76 ± 0.05</td>
    <td class="tg-0lax">0.77 ± 0.06</td>
    <td class="tg-0lax">0.78 ± 0.01</td>
    <td class="tg-0lax">0.78 ± 0.04</td>
    <td class="tg-0lax">0.78 ± 0.05</td>
    <td class="tg-0lax">0.80 ± 0.09</td>
  </tr>
</tbody></table>

Results with the PDF file processors PyMuPdf, GROBID and Nougat for the language model llama-3.1.:

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-c3ow{border-color:inherit;text-align:center;vertical-align:top}
.tg .tg-7btt{border-color:inherit;font-weight:bold;text-align:center;vertical-align:top}
.tg .tg-0pky{border-color:inherit;text-align:left;vertical-align:top}
.tg .tg-amwm{font-weight:bold;text-align:center;vertical-align:top}
.tg .tg-uzvj{border-color:inherit;font-weight:bold;text-align:center;vertical-align:middle}
.tg .tg-0lax{text-align:left;vertical-align:top}
.tg .tg-73oq{border-color:#000000;text-align:left;vertical-align:top}
</style>
<table class="tg"><thead>
  <tr>
    <th class="tg-7btt" colspan="14">llama-3.1</th>
  </tr></thead>
<tbody>
  <tr>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-7btt" colspan="4">PyMuPdf</td>
    <td class="tg-c3ow" colspan="4"><span style="font-weight:bold">GROBID</span></td>
    <td class="tg-amwm" colspan="4">Nougat</td>
  </tr>
  <tr>
    <td class="tg-uzvj"></td>
    <td class="tg-7btt">k</td>
    <td class="tg-7btt">1st</td>
    <td class="tg-7btt">2nd</td>
    <td class="tg-7btt">3rd</td>
    <td class="tg-7btt">4th</td>
    <td class="tg-7btt">1st</td>
    <td class="tg-7btt">2nd</td>
    <td class="tg-7btt">3rd</td>
    <td class="tg-amwm">4th</td>
    <td class="tg-amwm">1st</td>
    <td class="tg-amwm">2nd</td>
    <td class="tg-amwm">3rd</td>
    <td class="tg-amwm">4th</td>
  </tr>
  <tr>
    <td class="tg-7btt">C</td>
    <td class="tg-c3ow">50</td>
    <td class="tg-0pky">0.09 ± 0.01</td>
    <td class="tg-0pky">0.00 ± 0.00</td>
    <td class="tg-0pky">0.00 ± 0.00</td>
    <td class="tg-0pky">0.00 ± 0.00</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">0.09 ± 0.01</td>
    <td class="tg-0lax">0.00 ± 0.00</td>
    <td class="tg-0lax">0.00 ± 0.00</td>
    <td class="tg-0lax">0.00 ± 0.00</td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">B</td>
    <td class="tg-c3ow">5</td>
    <td class="tg-73oq">0.51 ± 0.07</td>
    <td class="tg-0pky">0.51 ± 0.04</td>
    <td class="tg-0pky">0.51 ± 0.06</td>
    <td class="tg-0pky">0.54 ± 0.08</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">0.52 ± 0.06</td>
    <td class="tg-0lax">0.46 ± 0.03</td>
    <td class="tg-0lax">0.45 ± 0.03</td>
    <td class="tg-0lax">0.46 ± 0.07</td>
  </tr>
  <tr>
    <td class="tg-c3ow">1</td>
    <td class="tg-0pky">0.13 ± 0.03</td>
    <td class="tg-0pky">0.11 ± 0.03</td>
    <td class="tg-0pky">0.11 ± 0.03</td>
    <td class="tg-0pky">0.14 ± 0.04</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">0.12 ± 0.02</td>
    <td class="tg-0lax">0.09 ± 0.02</td>
    <td class="tg-0lax">0.08 ± 0.03</td>
    <td class="tg-0lax">0.08 ± 0.04</td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">S</td>
    <td class="tg-c3ow">50</td>
    <td class="tg-0pky">0.34 ± 0.04</td>
    <td class="tg-0pky">0.23 ± 0.03</td>
    <td class="tg-0pky">0.28 ± 0.07</td>
    <td class="tg-0pky">0.26 ± 0.06</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">0.34 ± 0.03</td>
    <td class="tg-0lax">0.22 ± 0.03</td>
    <td class="tg-0lax">0.25 ± 0.04</td>
    <td class="tg-0lax">0.26 ± 0.11</td>
  </tr>
  <tr>
    <td class="tg-c3ow">20</td>
    <td class="tg-0pky">0.10 ± 0.03</td>
    <td class="tg-0pky">0.11 ± 0.03</td>
    <td class="tg-0pky">0.11 ± 0.03</td>
    <td class="tg-0pky">0.13 ± 0.05</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">0.09 ± 0.02</td>
    <td class="tg-0lax">0.11 ± 0.04</td>
    <td class="tg-0lax">0.12 ± 0.04</td>
    <td class="tg-0lax">0.13 ± 0.09</td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">L</td>
    <td class="tg-c3ow">20</td>
    <td class="tg-0pky">0.55 ± 0.05</td>
    <td class="tg-0pky">0.58 ± 0.04</td>
    <td class="tg-0pky">0.59 ± 0.06</td>
    <td class="tg-0pky">0.55 ± 0.09</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">0.56 ± 0.04</td>
    <td class="tg-0lax">0.58 ± 0.04</td>
    <td class="tg-0lax">0.54 ± 0.07</td>
    <td class="tg-0lax">0.53 ± 0.08</td>
  </tr>
  <tr>
    <td class="tg-c3ow">5</td>
    <td class="tg-0pky">0.23 ± 0.04</td>
    <td class="tg-0pky">0.22 ± 0.03</td>
    <td class="tg-0pky">0.23 ± 0.06</td>
    <td class="tg-0pky">0.22 ± 0.04</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">0.19 ± 0.05</td>
    <td class="tg-0lax">0.18 ± 0.05</td>
    <td class="tg-0lax">0.17 ± 0.06</td>
    <td class="tg-0lax">0.13 ± 0.04</td>
  </tr>
  <tr>
    <td class="tg-7btt">T</td>
    <td class="tg-c3ow">1</td>
    <td class="tg-0pky">0.64 ± 0.11</td>
    <td class="tg-0pky">0.58 ± 0.10</td>
    <td class="tg-0pky">0.55 ± 0.12</td>
    <td class="tg-0pky">0.55 ± 0.10</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax">0.57 ± 0.07</td>
    <td class="tg-0lax">0.62 ± 0.07</td>
    <td class="tg-0lax">0.62 ± 0.06</td>
    <td class="tg-0lax">0.58 ± 0.11</td>
  </tr>
</tbody></table>

Results with the PDF file processors PyMuPdf, GROBID and Nougat for the language model Gemma 2:

<style type="text/css">
.tg  {border-collapse:collapse;border-spacing:0;}
.tg td{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{border-color:black;border-style:solid;border-width:1px;font-family:Arial, sans-serif;font-size:14px;
  font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-c3ow{border-color:inherit;text-align:center;vertical-align:top}
.tg .tg-7btt{border-color:inherit;font-weight:bold;text-align:center;vertical-align:top}
.tg .tg-0pky{border-color:inherit;text-align:left;vertical-align:top}
.tg .tg-amwm{font-weight:bold;text-align:center;vertical-align:top}
.tg .tg-uzvj{border-color:inherit;font-weight:bold;text-align:center;vertical-align:middle}
.tg .tg-0lax{text-align:left;vertical-align:top}
.tg .tg-73oq{border-color:#000000;text-align:left;vertical-align:top}
</style>
<table class="tg"><thead>
  <tr>
    <th class="tg-7btt" colspan="14">Gemma 2</th>
  </tr></thead>
<tbody>
  <tr>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-7btt" colspan="4">PyMuPdf</td>
    <td class="tg-c3ow" colspan="4"><span style="font-weight:bold">GROBID</span></td>
    <td class="tg-amwm" colspan="4">Nougat</td>
  </tr>
  <tr>
    <td class="tg-uzvj"></td>
    <td class="tg-7btt">k</td>
    <td class="tg-7btt">1st</td>
    <td class="tg-7btt">2nd</td>
    <td class="tg-7btt">3rd</td>
    <td class="tg-7btt">4th</td>
    <td class="tg-7btt">1st</td>
    <td class="tg-7btt">2nd</td>
    <td class="tg-7btt">3rd</td>
    <td class="tg-amwm">4th</td>
    <td class="tg-amwm">1st</td>
    <td class="tg-amwm">2nd</td>
    <td class="tg-amwm">3rd</td>
    <td class="tg-amwm">4th</td>
  </tr>
  <tr>
    <td class="tg-7btt">C</td>
    <td class="tg-c3ow">50</td>
    <td class="tg-0pky">0.09 ± 0.01</td>
    <td class="tg-0pky">0.00 ± 0.00</td>
    <td class="tg-0pky">0.00 ± 0.00</td>
    <td class="tg-0pky">0.00 ± 0.00</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">B</td>
    <td class="tg-c3ow">5</td>
    <td class="tg-73oq">0.52 ± 0.05</td>
    <td class="tg-0pky">0.51 ± 0.09</td>
    <td class="tg-0pky">0.53 ± 0.06</td>
    <td class="tg-0pky">0.58 ± 0.10</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-c3ow">1</td>
    <td class="tg-0pky">0.13 ± 0.02</td>
    <td class="tg-0pky">0.14 ± 0.03</td>
    <td class="tg-0pky">0.12 ± 0.04</td>
    <td class="tg-0pky">0.16 ± 0.05</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">S</td>
    <td class="tg-c3ow">50</td>
    <td class="tg-0pky">0.34 ± 0.04</td>
    <td class="tg-0pky">0.22 ± 0.03</td>
    <td class="tg-0pky">0.26 ± 0.05</td>
    <td class="tg-0pky">0.24 ± 0.08</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-c3ow">20</td>
    <td class="tg-0pky">0.10 ± 0.03</td>
    <td class="tg-0pky">0.11 ± 0.02</td>
    <td class="tg-0pky">0.12 ± 0.03</td>
    <td class="tg-0pky">0.11 ± 0.06</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-uzvj" rowspan="2">L</td>
    <td class="tg-c3ow">20</td>
    <td class="tg-0pky">0.56 ± 0.04</td>
    <td class="tg-0pky">0.55 ± 0.04</td>
    <td class="tg-0pky">0.57 ± 0.06</td>
    <td class="tg-0pky">0.55 ± 0.08</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-c3ow">5</td>
    <td class="tg-0pky">0.22 ± 0.04</td>
    <td class="tg-0pky">0.22 ± 0.03</td>
    <td class="tg-0pky">0.25 ± 0.04</td>
    <td class="tg-0pky">0.23 ± 0.08</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
  <tr>
    <td class="tg-7btt">T</td>
    <td class="tg-c3ow">1</td>
    <td class="tg-0pky">0.74 ± 0.06</td>
    <td class="tg-0pky">0.71 ± 0.10</td>
    <td class="tg-0pky">0.68 ± 0.16</td>
    <td class="tg-0pky">0.69 ± 0.15</td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0pky"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
    <td class="tg-0lax"></td>
  </tr>
</tbody></table>

## License

The code and experiments are available as open source under the terms of the [Apache 2.0 License](https://github.com/AKSW/natuke/blob/main/LICENSE).

The dataset used for training and benchmark are available under the license [Attribution-NonCommercial-NoDerivatives 4.0 International (CC BY-NC-ND 4.0)](https://creativecommons.org/licenses/by-nc-nd/4.0/).
*Which allows the use of the data only on its current form.*

## Wiki

The original NatUKE benchmark has an extended version of the paper and other information at the wiki page: https://github.com/AKSW/natuke/wiki/NatUKE-Wiki.
