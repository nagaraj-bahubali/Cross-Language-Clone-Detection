
# README

This project is the reproduction of the paper "Cross-language clone detection by
learning over abstract syntax trees" to detect the clones(similar syntax) across different(Java and Python) programming languages. 


Metadata:
A reproduction as part of the MSR course at MSR course 2020/21 at UniKo, CS department, SoftLang Team.
Title: Cross-language clone detection by learning over abstract syntax trees (https://static.perez.sh/research/2019/cross-language-clone-detection/clone-detection-msr19.pdf)
DBLP link: https://dblp.org/rec/conf/msr/PerezC19.html?view=bibtex


Requirements:
Hardware: Windows/linux 
Storage space required: 1-2 GB

Software:
python 3.6.2
tensorflow 1.13.1
keras 2.1.2
numpy 1.18
scipy 1.1.0
scikit-learn 0.21.3
Docker application


Process:
Steps to reproduce the paper.


Data:
Input data: 
Two different datasets are created which cotains the source code of projects in Java and python language.
Java dataset: Contains all java projects available on Github in the Apache organization (463 MB)
Python data: Contains popular python projects available on Github (204 MB)


Temporary data:
1. With the available input data, training data (target,context pairs) for both Java and Python is prepared in order to generate token/node embeddings of ASTs.
2. Generating the embeddings for the nodes/token of ASTs in python and java code using skipgram model. You can find the architecture of token-level embedding generation.
    * Node/token embeddings of java code: (repo link)
    * Node/token embeddings of python code: (repo link)
3. Generating java-python pairs to train LSTM model that detects clones accross java and python code: (repo link)

Output data:
Weights of the LSTM model to detect clones is produced. This file can be used to evaluate the model. (repo link)
The architecture/overview of the clone detection model can be found here.



Delta:
Process delta: 
1. In this paper, authors tried with different hyperparameters for the skipgram model with respect to window size of ancestors and descendants to learn representation for the tokens of ASTs. 
Finally they chose the parameters that works best for generating the embeddings. We have followed all the steps and reproduced complete paper from generating dataset to 
training cross language clone detection model using the best parameters.
2. The paper also presents few baseline models (such as Randomly initialized token vectors) in order to evaluate the importance of the structure of ASTs in training the model.    
We have reproduced the actual model but not baseline models.
Data delta:
With the available input data, we generated the ASTs and vocabulary file for both java and Python. Unfortunately it led to some error as the nodes of ASTs were not normalized. So we 
used the the ASTs and the vocabulary files provided by the developer in one of the issues. Using these files, we continued with generating skipgram data and so on.




