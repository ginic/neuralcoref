# ✨NeuralCoref 4.0: English Coreference Resolution in spaCy with Neural Networks.

NeuralCoref is a pipeline extension for spaCy 2.1+ which annotates and resolves coreference clusters using a neural network. NeuralCoref is production-ready, integrated in spaCy's NLP pipeline and extensible to new training datasets.

For a brief introduction to coreference resolution and NeuralCoref, please refer to our [blog post](https://medium.com/huggingface/state-of-the-art-neural-coreference-resolution-for-chatbots-3302365dcf30).
NeuralCoref is written in Python/Cython and comes with a pre-trained statistical model for **English only**. NeuralCoref is accompanied by a visualization client [NeuralCoref-Viz](https://github.com/huggingface/neuralcoref-viz), a web interface  powered by a REST server that can be [tried online](https://huggingface.co/coref/). NeuralCoref is released under the MIT license.

✨ Version 4.0 out now! Available on pip and compatible with SpaCy 2.1+.

[![Current Release Version](https://img.shields.io/github/release/huggingface/neuralcoref.svg?style=flat-square)](https://github.com/huggingface/neuralcoref/releases)
[![spaCy](https://img.shields.io/badge/made%20with%20❤%20and-spaCy-09a3d5.svg)](https://spacy.io)
[![Travis-CI](https://travis-ci.org/huggingface/neuralcoref.svg?branch=master)](https://travis-ci.org/huggingface/neuralcoref)
[![NeuralCoref online Demo](https://huggingface.co/coref/assets/thumbnail-large.png)](https://huggingface.co/coref/)

- **Operating system**: macOS / OS X · Linux · Windows (Cygwin, MinGW, Visual Studio)
- **Python version**: Python 3.5+ (only 64 bit)
- **Package managers**: [pip]

## Install NeuralCoref

### Install NeuralCoref with pip

This is the easiest way to install NeuralCoref.

```bash
pip install neuralcoref
````

To be able to use NeuralCoref you will also need to have
- SpaCy
- an English model for SpaCy.

You can use whatever english model works fine for your application but note that the performances of NeuralCoref are strongly dependent on the performances of the SpaCy model and in particular on the performances of SpaCy model's tagger, parser and NER components. A larger SpaCy English model will thus improve the quality of the coreference resolution as well (see some details in the [Internals and Model](#-Internals-and-Model) section below).

Here is an example of how spacy and a (small) English model can be downloaded:

```bash
pip install spacy
spacy download en
```

### Install NeuralCoref from source

You can also install NeuralCoref from the source. You will need to install the dependencies first which includes Cython and SpaCy.

You can install from source as follows:

```bash
venv .env
source .env/bin/activate
git clone https://github.com/huggingface/neuralcoref.git
cd neuralcoref
pip install -r requirements.txt
pip install -e .
```

## Internals and Model

NeuralCoref is made of two sub-modules:

- a rule-based mentions-detection module which uses SpaCy's tagger, parser and NER annotations to identify a set of potential coreference mentions, and
- a feed-forward neural-network which compute a coreference score for each pair of potential mentions.

The first time you import NeuralCoref in python, it will download the weights of the neural network model in a cache folder.

The cache folder is set by defaults to `~/.neuralcoref_cache` (see [file_utils.py](./neuralcoref/file_utils.py)) but this behavior can be overided by setting the environment variable `NEURALCOREF_CACHE` to point to another location.

The cache folder can be safely deleted at any time and the module will download again the model the next time it is loaded.

## Loading NeuralCoref

### Adding NeuralCoref to the pipe of an English SpaCy Language

Here is the recommended way to instantiate NeuralCoref and add it to SpaCY's pipeline of annotations:

```python

# Load your usual SpaCy model (one of SpaCy English models)
import spacy
nlp = spacy.load('en')

# Add neural coref to SpaCy's pipe
import neuralcoref
neuralcoref.add_to_pipe(nlp)

# You're done. You can now use NeuralCoref as you usually manipulate a SpaCy document annotations.
doc = nlp(u'My sister has a dog. She loves him.')

doc._.has_coref
doc._.coref_clusters
```

### Loading NeuralCoref and adding it manually to the pipe of an English SpaCy Language

An equivalent way of adding NeuralCoref to a SpaCy model pipe is to instantiate the NeuralCoref class first and then add it manually to the pipe of the SpaCy Language model.

```python

# Load your usual SpaCy model (one of SpaCy English models)
import spacy
nlp = spacy.load('en')

# load NeuralCoref and add it to the pipe of SpaCy's model
import neuralcoref
coref = neuralcoref.NeuralCoref(nlp.vocab)
nlp.add_pipe(coref, name='neuralcoref')

# You're done. You can now use NeuralCoref the same way you usually manipulate a SpaCy document and it's annotations.
doc = nlp(u'My sister has a dog. She loves him.')

doc._.has_coref
doc._.coref_clusters
```

## Using NeuralCoref

NeuralCoref will resolve the coreferences and annotate them as [extension attributes](https://spacy.io/usage/processing-pipelines#custom-components-extensions) in the spaCy `Doc`,  `Span` and `Token` objects under the `._.` dictionary.

Here are a few examples on how you can navigate the coreference cluster chains and display clusters and mentions before we list all the extensions added by NeuralCoref to a spaCy document.

```python
import spacy
import neuralcoref
nlp = spacy.load('en')
neuralcoref.add_to_pipe(nlp)

doc = nlp(u'My sister has a dog. She loves him')

doc._.coref_clusters
doc._.coref_clusters[1].mentions
doc._.coref_clusters[1].mentions[-1]
doc._.coref_clusters[1].mentions[-1]._.coref_cluster.main

token = doc[-1]
token._.in_coref
token._.coref_clusters

span = doc[-1:]
span._.is_coref
span._.coref_cluster.main
span._.coref_cluster.main._.coref_cluster
```

**Important**: NeuralCoref mentions are spaCy [Span objects](https://spacy.io/api/span) which means you can access all the usual [Span attributes](https://spacy.io/api/span#attributes) like `span.start` (index of the first token of the span in the document), `span.end` (index of the first token after the span in the document), etc...

Ex: `doc._.coref_clusters[1].mentions[-1].start` will give you the index of the first token of the last mention of the second coreference cluster in the document.

Here is the full list of the Doc, Span and Token Extension Attributes added by NeuralCoref:

### Doc, Span and Token Extension Attributes

|  Attribute                |  Type              |  Description
|---------------------------|--------------------|-----------------------------------------------------
|`doc._.has_coref`          |boolean             |Has any coreference has been resolved in the Doc
|`doc._.coref_clusters`     |list of `Cluster`   |All the clusters of corefering mentions in the doc
|`doc._.coref_resolved`     |unicode             |Unicode representation of the doc where each corefering mention is replaced by the main mention in the associated cluster.
|`doc._.coref_scores`       |unicode             |Unicode representation of the doc where each corefering mention is replaced by the main mention in the associated cluster.
|`span._.is_coref`          |boolean             |Whether the span has at least one corefering mention
|`span._.coref_cluster`     |`Cluster`           |Cluster of mentions that corefer with the span
|`token._.in_coref`         |boolean             |Whether the token is inside at least one corefering mention
|`token._.coref_clusters`   |list of `Cluster`   |All the clusters of corefering mentions that contains the token

### The Cluster class

The Cluster class is a small container for a cluster of mentions.

A `Cluster` contains 3 attributes:

|  Attribute                |  Type              |  Description
|---------------------------|--------------------|-----------------------------------------------------
|`cluster.i`                |int                 |Index of the cluster in the Doc
|`cluster.main`             |`Span`              |Span of the most representative mention in the cluster
|`cluster.mentions`         |list of `Span`      |All the mentions in the cluster

The `Cluster` class also implements a few Python class methods to simplify the navigation inside a cluster:

|  Method              |  Output              |  Description
|----------------------|----------------------|-----------------------------------------------------
|`Cluster.__getitem__` |return `Span`         |Access a mention in the cluster
|`Cluster.__iter__`    |yields `Span`         |Iterate over mentions in the cluster
|`Cluster.__len__`     |return int            |Number of mentions in the cluster

## Parameters

You can pass several additional parameters to `neuralcoref.add_to_pipe` or `NeuralCoref()` to control the behavior of NeuralCoref.

Here is the full list of these parameters and their descriptions:

| Parameter       | Type                    | Description
|-----------------|-------------------------|----------------
|`greedyness`     |float                    |A number between 0 and 1 determining how greedy the model is about making coreference decisions (more greedy means more coreference links). The default value is 0.5.
|`max_dist`       |int                      |How many mentions back to look when considering possible antecedents of the current mention. Decreasing the value will cause the system to run faster but less accurately. The default value is 50.
|`max_dist_match` |int                      |The system will consider linking the current mention to a preceding one further than `max_dist` away if they share a noun or proper noun. In this case, it looks `max_dist_match` away instead. The default value is 500.
|`blacklist`      |boolean                  |Should the system resolve coreferences for pronouns in the following list: `["i", "me", "my", "you", "your"]`. The default value is True (coreference resolved).
|`store_scores`   |boolean                  |Should the system store the scores for the coreferences in annotations. The default value is True.
|`conv_dict`      |dict(str, list(str))     |A conversion dictionary that you can use to replace some *rare words* embeddings with an average of *common words* embeddings. Ex: `conv_dict={"Angela": ["woman", "girl"]}` will help resolving coreferences for `Angela` by using the embeddings for the more common `woman` and `girl` instead of the embedding of `Angela`.

### How to change a parameter

```python
import spacy
import neuralcoref

# Let's load a SpaCy model
nlp = spacy.load('en')

# First way we can control a parameter
neuralcoref.add_to_pipe(nlp, greedyness=0.75)

# Another way we can control a parameter
nlp.remove_pipe("neuralcoref")  # This remove the current neuralcoref instance from SpaCy pipe
coref = neuralcoref.NeuralCoref(nlp.vocab, greedyness=0.75)
nlp.add_pipe(coref, name='neuralcoref')
```

### Using the conversion dictionary to help resolve rare words

Here is an example on how we can use the parameter `conv_dict` to help resolving coreferences of a rare word like a name:

```python
import spacy
import neuralcoref

nlp = spacy.load('en')

# Let's try before using the conversion dictionary:
neuralcoref.add_to_pipe(nlp)
doc = nlp(u'Angela has a dog. She loves him')
doc._.coref_clusters
doc._.coref_resolved
# >>> [Angela: [Angela, She, him]]
# >>> 'Angela has a dog. Angela loves Angela'
# >>> Not very good...

# Here are three ways we can add the conversion dictionary
nlp.remove_pipe("neuralcoref")
neuralcoref.add_to_pipe(nlp, conv_dict={'Angela': ['woman', 'girl']})
# or
nlp.remove_pipe("neuralcoref")
coref = neuralcoref.NeuralCoref(nlp.vocab, conv_dict={'Angela': ['woman', 'girl']})
nlp.add_pipe(coref, name='neuralcoref')
# or after NeuralCoref is already in SpaCy's pipe, by modifying NeuralCoref in the pipeline
nlp.pipeline[-1][-1].set_conv_dict({'Angela': ['woman', 'girl']})

# Let's try agin with the conversion dictionary:
doc = nlp(u'Angela has a dog. She loves him')
doc._.coref_clusters
# >>> [Angela: [Angela, She], a dog: [a dog, him]]
# >>> 'Angela has a dog. Angela loves a dog'
# >>> A lot better!
```

## Using NeuralCoref as a server

A simple example of server script for integrating NeuralCoref in a REST API is provided as an example in [`examples/server.py`](examples/server.py).

There are many other ways you can manage and deploy NeuralCoref. Some examples can be found in [spaCy Universe](https://spacy.io/universe/).

## Re-train the model / Extend to another language

If you want to retrain the model or train it on another language, see our [training instructions](./neuralcoref/train/training.md) as well as our [blog post](https://medium.com/huggingface/how-to-train-a-neural-coreference-model-neuralcoref-2-7bb30c1abdfe)
