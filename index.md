---
title       : A General Framework For Weighted Gene co-Expression Network Analysis
subtitle    : Bin Zhang & Steve Horvath
author      : Keith Hughitt
job         : 
framework   : io2012        # {io2012, html5slides, shower, dzslides, ...}
highlighter : highlight.js  # {highlight.js, prettify, highlight}
hitheme     : tomorrow      # 
widgets     : [mathjax]            # {mathjax, quiz, bootstrap}
github:
    user: khughitt
    repo: slidify-wgcna
mode        : selfcontained # {standalone, draft}
--- .segue .dark

<!-- Custom Styles -->
<style type='text/css'>
    slides > slide {
        height: 800px;
        margin-top: -400px;
    }
    img {
        max-height: 560px;
        max-width: 964px;
    }
    slide a {border-bottom: none;}
    .references li { font-size: 18px; }
</style>

<!-- Custom JavaScript -->
<script src="http://ajax.aspnetcdn.com/ajax/jQuery/jquery-1.7.min.js"></script>
<script type='text/javascript'>
$(function() {
    $("p:has(img)").addClass('centered');
});
</script>

<!-- Slides start here -->
## Overview

---

## WGCNA Overview

![WGCNA overview](assets/img/wgcna_overview.jpg)

<div style='float: right; width: 38%; font-size: 12px;'>(Langfelder & Horvath, 2008)</div>

---

## Constructing a co-expression network

1. Choose a <span class='blue'>similarity metric</span>, construct a similarity
   matrix <span class='blue'>$S$</span>.
2. Choose an <span class='blue'>adjacency function</span> (e.g. signum/power)
3. Use adjacency function to map from similarity matrix, 
   <span class='blue'>$S$</span> to adjacency matrix, <span class='blue'>$A$</span>.

## Module detection

Once a co-expression network has been constructed, WGCNA can be used to detect
module of genes with similar expression profiles.

1. Choose a <span class='blue'>node dissimilarity</span> measure.
2. Example: 1 - correlation
3. WGCNA method: <span class='blue'>1 - Topological Overlap</span>

--- .segue .dark

## Constructing a co-expression network ##

---

## Similarity matrix

The first step is to choose a similarity metric, e.g. |Pearson correlation|,
and use it to construct a similarity matrix, $S$.

$$
s_{ij} = |cor(i, j)|
$$

Where

$$
cor(X, Y) = \rho_{X,Y} = \frac{Cov(X,Y)}{\sigma_X \sigma_Y}
$$

The more similar a pair of gene's expression profiles are across time, the
higher this value will be (max=1).

By applying the metric to each pair of genes in the dataset, an $n \times n$
similarity matrix is produced.

---

## Similarity matrix

Heatmap for 250 most highly expressed genes.
![Correlation matrix](figure/correlation_matrix.png)

---

## Adjacency matrix

Once a similarity matrix has been constructed, this is converted into an
[adjaceny matrix](http://en.wikipedia.org/wiki/Adjacency_matrix), which defines 
the co-expression graph or network.

An <span class='blue'>adjacency function</span> is chosen which maps from
<span class='blue2'>co-expression similarities</span> to </span class='blue2'>
edge weights</span>.

There are two major types of adjacency functions, the choice of which
determines whether the resulting network will be weighted or unweighted.

1. <span class='red'>Unweighted (hard threshold)</span>
 - Remove all edges below a certain similarity cutoff; set everything else to 1.
 - [Sign (signum) function](http://en.wikipedia.org/wiki/Signum_function)
2. <span class='red'>Weighted (soft threshold)</span>
 - Choose a function which maps from $(0,1)$ to $(0,1)$.
 - [Sigmoid function](http://en.wikipedia.org/wiki/Sigmoid_function)
 - [Power function](http://en.wikipedia.org/wiki/Power_function)

---

## Unweighted Network
Signum Function
<div style='font-size: 14px;'>
 $$ a_{ij} = signum(s_{ij}, \tau) \equiv
   \left\{ 
      \begin{array}{l l}
        1 & \quad \text{if}\ s_{ij} \ge \tau\\
        0 & \quad \text{if}\ s_{ij} \lt \tau
   \end{array} \right.
 $$
</div>

![plot of chunk signum_plot](figure/signum_plot.png) 


---

## Weighted Network
Sigmoid Function
<div style='font-size: 16px;'>
$$
a_{ij} = sigmoid(s_{ij}, \alpha, \tau_0) \equiv \frac{1}{1 + e^{-\alpha(s_{ij} - \tau_0)}}
$$
</div>

![plot of chunk sigmoid_plot](figure/sigmoid_plot.png) 


---

## Weighted Network
Power Function

<div style='font-size: 16px;'>
$$
  a_{ij} = power(s_{ij}, \beta) \equiv |s_{ij}|^\beta
$$
</div>

![plot of chunk power_plot](figure/power_plot.png) 

--- .seque .dark

## How do we select an appropriate adjacency function?

---

## Scale-free networks

- Many biological networks (including co-expression networks) are thought to
follow a [power law distribution](https://en.wikipedia.org/wiki/Power_law)^1.
- For co-expression networks with genes as nodes, the degree distribution $p(k)$
for genes follows:
$$
p(k) \sim k^{-\gamma}
$$
where $k$ is the number of connections to other genes.
- Networks which follow this degree distribution are referred to as "scale-free".
- Scale-free networks are <span class='blue'>robust to errors</span>, however,
- They are also <span class='blue'>vulnerable to attack</span> at particular 
  nodes (good for us!).

---

## Scale-free networks

The exponent $\gamma$ determines how quickly the distribution decays,
for example:

![plot of chunk powerlaw_plot1](figure/powerlaw_plot1.png) 


---

## Scale-free networks

- This property of biological networks can be used by us to help guide our
  selection of an adjacency function and parameters.
- The goal then becomes selecting a function and parameters such that the
  resulting co-expression network has the scale-free property.

![threshold selection](figure/choose_threshold.png)

--- .segue .dark

## Module detection ##

---

## Clustering gene expression profiles

K-means clustering of T. cruzi RNA-Seq time-course data.
![module detection](assets/img/tcruzi_cluster_1.png)

---

## Topological Overlap Matrix

>- The preferred method used by WGCNA to cluster gene expression profiles is
to first construct a similarity matrix using a measure called Topological
Overlap.
>- Topological overlap -> interconnectedness between two genes
>- The resulting Topological Overlap Matrix (TOM) is then subtracted from
   one to obtain a dissimilarity measure which can be used for clustering.
>- TOM $\Omega = [\omega_{ij}]$
<div class='blue'>
$$
\omega_{ij} = \frac{l_{ij} + a_{ij}}{\min{\{k_i, k_j\}} + 1 - a_{ij}}
$$
</div>
Where
<div class='blue'>
$$l_{ij} = \sum_u{a_{iu}a_{uj}}$$
</div>
And
<div class='blue'>
$$k_i = \sum_u{a_{iu}}$$

</div>

---

## Clustering

- [Average linkage hierarchical clustering](http://en.wikipedia.org/wiki/Hierarchical_clustering#Linkage_criteria)
  used to group genes based on their TOM dissimilarity.
- Gene modules then correspond to branches in the hierarchical clustering
  dendrogram.
- Smaller power law exponent: fewer modules, more genes
- Larger power law exponent: more modules, fewer genes
- For me: ~5-25 modules on average, depending on params.

---

## TOM Plot
TOM Plot can help us to visualize gene modules: red blocks along the diagonal
correspond to clusters of genes with a high topological overlap. These are our
clusters.

![TOM plot](figure/network_visualization_all.png)

---

## Module Eigengenes

Module eigengenes can be computed and a dendrodram of the eigengenes can be
constructed and used to guide the merging of similar modules.

![Module eigengenes](figure/eigengene_visualization.png)

---

## Comparison to other clustering methods

When comparing the results of WGCNA module detection to other commonly used
clustering methods, the results can be very different.

![Module dendrogram](figure/combine_modules.png)

---

## Exporting the network for visualization

In order to visualize the network in another application such as 
[Cytoscape](http://www.cytoscape.org/), it is first necessary to choose a
cutoff for which edges should be included.

- A <span class='blue'>Low cutoff</span> (e.g. topological overlap >= 0.15) will
  result in many edges and become very difficult to work with.
- A <span class='blue'>High cutoff</span> (e.g. 0.55) will have fewer edges, 
  but can still be in the tens or hundreds of thousands, and we may have lost 
  some important edges.

Alternatively, network modules can also be exported and visualized in isolation
with a higher cutoff value, however, inter-module edges will not lost in this
case.

---



