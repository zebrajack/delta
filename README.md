# DELTA - a DEep Language Technology plAtform 

## What is DELTA?

**DELTA** is a deep learning based end-to-end natural language and speech processing platform. 
DELTA aims to provide easy and fast experiences for using, deploying, and developing natural language processing and speech models 
for both academia and industry use cases. DELTA is mainly implemented using TensorFlow.

For details of DELTA, please refer to this [paper(TBA)](https://arxiv.org/).

## What can DELTA do?

DELTA has been used for developing several state-of-the-art algorithms for publications and delivering real production to serve millions of users. 
It helps you to train, develop, and deploy NLP and/or speech models, featuring:

 - Easy-to-use
   - One command to train NLP and speech models, including:
     - NLP: text classification, named entity recognition, question and answering, text summarization, etc
     - Speech: speech recognition, speaker verification, emotion recognition, etc
   - Use configuration files to easily tune parameters and network structures
 - Easy-to-deploy
   - What you see in training is what you get in serving: all data processing and features extraction are integrated into a model graph
   - Uniform I/O interfaces and no changes for new models
 - Easy-to-develop
   - Easily build state-of-the-art models using modularized components
   - All modules are reliable and fully-tested

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Benchmarks](#benchmarks)
- [ToDo](#todo)
- [Contributing](#contributing)
- [License](#license)
- [Acknowlegement](#acknowlegement)

## Installation

### Quick Installation

We use [conda](https://conda.io/) to install required packages. Please [install conda](https://conda.io/en/latest/miniconda.html) if you do not have it in your system. 

We provide two options to install DELTA, `nlp` version or `full` version. 
`nlp` version need minimal requirements and only installs NLP related packages: 
Note: Users from mainland China may need to set up conda mirror sources, see [./tools/install/install-delta.sh](tools/install/install-delta.sh) for details.
```shell
# Run the installation script for NLP version, with CPU or GPU.
cd tools
./install/install-delta.sh nlp [cpu|gpu]
```

If you want to use both NLP and speech packages, you can install the `full` version. The full version needs [Kaldi](https://github.com/kaldi-asr/kaldi) library, which can be pre-installed or installed using our installation script.
```shell
cd tools
# If you have installed Kaldi
KALDI=/your/path/to/Kaldi ./install/install-delta.sh full [cpu|gpu]
# If you have not installed Kaldi, use the following command
# ./install/install-delta.sh full [cpu|gpu]
```

To verify the installation, run:

```shell
# Activate conda environment
conda activate delta-py3.6-tf1.14
# Or use the following command if your conda version is < 4.6
# source activate delta-py3.6-tf1.14

# Add DELTA enviornment
source env.sh

# generate mock data for text classification.
cd egs/mock_text_cls_data/text_cls/v1
./run.sh
cd -
python3 delta/main.py --cmd train_and_eval --config egs/mock_text_cls_data/text_cls/v1/config/han-cls.yml
```

### Manual installation

For advanced installation, full version users, or more details, please refer to [manual installation](docs/installation/manual_setup.md).

### Docker install

For Docker users, we provide images with DELTA installed. Please refer to [docker installation](docs/installation/using_docker.md).


## Quick Start

### Existing Examples
DELTA organizes many commonly-used tasks as examples in under `./egs` directory. 
Each example is an NLP or speech task using a public dataset.
We provide the while pipeline including data processing, model training, evaluation, and deployment.

You can simply use the `run.sh` under each directory to prepare the dataset, and then train or evaluate a model. 
For example, you use the following command to download the CONLL2003 dataset and train and evaluate a BLSTM-CRF model for NER: 
```shell
./egs/conll2003/seq_label/v1/run.sh
python3 delta/main.py --cmd train --config egs/conll2003/seq_label/v1/config/seq-label.yml
python3 delta/main.py --cmd eval --config egs/conll2003/seq_label/v1/config/seq-label.yml
```

### Modeling
There are several modes to start a DELTA pipeline:

- train_and_eval
- train
- eval
- infer
- export_model

Before run any command, please make sure you need to `source env.sh` in the current command prompt or a shell script.

You can use `train_and_eval` to start the model training and evaluation:

```shell
python3 delta/main.py --cmd train_and_eval --config <your configuration file>.yml
```

This is equivalent to:

```shell
python3 delta/main.py --cmd train --config <your configuration file>.yml 
python3 delta/main.py --cmd eval --config <your configuration file>.yml 
```

For evaluation, you need to prepare a data file with features and labels. 
If you only want to do inference with feature only, you can use the `infer` mode:

```shell
python3 delta/main.py --cmd infer --config <your configuration file>.yml 
```

When the training is done, you can export a model `checkpoint` to `SavedModel`:

```shell
python3 delta/main.py --cmd export_model --config <your configuration file>.yml 
```


### Deployment
For model deployment, we provide many tools in the DELTA-NN package.
We organize the model deployment scripts under `./dpl` directory.

* Putting `SavedModel` and configure `model.yaml` into `dpl/model`.
* Using script under `dpl/gadapter` to convert model to other deployment model.  
* All compiled `tensorflow` lib and `delta-nn` lib are in `dpl/lib`.
* Testing, benchmarking or serving under docker.

## Benchmarks
In DELTA, we provide experimental results for each task on public datasets as benchmarks. 
For each task, we compare our implementation with a similar model chosen from a highly-cited publication.
You can reproduce the experimental results using the scripts and configuration in the `./egs` directory. 
For more details, please refer to [released models](docs/released_models.md).

### NLP tasks

| Task | Model | DataSet | Metric | DELTA | Baseline | Baseline reference |
|---|---|---|---|---|---|---|
| Sentence Classification | CNN | TREC | Acc | 92.2 | 91.2 | Kim (2014) |
| Document Classification | HAN | Yahoo Answer | Acc | 75.1 | 75.8 | Yang et al. (2016) |
| Named Entity Recognition | BiLSTM-CRF | CoNLL 2003 | F1 | 84.6 | 84.7 | Huang et al. (2015) |
| Intent Detection (joint) | BiLSTM-CRF-Attention | ATIS | Acc | 97.4 | 98.2| Liu and Lane (2016) |
| Slots Filling (joint) | BiLSTM-CRF-Attention | ATIS  | F1 | 95.2 | 95.9 | Liu and Lane (2016) |
| Natural Language Inference | LSTM | SNLI | Acc | 80.7 | 80.6 | Bowman et al. (2016) | 
| Summarization | Seq2seq-LSTM |  |  |  |  |
| Translation | Seq2seq-transformer |  |  |  |  |
| Pretrain-NER | ELMO | CoNLL 2003 | F1 | 92.19 | 92.22 | Matthew et al. (2018) |
| Pretrain-NER | BERT |  |  |  |  |

### Speech tasks

| Task | Model | DataSet | Metric | DELTA | Baseline | Baseline reference |
|---|---|---|---|---|---|---|
| Speech recognition | CTC | HKUST | CER |  |  | Mial et al. (2016) |
| Speech recognition | Seq2seq | HKUST | CER |  |  |  |
| Speaker verfication | X-Vector | VoxCeleb | EER |  |  |  |
| Emotion recogniation | ResNet50 | IEMOCAP | Acc |  |  |  |


## TODO

For more details, please refer to [TODO list](docs/todo.md).

## Contributing

Any kind of contribution is welcome. Please just open an issue without any hesitation.
For more details, please refer to [the contribution guide](CONTRIBUTING.md).

## License

The DELTA platform is licensed under the terms of the Apache license.
See [LICENSE](LICENSE) for more information.

## Acknowlegement

The DELTA platform depends on many open source repos.
See [References](docs/references.rst) for more information.

