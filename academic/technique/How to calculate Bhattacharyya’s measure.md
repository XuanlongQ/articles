# How to Calculate Bhattacharyya’s measure
The paper,`COSINE SIMILARITY APPROACHES TO RELIABILITY OF LIKERT SCALE AND ITEMS`, compares three method of calculate reliability of terms.

you could refer this paper, link is as follow: https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3202379


The author said that,Reliability of the test as per Bhattacharyya’s measure was found to be highest among the three methods discussed here.
The three methods are:
- Cronbach’s alpha
- Cosine similarity Method
- Bhattacharyya’s measure

In this article, I will give the code of `Cosine similarity Method` `Bhattacharyya’s measure`.

```python 

import numpy as np
def cosine_similarity(a,b):
    nominator = np.dot(a,b)

    a_norm = np.linalg.norm(a) 
    b_norm = np.linalg.norm(b)
    
    denominator = a_norm * b_norm
    cosine_similarity = nominator / denominator
    return cosine_similarity

def Bhattacharyya_measure(a,b):
    a_norm = np.linalg.norm(a) 
    b_norm = np.linalg.norm(b)
    ij_arr = []
    for i in range(len(a)):
        p_i = np.sqrt(a[i] / a_norm)
        p_j = np.sqrt(b[i] / b_norm)
        print("i is :",i)
        print("val is :", p_i , p_j)
        ij = p_i * p_j
        ij_arr.append(ij)
    mes = np.sum(ij_arr)
    return mes

a = np.array([1,2,3,4])
b = np.array([2,3,4,5])
x1 = Bhattacharyya_measure(a,b)
x2 = cosine_similarity(a,b)
print(x)
```

Notes:
when we calculate a word vector in embeddings, it may generate err, like:
- <font color = 'red'> "runtime warning: invalid value encountered in sqrt" </font>

The reason is there is negative value in embeddings, Thus I recommend you change to another `math` package in python.

```python
def Bhattacharyya_measure(a,b):
    a_norm = np.linalg.norm(a) 
    b_norm = np.linalg.norm(b)
    ij_arr = []
    for i in range(len(a)):
        p_i = np.lib.scimath.sqrt(a[i] / a_norm)
        p_j = np.lib.scimath.sqrt(b[i] / b_norm)
        ij = p_i * p_j
        ij_arr.append(ij)
    mes = np.sum(ij_arr)
    return mes

```

Plus, I also find some interesing article about Bhattacharyya distance and Bhattacharyya coefficients. Hope it is useful for you!
- [Understanding Bhattacharyya Distance and Coefficient for Probability Distributions](https://safjan.com/understanding-bhattacharyya-distance-and-coefficient-for-probability-distributions/) , his article is good, but when I try to replicate his code, I found there is no `distance` function in `from scipy.spatial import distance` which he cites in his articles.
- [Statistical Distances](https://www.kaggle.com/code/debanga/statistical-distances#Bhattacharyya-Distance), good paper only for dictionaries
- [Distance computations](https://docs.scipy.org/doc/scipy/reference/spatial.distance.html). For responding the first item, I check the scipy and i don't find the function.
- [pip install dictances](https://pypi.org/project/dictances/), A package for users, but I think there is something wrong in his `Bhattacharyya Distance` code. Just a simple comments.