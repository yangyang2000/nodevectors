[![Build Status](https://travis-ci.com/VHRanger/nodevectors.svg?branch=master)](https://travis-ci.com/VHRanger/nodevectors)

This package implements fast/scalable node embedding algorithms. This can be used to embed the nodes in graph objects -- we support [NetworkX](https://networkx.github.io/) graph types natively.
    
You can also use it to efficiently embed arbitrary scipy [CSR Sparse Matrices](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html), though not all algorithms here are optimized for this usecase.

## Installing

`pip install nodevectors`

## Supported Algorithms

- [Node2vec](https://cs.stanford.edu/~jure/pubs/node2vec-kdd16.pdf)

- GGVec (paper upcoming)

- [GraRep](https://dl.acm.org/doi/pdf/10.1145/2806416.2806512)

- [ProNe](https://www.ijcai.org/Proceedings/2019/0594.pdf)

- [GLoVe](https://nlp.stanford.edu/pubs/glove.pdf). This is useful to embed word co-occurence in sparse matrix form.

- Any [Scikit-Learn API](https://scikit-learn.org/stable/modules/classes.html) compatible model that supports the `fit_transform` method with the `n_component` attribute (eg. all [manifold learning](https://scikit-learn.org/stable/modules/manifold.html#manifold) models, [UMAP](https://github.com/lmcinnes/umap), etc.)

## Quick Example:
```python
    import networkx as nx
    from nodevectors import Node2Vec

    # Test Graph
    G = nx.generators.classic.wheel_graph(100)
 
    # Fit embedding model to graph
    g2v = Node2Vec()
    # way faster than other node2vec implementations
    # Graph edge weights are handled automatically
    g2v.fit(G)
 
    # query embeddings for node 42
    g2v.predict(42)

    # Save and load whole node2vec model
    # Uses a smart pickling method to avoid serialization errors
    g2v.save('node2vec.pckl')
    g2v = Node2vec.load('node2vec.pckl')
    
    # Save model to gensim.KeyedVector format
    g2v.save_vectors("wheel_model.bin")
    
    # load in gensim
    from gensim.models import KeyedVectors
    model = KeyedVectors.load_word2vec_format("wheel_model.bin")
    model[str(43)] # need to make nodeID a str for gensim
    
```

## Embedding a large graph

NetworkX doesn't support large graphs (>500,000 nodes) because it uses lots of memory for each node. We recommend using [CSRGraphs](https://github.com/VHRanger/CSRGraph) (which is installed with this package) to load the graph in memory:

```
import csrgraph
import nodevectors

G = cg.read_edgelist("path_to_file.csv")
ggvec_model = nodevectors.GGVec() 
embeddings = ggvec_model.fit_transform(G)
```

The ProNE and GGVec algorithms are the fastest. GGVec uses the least RAM to embed larger graphs. Additionally here are some recommendations:

- Don't use the `return_weight` and `neighbor_weight` if you are using the Node2Vec algorithm. It necessarily makes the walk generation step 40x-100x slower.

- If you are using GGVec, keep `order` at 1. Using higher order embeddings will take quadratically more time. Additionally, keep `negative_ratio` low (~0.05-0.1), `learning_rate` high (~0.1), and use aggressive early stopping values. GGVec generally only needs a few (less than 100) epochs to get most of the embedding quality you need.

- If you are using ProNE, keep the `step` parameter low.

- If you are using GraRep, keep the default embedder (TruncatedSVD) and keep the order low (1 or 2 at most).

## Embedding a VERY LARGE graph

(Upcoming).

GGVec can be used to learn embeddings directly from an edgelist file when the `order` parameter is constrained to be 1. This means you can embed arbitrarily large graphs without loading them into RAM.


### Usage

The public methods are all exposed in the quick example. The documentation is included in the docstrings of the methods, so for instance typing `g2v.fit?` in a Jupyter Notebook will expose the documentation directly.

## Why is it so fast?

We leverage [CSRGraphs](https://github.com/VHRanger/CSRGraph) to do the random walks. This uses CSR graph representations and a lot of Numba usage.
