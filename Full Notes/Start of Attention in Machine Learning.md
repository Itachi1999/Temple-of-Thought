27-04-2025 13:10
Status: #unrefined #blog
Tags: [[Artificial Intelligence]], [[Deep Learning]], [[Natural Language Processing]]


## Start of Attention in Machine Learning
#### Introduction

This blog explains the main concept behind the the paper that introduces attention mechanism in neural network models which later became cornerstone for the revolutionary paper "Attention is all you need. (Vaswani et al., 2017)"[{1}](#^1). I was not mature in Natural Language Processing (NLP) when I read about transformers and the Vaswani et al. 2017[{1}](#^1) back in 2022, so I thought that Transformers first introduced attention mechanism. I was ignorant for many years until now, I read the seminal paper which first introduces attention mechanism: Neural Machine Translation by Jointly Learning to Align and Translate(Bahdanau et al., 2015)[{2}](#^2) 

#### Background

So back in 2014, Neural Machine Translation was done using encoder-decoder based models where the encoder and decoder consists of RNN[{3}](#^3),[{4}](#^4). In these models, the encoder encodes the French(taking as an example) sentence into a single fixed length context vector`(c)`. If you look at figure 1, you would understand that if the length of French sentence was 40 instead of 4, it would have been the same fixed length vector `c` (for the decoder) which would be the output of the last RNN block. This has the following drawback:

> [!tip] Drawback
> The fixed length vector c is a bottleneck for longer sentences. It can not remember the context of the earlier parts of the sentence. [{3}](#^3) showed that these kind of encoder-decoder model's performance deteriorates rapidly for longer sentences.


<figure>
	  <img src="Encoder-Decoder without Attention.png" alt="Alt text" width="100%" height="200%">
	  <figcaption><em>Figure 1: Encoder-Decoder without Attention.</em></figcaption>
</figure>

#### Main Concept:

Overcoming these drawbacks mentioned in the previous section, the authors of the paper came up with a brilliant idea to "Learn to align and translate" simultaneously. This "Learning to align" is essentially what the authors called paying "attention" to certain parts of the input sentence. The following subsections discuss this idea in details.

> [!info] Two main ideas:
> - Encoder: A bidirectional RNN 
> - Decoder: Simulates searching through a source sentence while decoding each word in the translation.

##### Decoder: General Idea

The general idea of the decoder is that for generating target word $y_i$, it relies on distinct context vector $c_i$. 
$$
c_i = \sum_{j=1}^{T_x} \alpha_{ij}h_{ij} 
$$
The weight $\alpha_{ij}$ of each annotation ${h_j}$ is computed by:
$$
\alpha_{ij} = \frac{exp(e_{ij})}{\sum_{k=1}^{T_x} exp(e_{ik})}
$$
where 
$$
e_{ij} = a(s_{i-1}, h_j)
$$

is `alignment model`  

References:
1. <a id="1"></a>Attention is all you need. ^1
2. First attention paper ^2
3. Cho et al., 2014 ^3
4. Sutskever et al., 2014 ^4