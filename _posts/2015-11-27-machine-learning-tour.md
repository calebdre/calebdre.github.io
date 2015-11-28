---
layout: post
title: Machine Learning Tour
---
   
Machine Learning is a branch of computer science that studies the design of algorithms which learn as you give it information. 
The field is highly related to staticstics as these algorithms rely on statistics concepts. The basic premise is that we can feed
the algorithm information, we can implement predictive modeling techniques into the algorithms so it 'learns.' When we
give it new information it uses what it has 'learned' to make a prediction or perform another task.  
  
This blog post provides an overview of some machine learning concepts and equations in order to give a general understanding of
what it entails. Let's get into it!

### The K-Nearest Neighbor (KNN)
The KNN algorithm is one of the simplest machine learning algorithms. In order to achieve learning,
we measures the distance between a query (unknown) scenario and a set of known scenarios.
Here's a graph to illustrate (plusses and minuses are types of outcomes):  
  
> ![KNN Graph](../images/knn1.jpg)   

Given some outcomes and a query scenario, the algorithm will pick the K (input by the user)
nearest scenarios (neighbors) to the query scenarios and determine their outcomes. It will 
determine that the outcome that occurs the most times around it will be the outcome of query
scenario.  
  
In order to write such an algorithm, we need to determine how to compute the distances:    

$$ \sum_{i=0}^N \sqrt{(\frac {x - \overline{x}}{\sigma (x) })\^2 - y\_i\^2} $$

where \\( x \\) is the original value and \\( \overline{x} \\) is the arithmetic mean of feature \\( x \\)
across the dataset. With this equation, we can create an algorithm by letting matrix \\( D = N \times P \\) 
represent our data where \\( P \\) scenarios \\( s\^1, ... , s\^P\\) where each senarion \\( s\^i \\) 
contains \\( N \\) features \\( s\^i = [ s\_1\^i , ... , s\_N\^i]\\).  
  
We can let vector \\(r\\) store the output values of \\(M\\) nearest neighbors to query scenario \\(q\\). Let vector \\(o\\) 
with length \\(P\\) accompeny the matrix, listing the output value \\(o\^i\\) for each scenario \\(s\^i\\). 
Then we can loop through the data set measuring the distance between the set and \\(q\\):  
$$ \text{if } q \text{ is not set or } q < d(q,s\^i):q \rightarrow d(q,s\^i), t \rightarrow o\^i $$
Calculate the arithmetic mean output across r like so:

$$ \overline{r} = \frac1M \sum_{i=1}^M r\_i $$  
Return \\(\overline{r}\\) as the output value for the query scenario \\(q\\).  
  
Some example applications of KNN are any nearest neighbor based content retrieval type problems. I.E. 
problems where we need to find the closest match of something. Such problems include anything from image recognition to data mining.

### Support Vector Machine 
SVMs (Supprt Vector Machine) are a bit more involed. This algorithm achieves learning by finding the best
hyperplane that separates all data points of one class from those of the other class. Support vectors are 
the data points that are closest to the separating hyperplane. Here's an illustration:
  
> ![SVM illustration](../images/svm1.png)  

We start putting together the algorithm by having \\(L\\) training points where each input \\(x\_i\\)
has \\(D\\) attributes (i.e. is of dimensionality \\(D\\) and is in one of the two classes \\(y\_i = -1 \text{ or } +1\\),
i.e. out training data is of the form:  
$$ \lbrace x\_i ,y\_i \rbrace \text{ where } i = 1 ... L, y\_i \in \lbrace -1 , 1 \rbrace , x \in \Re \^D $$

With this, we can implement an SVM by selecting variables \\(w\\) and \\(b\\) so that our training
data can be described by \:
$$ y\_i(x\_i \cdot w + b) - 1 \le 0 \text{   } \forall\_i $$

Considering just the hyperplane, we can describe the planes \\(H\_1\\) and \\(H\_2\\) that lie on 
the Support Vector points with:  
$$ x\_i \cdot b \le +1 \text{   for} H\_1 $$
$$ x\_i \cdot b \ge -1 \text{   for} H\_2 $$
  
The distances between the two planes and the hyperplane taken together is the SVM's margin. Our
goal is to orient the hyperplaceto be as far from the Support Vectors as possible; we need to maximize
the margin.  
  

### Naive Bayes
This algorithm is based on Bayes Theorum.  
##### Bibliography
[Statsoft](http://www.statsoft.com/Textbook/k-Nearest-Neighbors#classification)  

[Saravanan Thirumuruganathan](https://saravananthirumuruganathan.wordpress.com/2010/05/17/a-detailed-introduction-to-k-nearest-neighbor-knn-algorithm/)

[DataCamp](http://blog.datacamp.com/machine-learning-in-r/)

[MathWorks](http://www.mathworks.com/help/stats/support-vector-machines-svm.html)

[Tristan Fletcher](http://www.tristanfletcher.co.uk/SVM%20Explained.pdf)

[Ram Narasimhan](http://stackoverflow.com/a/20556654/2229572)