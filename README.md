DrQA
---
> Facebook has finally released its official implementation of the paper half a month after this project emerged. This project will stick to its original purpose: it will provide a **clean and minimized** implementation of the paper, so one can quickly read through the code to understand the model, easily make some modifications to test new ideas, or plug the necessary parts into a larger framework. If you plan to deploy this model in a more industrial environment, please refer to [facebookresearch/DrQA](https://github.com/facebookresearch/DrQA). If you would like to embed this model in a chatbot framework, please refer to [facebookresearch/ParlAI](https://github.com/facebookresearch/ParlAI/).


A pytorch implementation of the ACL 2017 paper [Reading Wikipedia to Answer Open-Domain Questions](http://www-cs.stanford.edu/people/danqi/papers/acl2017.pdf) (DrQA).

Reading comprehension is a task to produce an answer when given a question and one or more pieces of evidence (usually natural language paragraphs). Compared to question answering over knowledge bases, reading comprehension models are more flexible and have revealed a great potential for zero-shot learning.

[SQuAD](https://rajpurkar.github.io/SQuAD-explorer/) is a reading comprehension benchmark where there's only a single piece of evidence and the answer is guaranteed to be a part of the evidence. Since the publication of SQuAD dataset, there has been fast progress in the research of reading comprehension and a bunch of great models have come out. DrQA is one that is conceptually simpler than most others but still yields strong performance even as a single model.

The motivation for this project is to offer a clean version of DrQA for the machine reading comprehension task, so one can quickly do some modifications and try out new ideas. Most of the model code is borrowed from [ParlAI](https://github.com/facebookresearch/ParlAI/). Click [here](#detailed-comparisons) to see the comparison with what's described in the original paper and with two "offical" projects ParlAI and DrQA.

## Requirements
- python >=3.5 
- pytorch 0.2.0 (please refer to [the previous version](https://github.com/hitvoice/DrQA/tree/bc0152c7ad69c56fda23f50adabd4355559b3a74) if you use pytorch 0.1.12)
- numpy
- pandas
- msgpack
- spacy 1.x

## Quick Start
### Setup
- download the project via `git clone https://github.com/hitvoice/DrQA.git; cd DrQA`
- make sure python 3 and pip is installed.
- install [pytorch](http://pytorch.org/) matched with your OS, python and cuda versions.
- install the remaining requirements via `pip install -r requirements.txt`
- download the SQuAD datafile, GloVe word vectors and Spacy English language models using `bash download.sh`.

### Train

```bash
# prepare the data
python prepro.py
# train for 20 epoches with batchsize 32
python train.py -e 20 -bs 32
```

## Results
### EM & F1
||EM|F1|
|---|---|---|
|in original paper|69.5|78.8|
|in this project|69.3|78.6|

Compared to the implementation in ParlAI:

<img src="https://rawgit.com/hitvoice/DrQA/master/img/em.svg" width="500">

<img src="https://rawgit.com/hitvoice/DrQA/master/img/f1.svg" width="500">

The command to run the ParlAI implementation:
```bash
git clone https://github.com/facebookresearch/ParlAI.git ~/ParlAI
cd ~/ParlAI; python setup.py develop
python examples/train_model.py -m drqa -t squad -bs 32 -e 30 -vp 20 -dbf True --validation-every-n-secs 400 -mf /home/ubuntu/ParlAI/models --embedding_file /home/ubuntu/glove/glove.840B.300d.txt --embedding_dim 300 --fix_embeddings False --tune_partial 1000 --dropout_rnn 0.3 --dropout_emb 0.3 | tee output.log
```

### training time
The experiments are run on a machine with a single NVIDIA Tesla K80 GPU, 8 CPUs (2.3GHz) and 59G RAM.

||training time (seconds/epoch)|
|---|---|
|implementation in this project|770|
|implementation in ParlAI|850|

### related discussions
Here's what the paper says when introducing the embedding layer:
> We keep most of the pre-trained word embeddings fixed and only fine-tune the 1000 **most frequent question words** because the representations of some keywords such as *what*, *how*, *which*, *many* could be crucial for QA systems.

So what's the difference between most frequent words and most frequent question words? Here are the top 20 words of each:

||sort by all|sort by question|
|---|---|---|
|1|the|?|
|2|,|the|
|3|of|What|
|4|.|of|
|5|and|in|
|6|in|to|
|7|to|was|
|8|a|is|
|9|"|did|
|10|is|what|
|11|-|a|
|12|was|'s|
|13|The|Who|
|14|as|How|
|15|(|for|
|16|)|and|
|17|?|,|
|18|for|are|
|19|by|many|
|20|that|When|

The venn diagram:

<img src="https://rawgit.com/hitvoice/DrQA/master/img/vocab.svg" width="500">

26% words are different in top 1000 words of the two vocabularies. When tuning 1000 most frequent question words instead of 1000 most frequent words, about 1.5% boost of the F1 score is observed.

### Detailed Comparisons

Compared to what's described in the original paper:
- The grammatical features are generated by SpaCy instead of Stanford Core NLP. It's much faster (5 minutes vs 20 hours) but less accurate.
- The training samples are shuffled completely in each epoch. Performance degrades significantly when sorting the samples by length, dividing into mini-batches and then shuffle the mini-batches as recorded in the paper.
- The original paper does not make it clear whether POS and NER is a one-hot feature or has its own trainable embedding matrix. This implementation treats these two tags as discrete features with their own embedding matrices, which is found to be better in performance and makes the model more flexible.

Compared to the code in ParlAI:
- The DrQA model is not longer wrapped in a chatbot framework, which makes the code more readable, easier to modify and is faster to train. The preprocessing for text corpus is performed only once, while in a dialog framework raw text is transmitted each time and preprocessing for the same text must be done again and again.
- This is a full implementation of the original paper, while the model in ParlAI is a partial implementation, missing all grammatical features (lemma, POS tags and named entity tags). 
- When tuning top-k embeddings, the model will tune the embeddings of top-k question words as the original paper states, while the word dictionary in ParlAI is sorted by the frequency of all words. This does make a difference (see the discussion above).
- Some minor bug fixes and enhancements. Some of them have been merged into ParlAI.

Compared to the code in facebookresearch/DrQA:
- This project is much more light-weighted, while lacking the document retriever, the inference and interactive inference API, the extendibility to other datasets and some other enhancements.
- The implementation in facebookresearch/DrQA tokenizes the dataset using a Java-coded Stanford CoreNLP, while in this project we use a faster and simpler Spacy.
- The implementation in facebookresearch/DrQA treats the POS and NER tags as one-hot features, this implementation treats these two tags as discrete features with their own embedding matrice.
- The implementation in facebookresearch/DrQA is able to train on multiple GPUs, while (currently and for simplicity) in this implementation we only support single-GPU training.

### About
Maintainer: [Runqi Yang](https://hitvoice.github.io/about/). 

Credits: thank Jun Yang for code review and advice.

Most of the pytorch model code is borrowed from [Facebook/ParlAI](https://github.com/facebookresearch/ParlAI/) under a BSD-3 license.
