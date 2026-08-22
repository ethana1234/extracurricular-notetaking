[Impl Repo](https://github.com/ethana1234/hierarchical-nsw-python)

## Overview
[[Paper Summary]] goes over my notes for HNSWs, my goal with the implementation was to get a working python version that had decent recall@10 (>.95) using the [SIFT1M](http://corpus-texmex.irisa.fr/) dataset.
While I wanted to leverage numpy vectorization as much as possible, my initial version of the code was not properly vectorized outside of the brute force ground truth calculations and the recall function. I ended up trying to just the algorithm pseudocode directly to python, resulting in a pretty naive/slow implementation. 

## Evaluation
I used a few different evaluations, all focused on recall@K as the success. This includes testing random query vectors along with brute forcing K nearest neighbors and leveraging [faiss](https://github.com/facebookresearch/faiss)'s `HNSWIndex`, as well as just the ground truth data from the SIFT dataset.

## Performance
### Basic Python Impl
