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
	  <img src="Encoder-Decoder.svg" alt="Alt text" width="100%" height="200%">
	  <figcaption><em>Figure 1: Encoder-Decoder without Attention.</em></figcaption>
</figure>

#### Main Concept:

Overcoming these drawbacks mentioned in the previous section, the authors of the paper came up with a brilliant idea to "Learn to align and translate" simultaneously. This "Learning to align" is essentially what the authors called paying "attention" to certain parts of the input sentence. The following subsections discuss this idea in details.

> [!info] Two main ideas:
> - Encoder: A bidirectional RNN 
> - Decoder: Simulates searching through a source sentence while decoding each word in the translation.

<figure>
	  <img src="Encoder-Decoder Attention.svg" alt="Alt text" width="100%" height="200%">
	  <figcaption><em>Figure 1: Encoder-Decoder with Attention.</em></figcaption>
</figure>

##### Decoder: General Idea

The general idea of the decoder is that for generating target word $y_i$, it relies on distinct context vector $c_i$. 
$$
c_i = \sum_{j=1}^{T_x} \alpha_{ij}h_{ij} 
$$
where $h_{ij}$ is the annotation or output of the RNN block in the encoder for input $x_j$ in the $i^{th}$ iteration of decoding (generating the  $i^{th}$ word). The approach of taking a weighted sum of the all the annotations can be seen as computing the `expected annotation` where the probabilities are $\alpha_{ij} \in [0,1]$.  
The weight $\alpha_{ij}$ of each annotation ${h_j}$ is computed by:
$$
\alpha_{ij} = \frac{exp(e_{ij})}{\sum_{k=1}^{T_x} exp(e_{ik})}
$$
where 
$$
e_{ij} = a(s_{i-1}, h_j)
$$
is an `alignment model`, it scores on how input around position $j$ and the output at position $i$ match. The $s_{i-1}$ is the RNN hidden state decoder which produces $y_{i-1}$. The alignment model $a$ is trained on using a feedforward neural network. The annotation $h_{i}$ represents the importance/context words around position $j$ for the input sentence $x_j$. 

> [!info] Intuition
> The probability $\alpha_{ij}$ or its associated energy $e_{ij}$ represents importance of $h_j$ with respect to $s_{i-1}$ on deciding the next state $s_i$ in turn generating $y_i$.
> This theory implies that generating a word in a particular position ($y_i$) the decoder always looks at the parts of the sentence and decides where to put *attention* to.
> The encoder no longer encodes the whole sentence into a single fixed length vector. The decoder selectively retrieves parts of the sentence and creates variable context vector for every word.  

##### Encoder: Bidirectional RNN

RNN typically reads input sentences from start ($x_1$) to the end ($x_{T_x}$). But in that case the annotations $h_j$ only represent the summarization of the preceding words in the sentence. The author here proposed to use Bidirectional RNN (BRNN) for which the annotations $h_j$ not only represent the summarization of the preceding words, but also the following words. 
BRNN consists of a forward RNN's and a backward RNN's. The forward RNN reads the input sentence from $x_1$ to $x_{T_x}$ and creates hidden states
$$
(f_1, f_2, f_3, ..., f_{T_x})
$$
The backward RNN reads the input sentence from $x_{T_x}$ to $x_1$ and creates hidden states
$$
(b_{T_x}, b_{T_x-1}, b_{T_x-2}, ..., b_2, b_1)
$$
To obtain the annotation $h_j$ for word $x_j$, the forward hidden states and backward hidden states are concatenated:
$$
h_j = [f_j^T;b_j^T]^T
$$
RNNs have a tendency to represent recent inputs, so, the annotation $h_j$ will b focused on input words around $x_j$.
These annotations will be used in decoder and the alignment model to calculate the context vector ($c_i$).

#### Conclusion
write a conclusion

Stay tuned for the next blog where this paper is explained practically by programming it through PyTorch.

#### References:
1. <a id="1"></a>Attention is all you need. ^1
2. First attention paper ^2
3. Cho et al., 2014 ^3
4. Sutskever et al., 2014 ^4