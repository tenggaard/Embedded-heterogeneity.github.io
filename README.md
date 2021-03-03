# Embedded Understanding

Research project carried out by Thyge Enggaard under the supervision of Sune Lehmann and Morten Axel Pedersen. 

## About this project
How can we formaly study differences in how words are understood? In this project, I will attempt to utilize word embeddings to elicit differences in how words are used, and subsequently explore whether this can be interpreted as differences in understanding.


## Methodology
The embedding of a corpus is an attempt to represent (semantic) similarities between words based on the distribution (co-occurence) of words in the corpus. Here, I will briefly elaborate on three challenges when working with word embeddings, and how the focus on representing a particular corpus as well as possible might demand different solutions to these challenges compared to when the focus is on training a model, that should be relevant when applied to a different corpus or task:
1. Word can be similar in many respects, which is important to consider when interpreting the results (distances between words) 
1. Training an embedding requires making multiple choices 'in advance'
1. Using an embedding in a valid way requires taking its (potential) stochastic nature into account   

### Word similarity
While word embedding models often take syntactical relations as input (how 'directly' associated two words are - 'black' and 'car' might e.g. appear frequently in succession), the resulting embedding often capture paradigmatic relations (how 'equivalent' two words are - 'car' and 'bus' might e.g. be close to each other in the embedding). When interpreting distances in word embeddings (e.g. between two words, or between a word and a dimension spanned by other words), understanding in what sense words are 'close' or 'distant' can be important for the interpretation.      

### Embedding choices
Training word embeddings require making choices in three areas:
1. Corpus pre-processing: Embedding the 'raw' corpus can pose various technical challenges (e.g. rare words, large vocabulary). Many pre-processing steps can be applied, e.g. removing words deemed 'unimportant' (often so called stop words), lemmatizing/stemming words, removing words with few characters and/or removing numbers. In addition, given the analysis, certain distinctions might be relevant to make explicit, such as the part-of-speech or grammatical tense or voice.    
1. Embedding architecture: There are many ways to obtain embeddings. At a high-level, I distinguish between embeddings that are static (e.g. (SVD-)PPMI, SGNS, Fasttext, Glove) and contextual/dynamic (e.g. BERT). Within each of these, many other considerations apply, such as whether the embedding should consider subword-information (e.g. Fasttext) or not (e.g. SGNS).
1. Hyper parameters: Training word embeddings requires determining multiple hyper parameters in advance.
    1. Architecture parameters: The model for transforming words to vectors often involve several parameters. A common one is the window-size - how close (in number of tokens) must to word-types be in order to count as associated.
    1. Learning paramters: In addition, the learning procedure entails hyper parameters, such the number of epochs (the number of times the corpus is ingested) or the learning rate (how 'aggressively' the model updates its internal parameters).

Ideally, choices should be made based on an understanding of how each decision affects properties of the embedding.  To the best of my (limited) knowledge, there is however still significant uncertainty about many of these relations. Based on what I have read:
* TBD 

Hence, these choices are often made:
* based on how well they have worked in other applications, raising questions of generalizability
* by applying different options and comparing the results, raising questions of computational feasibility

### Stochatic embeddings
In addition to these explicit choices (often made implicit through default settings), word embeddings are stochastic - several of the steps involved rely on randomization (e.g. the initial allocation of word vectors, sampling of word pairs in a batch - this is not the case for PPMI embeddings). This leads to instability - applying the same set of choices multiple times will likely not lead to identical embeddings. 


## Analysis so far

The idea I currently pursue is a 'direct, global' comparison of two embeddings:
1. Train an embedding on two corpora 
1. Align the embeddings, by rotating one of the embeddings to best match the other (orthogonal transformation)
1. Identify words with highest and lowest aligned distance

Indirect alternative (i.e. measures that do not require explicit alignment) include:
* Indirect, global comparison: For each word, compare its distance to all other words between the two embeddings
* Indirect, local comparison: For each word, compare its neighboorhodd, e.g. the overlap among the N nearest neighbors


### Data
I currently apply my ideas to two corpuses, consisting of (almost) all comments in the subreddit r/Republican and r/Democrats.


### Pre-processing of comments
Currently, I preprocess as follows: 
* Remove comments that display as removed by the user
* Remove a few obvious bot comments
* Replace urls and usernames with a REM-token
* Remove 'r/' from subreddit names
* Replace underscore with space    
* Replace '!', '?' and '\n' (newline) with a dot ('.')
* Replace characters, that are neither letter or digit with space
* Remove multiple spaces
* Remove multiple dots
* Tokenize the text (currently using the crazy-tokenizer developed for reddit, based on spaCY - https://redditscore.readthedocs.io/en/master/tokenizing.html)
* Parse the tokenized text for lemma and part-of-speech (POS) (currently using Stanza - https://stanfordnlp.github.io/stanza/)
* Lemmatize and lowercase words
* Replace stop words and single-character words a REM-token     
* Append the POS-tag to each word ('dog' transformed to 'dog_NOUN')
    
TO DO
* Find a principled way to filter out bot comments
    * Option \#1: Identify groups of comments with high similarity (e.g. TF-IDF) and remove if they are from bots --> not feasible 

  
### Embeddings
I have obtained the following embeddings:
* SVD-PPMI
* Skip-gram with negative sampling (SGNS-version of word2vec), based on the Gensim-library.
* FastText


### Embedding properties - vector length, frequency and isotrophy
TBD


### Identify optimal rotation
Obtaining the optimal, distance-preserving rotation (i.e. the rotation that yields the lowest avg. rotated distance) is known as the Orthogonal Procustes Problem:
* Let E_r and E_d denote the unrotated republican and democratic embeddings respectively
* Let U, V.T be the matrixes obtained from the svd of the product of the embeddings, i.e. E_d.T x E_r = U x S x V.T
* The rotation matrix R, that minimizes the distance between E_r and E_d x R, has the analytical solution R = U x V.T 

TO DO
* Consider using the same distance measure when training the embedding and aligning the rotation 

### Rotated distances

