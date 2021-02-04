# Reproduction steps

## 0. Environment setup

To run the project we need `python=3.6.2` So we recommend to setup a virtual environment.
We used conda to create the virtual environment(takes considerable amount of time).
But feel free to use any.

Open Anaconda Prompt to setup and activate the environment
```
conda create -n msr_venv python=3.6.2 anaconda
conda activate msr_venv
```

Once the environment is setup, download the repository and `cd` to the process folder inside project root folder and run below
commands
```
pip install -r requirements.txt
python setup.py develop
```

Also download and unzip the necessary [dataset][15] and place it under [`data`][14] folder of repository and unzip it.<br/>
Your project folder should look something like below.

```
data
    dataset
        asts.json
        java-embeddings.npy
        java-vocab.tsv
        python-embeddings.npy
        trained-model.h5
        java-asts.json
        java-python-clones.db
        python-asts.json
        python-vocab.tsv	
    docker-generated-data
    README.md
doc
process
README.md
```


There are two major steps to reproduce this project

1. Data preparation
2. Training the Code Clone Detection model

If you want to skip data preperation and directly train the Code Clone Detection model click [here][12] for instructions
## 1. Data preparation

Go to The project directory that is in Cross-Language-Clone-Detection-master in your CLI. Which is one directory back to process folder
```
cd ..
```

To generate and preprocess the data we will use [Docker][1].
Start the docker application in your windows machine.

The Docker image is available as [`tuvistavie/bigcode-tools`][2]
To install it, you can run (NOTE: Docker should be started before running below command)

```
docker run tuvistavie/bigcode-tools ls
```
Note that download might take a while.

#### 1. Setting up workspace to store generated data
 
 We will use `docker-generated-data` folder within `data` folder to store all the data genarated from running the docker commands. 
 Make sure you are still under `process` folder.
```
set DOCKER_GENERATED_DATA=%CD%/data/docker-generated-data
```
To check the environment variable run below command 

```
echo %DOCKER_GENERATED_DATA%
```

To reduce Docker command boilerplate, we will alias the `run` command as follows

```
set docker-bigcode=docker run -p 6006:6006 -v %DOCKER_GENERATED_DATA%:/bigcode-tools/workspace tuvistavie/bigcode-tools
```

This will map the container `/bigcode-tools/workspace` directory to the host `%DOCKER_GENERATED_DATA%`
directory, and expose the container port `6006` to the same port on the host.

#### 2. Downloading some data

We will use a subset of Apache commons. First, we need to search using
the GitHub API:

```
%docker-bigcode% bigcode-fetcher search --language=java --user=apache --keyword=commons --stars=>20 -o workspace/java-projects.json
```

This should create a list of projects in `%DOCKER_GENERATED_DATA%/java-projects.json`

We are now going to download all the data into `%DOCKER_GENERATED_DATA%/repositories`

```
%docker-bigcode% bigcode-fetcher download -i workspace/java-projects.json -o workspace/repositories
```

There should now be more than 20 repositories inside `%DOCKER_GENERATED_DATA%/repositories/apache`.


#### 3. Preprocessing the data

We will now generate the ASTs for all the data in the downloaded repositories.

```
%docker-bigcode% bigcode-astgen-java --batch -o workspace/java-asts workspace/repositories/**/*.java
```

This will create three files:

1. `%DOCKER_GENERATED_DATA%/java-asts.json`: list of ASTs as documented in [bigcode-astgen](../bigcode-astgen/README.md)
2. `%DOCKER_GENERATED_DATA%/java-asts.txt`: the name of the file from which each AST was extracted
3. `%DOCKER_GENERATED_DATA%/java-asts_failed.txt`: the list of files for which parse failed 

We can visualize one of the generated AST. We will try with the first file in the dataset,
but if there is a warning, you can try with some other index containing a smaller AST.

```
%docker-bigcode% bigcode-ast-tools visualize-ast workspace/java-asts.json -i 0 --no-open -o workspace/ast0.png
```

This should generate `%DOCKER_GENERATED_DATA%/ast0.png`, which should look something like this ( image depends on the ast picked, so could be different for you)

![AST image][4]

#### 4. Generating embeddings

First, we will generate a vocabulary.

```
%docker-bigcode% bigcode-ast-tools generate-vocabulary --strip-identifiers -o workspace/java-vocabulary.tsv workspace/java-asts.json
```

This should generate `%DOCKER_GENERATED_DATA%/java-vocabulary.tsv` which looks like this

```
head -n 5 %DOCKER_GENERATED_DATA%/java-vocabulary.tsv
id	type	metaType	count
0	SimpleName	Other	1782623
1	NameExpr	Expr	655320
2	MethodCallExpr	Expr	403136
3	ClassOrInterfaceType	Type	344500
```

NOTE:The vocabulary generated with the repositories downloaded above is not good enough to train the skipgram model.
So we tried to generate the vocabulary from the actual set of Java and Python repositiries given [here][11].
Since the dataset was huge we faced `java.lang.OutOfMemoryError`. Hence we used the asts and vocabularies provided
by the developers to further generate training data for skipgram model. To make these asts and vocabularies available 
for docker to run, run below commands to copy them into docker workspace.
```
copy "%CD%\data\dataset\java-asts.json" "%DOCKER_GENERATED_DATA%\java-asts.json"
copy "%CD%\data\dataset\java-vocab.tsv" "%DOCKER_GENERATED_DATA%\java-vocab.tsv"
copy "%CD%\data\dataset\python-asts.json" "%DOCKER_GENERATED_DATA%\python-asts.json"
copy "%CD%\data\dataset\python-vocab.tsv" "%DOCKER_GENERATED_DATA%\python-vocab.tsv"
```

Now, we will generate [skipgram][6]-like data for Java and Python to train our model.
Run the below commands to generate the training data for Java 
```
mkdir "%DOCKER_GENERATED_DATA%\java-skipgram-data"
%docker-bigcode% bigcode-ast-tools generate-skipgram-data -v workspace/java-vocab.tsv --ancestors-window-size 2 --children-window-size 1 --without-siblings -o workspace/java-skipgram-data/skipgram-data workspace/java-asts.json
```

This will create `%DOCKER_GENERATED_DATA%/java-skipgram-data/skipgram-data-001.txt.gz`
(the number of files created depends on the number of cores)

we will follow the similar process for generating training data for Python
```
mkdir "%DOCKER_GENERATED_DATA%\python-skipgram-data"
%docker-bigcode% bigcode-ast-tools generate-skipgram-data -v workspace/python-vocab.tsv --ancestors-window-size 2 --children-window-size 1 --without-siblings -o workspace/python-skipgram-data/skipgram-data workspace/python-asts.json
```

We will now learn 50 dimensions embeddings on this data.
For Java run below commands<br/>
NOTE: This might take a while ( took us one hour and for some reason stdout seems not to be flushed when using Docker,
so there might be no output until the command finishes). You can skip the training as we have already generated the embeddings
which is available under `data/dataset/` as `java-embeddings.npy` and can directly go to 2nd part of the reproduction for Training and evaluating the Code Clone Detection model.

```
%docker-bigcode% sh -c "bigcode-embeddings train -o workspace/java-embeddings --vocab-size=10000 --emb-size=50 --optimizer=gradient-descent --batch-size=64 workspace/java-skipgram-data/skipgram-data*"
```

You will be presented with tensorflow warnings, just ignore them.

Once it is finished run below command and copy the first file name without extension ".meta"

```
DIR "%CD%\data\docker-generated-data\java-embeddings\*.meta" /B /O:-D
```

You will get list of files like below
```
embeddings.bin-2858147.meta
embeddings.bin-2286515.meta
embeddings.bin-1714878.meta
embeddings.bin-1143244.meta
embeddings.bin-571627.meta
```
select the only name of first file for e.g. in this case "embeddings.bin-2858147" and copy it

Now paste the file name in below command after = (NOTE: Do not add space before or after)

```
set JAVA_MODEL_NAME=embeddings.bin-2858147
```
Now run below command to export the embeddings to %docker-generated-data%/java-embeddings as numpy file.

```
%docker-bigcode% bigcode-embeddings export "workspace/java-embeddings/%JAVA_MODEL_NAME%"  -o workspace/java-embeddings.npy
```
This will export the embeddings as numpy file to %DOCKER_GENERATED_DATA%. 


For Python run below commands. it's optional as The embeddings are already stored in folder `/data/datasets` as `python-embeddings.npy` <br/>

NOTE : The training for python will take around 10 to 15 minutes
```
%docker-bigcode% sh -c "bigcode-embeddings train -o workspace/python-embeddings --vocab-size=10000 --emb-size=50 --optimizer=gradient-descent --batch-size=64 workspace/python-skipgram-data/skipgram-data*"

```
You will be presented with tensorflow warnings, just ignore them.


Once it is finished run below command and copy the first file name without extension ".meta"

```
DIR "%CD%\data\docker-generated-data\python-embeddings\*.meta" /B /O:-D
```

You will get list of files like below
```
embeddings.bin-409660.meta
embeddings.bin-327717.meta
embeddings.bin-245783.meta
embeddings.bin-163858.meta
embeddings.bin-81930.meta
```
select the only name of first file for e.g. in this case "embeddings.bin-409660" and copy it

Now paste the file name in below command after = . (NOTE: Do not add space before or after)

```
set PYTHON_MODEL_NAME=embeddings.bin-409660
```
Now run below command to export the embeddings to %docker-generated-data%/java-embeddings as numpy file.

```
%docker-bigcode% bigcode-embeddings export "workspace/python-embeddings/%PYTHON_MODEL_NAME%"  -o workspace/python-embeddings.npy
```
This will export the embeddings as numpy file to %DOCKER_GENERATED_DATA%. 

With this, we now have the  embeddings for Java and Python tokens which are saved as java-embeddings.npy and python-embeddings.npy respectively in the folder `$DOCKER_GENERATED_DATA`

## 2. Training and evaluating the Code Clone Detection model

The model is already configured in [`config.yml`][13] to use the following steps. Make sure you are under `process` folder in your CLI.

#### 1. Generating training samples
Before training the model, the clones pair for training/cross-validation/test must first be generated using the following command. 
(Ignore the warnings you get by running this command)

```
python %CD%/bin/suplearn-clone generate-dataset -c config.yml
```

#### 2. Training the model
Once the data is generated, the model can be trained by simply using the following command. It took us almost 36 hours to train the model. The logs are available [here][17]. If you want to directly evaluate the model follow next step (testing the model) as we have made the trained weights available in `data/dataset` folder. But make sure the training samples are generated from above step. Else below command will not work.
(Ignore the warnings you get by running this command)
```
python %CD%/bin/suplearn-clone --debug train -c config.yml
```

#### 3. Testing the model
The model can be evaulated on test data by using the following command. It takes around 15 to 20 minutes to generate the results. The results will be stored under `process` folder in file `final_results.json`
(Ignore the warnings you get by running this command)
```
python %CD%/bin/suplearn-clone evaluate -c config.yml -m ../data/dataset/trained-model.h5 --data-type=test -o final_results.json
```


[1]: https://docs.docker.com/engine/installation/
[2]: https://hub.docker.com/r/tuvistavie/bigcode-tools/
[3]: http://www.graphviz.org/
[4]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/Windows/ast0.png?raw=true
[5]: https://user-images.githubusercontent.com/1436271/31432039-6f640728-aeb0-11e7-9758-4454f492ca5d.png
[6]: https://www.tensorflow.org/tutorials/word2vec
[7]: https://github.com/tensorflow/tensorboard
[8]: https://user-images.githubusercontent.com/1436271/31433555-af44a02e-aeb4-11e7-86c7-79d224c3f908.png
[9]: https://user-images.githubusercontent.com/1436271/31434689-071240d8-aeb8-11e7-9c72-cc10b08a48e9.png
[10]: https://user-images.githubusercontent.com/1436271/31435872-03864c08-aebc-11e7-9ea3-be405ee8babd.png
[11]: https://daniel.perez.sh/research/2019/cross-language-clones/
[12]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/doc/README.md#2-training-the-code-clone-detection-model
[13]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/blob/master/process/config.yml
[14]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/tree/master/data
[15]: https://cloud.uni-koblenz-landau.de/s/8iwYX7MfnkifxRM
[16]: https://github.com/nagaraj-bahubali/Cross-Language-Clone-Detection/tree/master/process
[17]: https://gist.github.com/nagaraj-bahubali/993241fcf35e2c95ccae94abeabccb47
