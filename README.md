
# README

This project is the reproduction of the paper "Cross-language clone detection by
learning over abstract syntax trees" to detect the clones(similar syntax) across different(Java and Python) programming languages. 


## Metadata: <br />
A reproduction as part of the MSR course at MSR course 2020/21 at UniKo, CS department, SoftLang Team. <br />
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
* Docker application <br />

## Process: <br />
**Steps:**
[Here][1] are the steps to reproduce the paper. <br />

**Validate:** To directly evaluate the model, generate the training/validation/test dataset in [step 1][2] and test the model in [step 3][2] under code clone detection model. 

## Data: <br />
**Input data:** <br />
1. Token leven vector generation dataset: <br/>
Two different datasets are created which contains the source code of projects in Java and python language. This dataset is used to generate token leven vectors for Java and Python.<br/> 
Java dataset: Contains all java projects(count- 1027) available on Github in the Apache organization (463 MB) <br/> 
Python data: Contains popular python projects(count- 879) available on Github (204 MB) <br/>

2. Code clones dataset: <br/>
This dataset contains the code fragments available in Java and Python along with the label whether the two code fragments are similar or not. This information is 
not readily available and thus we used the Java and Python code clone dataset created by the developers. <br/>


**Temporary data:** <br />
1. With the available input data, training data (target,context pairs) for both Java and Python is prepared in order to generate token/node embeddings of ASTs. <br/>
2. Generating the embeddings for the nodes/token of ASTs in python and java code using skipgram model. You can find the architecture of token-level embedding generation. <br/>
    * Node/token embeddings of java code: (repo link)
    * Node/token embeddings of python code: (repo link)
3. Generating java-python pairs to train LSTM model that detects clones accross java and python code. <br/>

**Output data:**<br />
Weights of the LSTM model to detect clones is produced. This file can be used to evaluate the model. <br/>
The architecture/overview of the clone detection model can be found here. <br/>


## Delta:<br />
**Process delta:** <br />
1. In this paper, authors tried with different hyperparameters for the skipgram model with respect to window size of ancestors and descendants to learn representation for the tokens of ASTs. 
Finally they chose the parameters that works best for generating the embeddings. We have followed all the steps and reproduced complete paper from generating dataset to 
training cross language clone detection model using the best parameters. <br/>
2. The paper also presents few baseline models (such as Randomly initialized token vectors) in order to evaluate the importance of the structure of ASTs in training the model. We have reproduced the actual model but not baseline models. <br/>

**Data delta:** <br />
1. As the technique of generation of the Token leven vector generation dataset was indicated in the paper, we created this dataset on our own. With this, we generated the ASTs and vocabulary file for both Java and Python. Unfortunately it led to some error as the nodes of ASTs were not normalized. So we used the the ASTs and the vocabulary files alone provided by the developer in one of the issues. Using these files, we continued with generating skipgram data and so on.

2. The code clone dataset given by the authors/developers was used in our reproduction project. This was used to train LSTM model.


[1]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/README.md
[2]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/README.md
