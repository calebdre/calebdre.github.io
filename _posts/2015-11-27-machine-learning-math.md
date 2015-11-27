---
layout: post
title: Machine Learning & Math
---
   
Machine Learning is a branch of computer science that studies the design of algorithms that can learn as you give it information. 
The field is highly related to staticstics as these algorithms rely on statistics concepts. The basic premise is that we can feed
the algorithm information, we can implement predictive modeling techniques into the algorithms so it 'learn', so that when we
give it new information it can use what it has 'learned' to make a prediction or perform another task.  
  
This blog post will give an overview of some machine learning concepts and equations in order to give a general understanding of
what it entails. Let's get into it:

### The K-Nearest Neighbor Algorithm (KNN)
The KNN is one of the simplest machine learning algorithms. In order to achieve learning,
we measures the distance between an query (unknown) scenario and the set of known scenarios.
Here's a graph to illustrate (plusses and minuses are type of outcomes):  
  
> ![KNN Graph](../images/knn1.jpg)   

Given some outcomes and a query scenario, the algorithm will pick the K (input by the user)
nearest scenarios (neighbors) to the query scenarios and determine their outcomes. It will 
determine that the outcome that occurs the most times around it will be the outcome of query
scenario.  
  
In order to write an algorithm like that, we need to determine how to compute the distance:    

$$ \sum_{i=0}^N \lvert \frac {x - \overline{x}}{\sigma (x) } - y\_i \rvert $$

where \\( x \\) is the value and \\( \overline{x} \\) is the arithmetic mean of feature \\( x \\)
across the dataset 

##### Bibliography
http://www.statsoft.com/Textbook/k-Nearest-Neighbors#classification  

https://saravananthirumuruganathan.wordpress.com/2010/05/17/a-detailed-introduction-to-k-nearest-neighbor-knn-algorithm/

http://blog.datacamp.com/machine-learning-in-r/