
# README

This project is the reproduction of the paper "Cross-language clone detection by
learning over abstract syntax trees" to detect the clones(similar syntax) across different(java and python) programming languages. 


## Metadata: <br />

This is a part of the MSR course at MSR course 2020/21 at UniKo, CS department, SoftLang Team. <br />

**Title:** Cross-language clone detection by learning over abstract syntax trees (https://static.perez.sh/research/2019/cross-language-clone-detection/clone-detection-msr19.pdf)<br />

**DBLP link:** https://dblp.org/rec/conf/msr/PerezC19.html?view=bibtex <br />


## Requirements: <br />
**Hardware:** <br />
* OS: Windows/linux <br />
* Storage space required: 1-2 GB <br />

**Software:** <br />
* python 3.6.2 <br />
* tensorflow 1.13.1 <br />
* keras 2.1.2 <br />
* numpy 1.18 <br />
* scipy 1.1.0 <br />
* scikit-learn 0.21.3 <br />
* [Anaconda][11] application <br />
* [Docker][5] application <br />

## Process: <br />
**Steps:**

Here are the steps to reproduce the paper on [linux][1](macOS) and on [Windows][8] system. <br />


**Validation:** <br/>

* To directly evaluate the model, generate the training/validation/test dataset in step 1 and test the model in step 3 under the [Training and evaluating code clone detection model][2]. <br/>
* The performance of the model is measured in terms of accuracy, recall and precision. After evaluating the model, a [result.json][9] file will be created that describes the performance metrics. Also the training log can be found [here][10].

## Data: <br />
**Input data:** <br />
1. Token leven vector generation [dataset][4]: <br/>
Two different datasets are used which contains the source code of projects in Java and python language. This dataset is used to generate token leven vectors for java and python.<br/> 
* Java dataset: Contains all java projects(count- 1027) available on Github in the Apache organization(463 MB) <br/> 
* Python data: Contains popular python projects(count- 879) available on Github (204 MB)  <br/>

2. Code clones [dataset][4]: <br/>
This dataset contains the code fragments available in java and python along with the label whether the two code fragments are similar or not. This information is 
not readily available and thus we used the java and python code clone dataset created by the developers. <br/>


**Temporary data:** <br />
1. With the available input data, training data (target,context pairs) for both Java and Python is prepared in order to generate token-level vectors. <br/>
2. Generating the embeddings for the tokens in python and java code using skipgram model.  <br/>
    * Token level embeddings of java code
    * Token level embeddings of python code
3. Java-python code fragment pairs to train LSTM model that detects clones accross java and python code. <br/>

You can find the [architecture][6] of token-level vector generation.

**Output data:**<br />

Weights of the LSTM model to detect clones is produced. This file can be used to evaluate the model. <br/>
The architecture/overview of the clone detection model can be found [here][7]. <br/>


## Delta:<br />
**Process delta:** <br />
1. In this paper, authors tried with different hyperparameters for generating training data for token vector generation model with respect to window size of ancestors and descendants to learn representation for the tokens of ASTs. <br/>
As mentioned in papaer, authors tried with<br/> 
* Ancestors window size: 0 to 5 <br/>
* Descendants window size: 0 to 4  <br/>
* Siblings included: yes and no <br/>
* Output vector dimension: 10, 20, 50, 100 and 200 <br/>

Finally they chose the parameters that works best for generating the embeddings. <br/>
* Ancestors window size: 2 <br/>
* Descendants window size: 1 <br/>
* Siblings included: no <br/>
* Output vector dimension: 50 <br/>

We have followed all the steps from generating dataset to training cross language clone detection model using only the best parameters. <br/>

2. The paper also presents few baseline models (such as Randomly initialized token vectors, pre-trained token vectors without values) in order to evaluate 
the importance of the structure of ASTs in training the model. We have reproduced the actual model but not baseline models. Below are the hyperparameters of the 
code clone detection model used by the authors: <br/>

* Token vector dimension: 100 <br/>
* Encoder layer: bidirectional LSTM, stacked with 2 layers <br/>
* layer dimensions: 100 and 50 <br/>
* Classifier single hidden layer: 64 units <br/>
* Optimizer: RMSprop <br/>
* Epochs: 50  <br/>

We trained the model with 5 epochs as it took considerably large amount of time (36 hours) keeping other hyperparameters same. <br/>


**Data delta:** <br />

1. As mentioned in the paper, we generated the sample of java and python repositories and also
the ASTs and vocabulary files for both. But this vocabulary file was not good enough to train the skipgram model. <br/>
So we tried to generate the vocabulary from the larger python and java repositories given [here][4]. Since the dataset was huge, we faced java.lang.OutOfMemoryError. So we used the the ASTs and the vocabulary files alone
provided by the developer. Using these files, we continued with generating skipgram data and so on. <br/>
So in a nutshell, we mimicked the complete process from generating the input dataset to training the code clone detection model. Instead of using the ASTs and vocabulary files we generated, we used the files provided by the developer and continued with the remaining steps. 

2. To train the code clone fragments with java-python pairs, we used the clone dataset provided by the authors/developers. 


[1]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/README.md
[2]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/README.md#2-training-and-evaluating-the-code-clone-detection-model
[3]: https://cloud.uni-koblenz-landau.de/s/8iwYX7MfnkifxRM
[4]: https://daniel.perez.sh/research/2019/cross-language-clones/
[5]: https://docs.docker.com/desktop/
[6]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/token-embeddings-generation.png
[7]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/clone-detection.png
[8]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/Windows/README.md
[9]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/data/results.json
[10]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/data/Training%20output.txt
[11]: https://www.anaconda.com/products/individual#Downloads
