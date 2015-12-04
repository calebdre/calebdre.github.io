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

### K-Nearest Neighbors (KNN)
The KNN algorithm is one of the simplest machine learning algorithms. In order to achieve learning,
we measures the distance between a query (unknown) scenario and a set of known scenarios.
Here's a graph to illustrate (plusses and minuses are types of outcomes):

![KNN Graph](../images/knn1.jpg)
> from [Statsoft](http://www.statsoft.com/Textbook/k-Nearest-Neighbors#classification)
Let's dive into it:  

Given some outcomes and a query scenario, the algorithm will pick the K (input by the user)
nearest scenarios (neighbors) to the query scenarios and determine their outcomes. It will
determine that the outcome that occurs the most times around it will be the outcome of query
scenario.

In order to write such an algorithm, we need to determine how to compute the distances:

$$ \sum_{i=0}^N \sqrt{(\frac {x - \overline{x}}{\sigma (x) })\^2 - y\_i\^2} $$

where \\( x \\) is the original value, \\( \overline{x} \\) is the arithmetic mean of feature \\( x \\)
across the dataset, and \\(\sigma (x)\\) is its standard deviation. With this equation, we can create an algorithm by letting matrix \\( D = N \times P \\)
represent our data where \\( P \\) scenarios \\( s\^1, \ldots , s\^P\\) where each senarion \\( s\^i \\)
contains \\( N \\) features \\( s\^i = [ s\_1\^i , \ldots , s\_N\^i]\\).

We can let vector \\(r\\) store the output values of \\(M\\) nearest neighbors to query scenario \\(q\\). Let vector \\(o\\)
with length \\(P\\) accompeny the matrix, listing the output value \\(o\^i\\) for each scenario \\(s\^i\\).
Then we can loop through the data set measuring the distance between the set and \\(q\\):
$$ \text{if } q \text{ is not set or } q < d(q,s\^i):q \rightarrow d(q,s\^i), t \rightarrow o\^i $$
Calculate the arithmetic mean output across r like so:

$$ \overline{r} = \frac1M \sum_{i=1}^M r\_i $$
Return \\(\overline{r}\\) as the output value for the query scenario \\(q\\).

Some example applications of KNN are any nearest neighbor based content retrieval type problems. I.E.
problems where we need to find the closest match of something. Such problems include anything from image recognition to data mining.

Here is an implementation of KNN [find the used dataset here](https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data):
{% highlight python %}
import csv
import math
import operator

def loadDataset(filename):
	trainingScenarios=[]

	with open(filename, 'rb') as csvfile:
	    lines = csv.reader(csvfile)
	    dataset = list(lines)

	    for x in range(len(dataset)-1):
	        for y in range(4):
	            dataset[x][y] = float(dataset[x][y])
	            trainingScenarios.append(dataset[x])

 	return trainingScenarios

def findEuclideanDistance(point1, point2, length):
	distance = 0

	for x in range(length):
		distance += pow((point1[x] - point2[x]), 2)

	return math.sqrt(distance)

def findKNearestNeighbors(trainingScenarios, queryScenario, k):
	distances = []
	length = len(queryScenario)-1

	for x in range(len(trainingScenarios)):
		dist = findEuclideanDistance(queryScenario, trainingScenarios[x], length)
		distances.append((trainingScenarios[x], dist))

	distances.sort(key=operator.itemgetter(1))
	neighbors = []

	for x in range(k):
		neighbors.append(distances[x][0])

	return neighbors

def predict(neighbors):
	classVotes = {}

	for x in range(len(neighbors)):
		response = neighbors[x][-1]

		if response in classVotes:
			classVotes[response] += 1
		else:
			classVotes[response] = 1

	sortedVotes = sorted(classVotes.iteritems(), key=operator.itemgetter(1), reverse=True)
	return sortedVotes[0][0]

# prepare data
trainingScenarios = loadDataset('iris.data')
k = 3
queryScenario = [6.4, 2.8, 5.6, 2.2]

#find KNN
neighbors = getKNeighbors(trainingScenarios, queryScenario, k)
result = predict(neighbors)
print('Prediction: ', result)
{% endhighlight %}

Let's break it down:
{% highlight python %}
def loadDataset(filename):
	trainingScenarios=[]

	with open(filename, 'rb') as csvfile:
	    lines = csv.reader(csvfile)
	    dataset = list(lines)

	    for x in range(len(dataset)-1):
	        for y in range(4):
	            dataset[x][y] = float(dataset[x][y])
	            trainingScenarios.append(dataset[x])

	return trainingScenarios
{% endhighlight %}
This function just loads data from a file [(this one)](https://archive.ics.uci.edu/ml/machine-learning-databases/iris/iris.data) into an array. We assume that it's  a csv with each line having this format:

```
scenariopoint1, scenariopoint2, scenariopoint3, scenariopoint4, outcome
```

We turn it into a dictionary, append it to an array, and return it.

{% highlight python %}
def findEuclideanDistance(point1, point2, length):
	distance = 0

	for x in range(length):
		distance += pow((point1[x] - point2[x]), 2)

	return math.sqrt(distance)
{% endhighlight %}
Here, we just implement the euclidean distance formula so we can use it later to find the nearest neighbors.

{% highlight python %}
def findKNearestNeighbors(trainingScenarios, queryScenario, k):
	distances = []
	length = len(queryScenario)-1

	for x in range(len(trainingScenarios)):
		dist = findEuclideanDistance(queryScenario, trainingScenarios[x], length)
		distances.append((trainingScenarios[x], dist))

	distances.sort(key=operator.itemgetter(1))
	neighbors = []

	for x in range(k):
		neighbors.append(distances[x][0])

	return neighbors
{% endhighlight %}
This is a simpler implementation of the above KNN formula; we don't try to find the arithmetic mean or the
standard deviation - we essentially assume that all of our numbers are on the same scale (which they are in this example).
We loop trough the training scenario set, calculating it's distance from the query scenario, adding each to an array.
Then it sorts the distances by ascending value, adds `k` number of neighbors to a list, and returns it.

{% highlight python %}
def predict(neighbors):
	classVotes = {}

	for x in range(len(neighbors)):
		response = neighbors[x][-1]

		if response in classVotes:
			classVotes[response] += 1
		else:
			classVotes[response] = 1

	sortedVotes = sorted(classVotes.iteritems(), key=operator.itemgetter(1), reverse=True)
	return sortedVotes[0][0]
{% endhighlight %}
Here, loop through each neighbor and give each scenario outcome a vote. We then return the outcome with
the highest votes.

{% highlight python %}
# prepare data
trainingScenarios = loadDataset('iris.data')
k = 3
queryScenario = [6.4, 2.8, 5.6, 2.2]

#find KNN
neighbors = getKNeighbors(trainingScenarios, queryScenario, k)
result = predict(neighbors)
print('Prediction: ', result)
{% endhighlight %}

We've finished declaring our functions and now we're ready to start the program. First we load the training dataset
into avariable and declare how many `k` neighbors we want. Then we create a query scenario to test against.
we use the `getKNearestNeighbors` function to get the `k` nearest neightbors, then feed it to the `predict` function
and print it out.

After running this, we should get `Iris-virginica` as the result.


### Support Vector Machine
SVMs (Supprt Vector Machine) are a bit more involed. This algorithm achieves learning by finding the best
hyperplane that separates all data points of one class from those of the other class. Support vectors are
the data points that are closest to the separating hyperplane. Here's an illustration:

![SVM illustration](../images/svm1.png)
> From [Math Works](http://www.mathworks.com/help/stats/support-vector-machines-svm.html)
Let's dive into it:  

We start putting together the algorithm by having \\(L\\) training points where each input \\(x\_i\\)
has \\(D\\) attributes (i.e. is of dimensionality \\(D\\) and is in one of the two classes \\(y\_i = -1 \text{ or } +1\\),
i.e. out training data is of the form:
$$ \lbrace x\_i ,y\_i \rbrace \text{ where } i = 1 \ldots L, y\_i \in \lbrace -1 , 1 \rbrace , x \in \Re \^D $$

With this, we can implement an SVM by selecting variables \\(w\\) and \\(b\\) so that our training
data can be described by \:
$$ y\_i(x\_i \cdot w + b) - 1 \le 0 \text{   } \forall\_i $$

Considering just the hyperplane, we can describe the planes \\(H\_1\\) and \\(H\_2\\) that lie on
the Support Vector points with:
$$ x\_i \cdot b \le +1 \text{   for } H\_1 $$
$$ x\_i \cdot b \ge -1 \text{   for } H\_2 $$

The distances between the two planes and the hyperplane taken together is the SVM's margin. Our
goal is to orient the hyperplaceto be as far from the Support Vectors as possible; we need to maximize
the margin.

In order to do this, we need to solve these problems:
$$ \overset{\text{max}}{\alpha} \left[ \sum_{i=1}^L \alpha\_i - \frac12 \alpha\^T H\alpha \right] \text{ s.t. } \alpha\_i \ge 0 \text{ } \forall\_i$$

and:

$$  \sum_{i=1}^L \alpha\_i y\_i = 0 $$

Where \\(\alpha \text{ } (\alpha\_i \ge 0 \text{ } \forall\_i )\\) is a Lagrange multiplier. We do this because we need to find
a \\(\alpha\\) which maximizes and a \\(w\\) and \\(b\\) which minimizes. This, however, is a convex quadratic optimization problem,
which we need a [Quadratic Programming solver to solve.](http://doc.cgal.org/latest/QP_solver/index.html) It will return
\\(\alpha\\) and \\(w\\). We can find \\(b\\) by using the equation:

$$ b = y\_s - \sum_{m\in S} \alpha\_m y\_m x\_m \cdot x\_s $$
where \\(S\\) is the set of indices of the Support Vectors. It is determined by finding the indices \\(i\\) where
\\(\alpha\_i \gt 0\\).

Now we can create an algorithm:

* Create \\(H\\), where \\(H\_{ij} = y\_i y\_j x\_i \cdot x\_j \\)
* Find \\(\alpha\\) and \\(w\\) according to the above equations using a QP solver
* Determine the set of Support Vectors \\(S\\) by finding the indices such that \\(\alpha \gt 0\\)
* Each new point \\({x}'\\) is classified by evaluating \\({y}' = sgn(w \cdot {x}' + b)\\)

Applications of SVMs include pattern recognition and other classification type problems. Here is an implementation
of an SVM classifier:

{% highlight python %}
import matplotlib.pyplot as plt
from sklearn import datasets
from sklearn import svm

digits = datasets.load_digits()
classifier = svm.SVC(C=100)

x = digits.data[:-1]
y = digits.target[:-1]

classifier.fit(x,y)

print("Prediction: ", classifier.predict(digits.data[-1])

plt.imshow(digits.images[-1], cmap=plt.cm.gray_r, interpolation="nearest")
plt.show()
{% endhighlight %}

Let's go through each line:
{% highlight python %}
import matplotlib.pyplot as plt
from sklearn import datasets
from sklearn import svm
{% endhighlight %}
Instead of starting from scratch, we're going to use [sci-kit learn](http://scikit-learn.org) to provide us the
algorithm and sample data sets to see how using tools to help with implementing machine learning algorithms can make things
a lot easier. We'll also use [matplotlib](http://matplotlib.org/) to help see our results.

{% highlight python %}
digits = datasets.load_digits()
classifier = svm.SVC(C=100)
{% endhighlight %}
sci-kit learn has a lot of data sets for us to play around with. We're going to use its
digits data set. This set includes lists of coordinates for examples of what numbers look like.
So when we give the SVM coordinates of numbers, it can tell us what number it is.
Then we set up the classifier to be used later on.

{% highlight python %}
x = digits.data[:-1]
y = digits.target[:-1]

classifier.fit(x,y)
{% endhighlight %}

`digits` has 1797 examples of numbers. Here, we set `x` equal to the coordinates of the numbers
and `y` to the actual numbers. We save the last one from both, however, so that we can give it to the
algorithm later to test whether or not it learned to recognize numbers or not. We then train the
classifier by telling it to match each coordinate with each number. At this point we have
essentially 'taught' the algorithm how to recognize numbers.

{% highlight python %}
print("Prediction: ", classifier.predict(digits.data[-1])

plt.imshow(digits.images[-1], cmap=plt.cm.gray_r, interpolation="nearest")
plt.show()
{% endhighlight %}
We now use pass the last coordinates that we had to the classifier's `predict` function and print it out
in order to test whether or not the algorithm worked. The last two lines use matplotlib to show us
and image of what the last coordinates had in order for us to look at it and find out if the number was
what the algorithm said it was. Here's the result:

![SVM Prediction](../images/svm3.png)
(what the algorithm predicted) from

![SVM data](../images/svm2.png)
(what we gave to the algorithm)

> [both from Harrison Kinsley](https://youtu.be/KTeVOb8gaD4?t=2m38s)

### Naive Bayes
This algorithm is based on Bayes' rule, which states that if we know the probability \\(P(B\text{ }|\text{ }A)\\)
then we can find out the probability \\(P(A\text{ }|\text{ }B)\\) in terms of \\(P(B\text{ }|\text{ }A)\\).

This algorithm achieves learning by taking the query predictors and comparing it to those of each of the known outcomes,
giving a probability rating for each. The outcome of the query is the one with the highest probability rating.

That probability is given by the equation:
$$ P(c \text{ }| \text{ }x) = \frac{P(x \text{ }| \text{ } c) P(c)}{P(x)} $$
Where \\(c\\) is the outcome and \\(x\\) is the predictor. So:

* \\(P(c\text{ }| \text{ }x)\\) is the probability of outcome \\(c\\) given predictor \\(x\\)
* \\(P(x\text{ }| \text{ }c)\\) is the is the probability of predictor \\(x\\) given outcome \\(c\\)
* \\(P(c)\\) is the original probability of \\(c\\)
* \\(P(x)\\) is the original probability of \\(x\\)

When we let
$$ D = \left[ n\^1\_{S\_1} \ldots n\^k\_{S\_d} \right] \text{ where } n\_p\^i \text{ is the outcome } n \text{ with the set of } S \text{ predictors } $$
we can use this algorithm to find out which \\(n\\) that the query \\(q\\) predictors can be classified as:

* loop through each \\(n\^i\_S\\) in \\(D\\) calculating the probability rating and store them in set \\(R\\):
$$ \prod_{i=1}^D \frac{P(q\^{i-1} \text{ } | \text{ } n) \cdot P(n)}{P(q)} $$
* return the highest rating in \\(R\\)

Here is an implementation of a Naive Bayes algorithm:
{% highlight python %}
import csv
import random
import math

def loadCsv(filename):
	lines = csv.reader(open(filename, "rb"))
	dataset = list(lines)
	for i in range(len(dataset)):
		dataset[i] = [float(x) for x in dataset[i]]
	return dataset

def splitDataset(dataset, splitRatio):
	trainSize = int(len(dataset) * splitRatio)
	trainSet = []
	copy = list(dataset)
	while len(trainSet) < trainSize:
		index = random.randrange(len(copy))
		trainSet.append(copy.pop(index))
	return [trainSet, copy]

def separateByClass(dataset):
	separated = {}
	for i in range(len(dataset)):
		vector = dataset[i]
		if (vector[-1] not in separated):
			separated[vector[-1]] = []
		separated[vector[-1]].append(vector)
	return separated

def mean(numbers):
	return sum(numbers)/float(len(numbers))

def stdev(numbers):
	avg = mean(numbers)
	variance = sum([pow(x-avg,2) for x in numbers])/float(len(numbers)-1)
	return math.sqrt(variance)

def summarize(dataset):
	summaries = [(mean(attribute), stdev(attribute)) for attribute in zip(*dataset)]
	del summaries[-1]
	return summaries

def summarizeByClass(dataset):
	separated = separateByClass(dataset)
	summaries = {}
	for classValue, instances in separated.iteritems():
		summaries[classValue] = summarize(instances)
	return summaries

def calculateProbability(x, mean, stdev):
	exponent = math.exp(-(math.pow(x-mean,2)/(2*math.pow(stdev,2))))
	return (1 / (math.sqrt(2*math.pi) * stdev)) * exponent

def calculateClassProbabilities(summaries, inputVector):
	probabilities = {}
	for classValue, classSummaries in summaries.iteritems():
		probabilities[classValue] = 1
		for i in range(len(classSummaries)):
			mean, stdev = classSummaries[i]
			x = inputVector[i]
			probabilities[classValue] *= calculateProbability(x, mean, stdev)
	return probabilities

def predict(summaries, inputVector):
	probabilities = calculateClassProbabilities(summaries, inputVector)
	bestLabel, bestProb = None, -1
	for classValue, probability in probabilities.iteritems():
		if bestLabel is None or probability > bestProb:
			bestProb = probability
			bestLabel = classValue
	return bestLabel

def getPredictions(summaries, testSet):
	predictions = []
	for i in range(len(testSet)):
		result = predict(summaries, testSet[i])
		predictions.append(result)
	return predictions

def getAccuracy(testSet, predictions):
	correct = 0
	for i in range(len(testSet)):
		if testSet[i][-1] == predictions[i]:
			correct += 1
	return (correct/float(len(testSet))) * 100.0

filename = 'pima-indians-diabetes.data.csv'
splitRatio = 0.67
dataset = loadCsv(filename)
trainingSet, testSet = splitDataset(dataset, splitRatio)
print('Split {0} rows into train={1} and test={2} rows').format(len(dataset), len(trainingSet), len(testSet))
# prepare model
summaries = summarizeByClass(trainingSet)
# test model
predictions = getPredictions(summaries, testSet)
accuracy = getAccuracy(testSet, predictions)
print('Accuracy: {0}%').format(accuracy)
{% endhighlight %}

> from [Jason Brownlee](http://machinelearningmastery.com/naive-bayes-classifier-scratch-python/)  
  
  
Let's dive into it
{% highlight python %}
def loadCsv(filename):
	lines = csv.reader(open(filename, "rb"))
	dataset = list(lines)
	for i in range(len(dataset)):
		dataset[i] = [float(x) for x in dataset[i]]
	return dataset

def splitDataset(dataset, splitRatio):
	trainSize = int(len(dataset) * splitRatio)
	trainSet = []
	copy = list(dataset)
	while len(trainSet) < trainSize:
		index = random.randrange(len(copy))
		trainSet.append(copy.pop(index))
	return [trainSet, copy]

def separateByClass(dataset):
	separated = {}
	for i in range(len(dataset)):
		vector = dataset[i]
		if (vector[-1] not in separated):
			separated[vector[-1]] = []
		separated[vector[-1]].append(vector)
	return separated
{% endhighlight %}

These functions serve as data retreival. It pulls data from a csv files and converts it to a multidimentional array.
[Here is the data being used in this example](https://archive.ics.uci.edu/ml/machine-learning-databases/pima-indians-diabetes/pima-indians-diabetes.data), 
and [here is an explanation of that data](https://archive.ics.uci.edu/ml/machine-learning-databases/pima-indians-diabetes/pima-indians-diabetes.names)

The `separateByClass` function takes the dataset and organizes it into a dictionary where the keywords are the classes. We assume the
classes is the last entry in each dataset item. 
 
{% highlight python %}
def mean(numbers):
	return sum(numbers)/float(len(numbers))

def stdev(numbers):
	avg = mean(numbers)
	variance = sum([pow(x-avg,2) for x in numbers])/float(len(numbers)-1)
	return math.sqrt(variance)
{% endhighlight %}

 We use thise helper functions in order to do the math that the algorithm requires. 
 
 {% highlight python %}
 
 def summarize(dataset):
    summaries = [(mean(attribute), stdev(attribute)) for attribute in zip(*dataset)]
    del summaries[-1]
    return summaries

def summarizeByClass(dataset):
    separated = separateByClass(dataset)
    summaries = {}
    for classValue, instances in separated.iteritems():
        summaries[classValue] = summarize(instances)
    return summaries
 
{% endhighlight %}

These functions give us what we need to separate the training set into groups that we
can run the algorithm on.

{% highlight python %}

def calculateProbability(x, mean, stdev):
    exponent = math.exp(-(math.pow(x-mean,2)/(2*math.pow(stdev,2))))
    return (1 / (math.sqrt(2*math.pi) * stdev)) * exponent

def calculateClassProbabilities(summaries, inputVector):
    probabilities = {}
    for classValue, classSummaries in summaries.iteritems():
        probabilities[classValue] = 1
        for i in range(len(classSummaries)):
            mean, stdev = classSummaries[i]
            x = inputVector[i]
            probabilities[classValue] *= calculateProbability(x, mean, stdev)
    return probabilities

{% endhighlight %} 
 
 These functions implement the algorithm according to the formula above.
 
 {% highlight python %}
 def predict(summaries, inputVector):
    probabilities = calculateClassProbabilities(summaries, inputVector)
    bestLabel, bestProb = None, -1
    for classValue, probability in probabilities.iteritems():
        if bestLabel is None or probability > bestProb:
            bestProb = probability
            bestLabel = classValue
    return bestLabel

def getPredictions(summaries, testSet):
    predictions = []
    for i in range(len(testSet)):
        result = predict(summaries, testSet[i])
        predictions.append(result)
    return predictions
 {% endhighlight %}
 
 These functions run the algorithm. Onces we use the first functions to get the data that
 we need and format it in a way that the algorithm can comsume, we use these functions to 
 perform the algorithm on the data set and get the results.
 
 {% highlight python %}
 
 def getAccuracy(testSet, predictions):
    correct = 0
    for i in range(len(testSet)):
        if testSet[i][-1] == predictions[i]:
            correct += 1
    return (correct/float(len(testSet))) * 100.0
 
 {% endhighlight %}
 
 This function takes the predictions that we got and tests them against the actual values.
 We use this to see how accurate our algorithm is. 
 
 
 {% highlight python %}
splitRatio = 0.67
dataset = loadCsv('pima-indians-diabetes.data.csv')
trainingSet, testSet = splitDataset(dataset, splitRatio)
print('Split {0} rows into train={1} and test={2} rows').format(len(dataset), len(trainingSet), len(testSet))
# prepare model
summaries = summarizeByClass(trainingSet)
# test model
predictions = getPredictions(summaries, testSet)
accuracy = getAccuracy(testSet, predictions)
print('Accuracy: {0}%').format(accuracy)
 
 {% endhighlight %}
 
Here is where the program start to actually run. First we declare the `splitRatio`. This is the ratio
of data/test that we want. We're saying we want 67% of the data to be training and 33% to go to testing the accuracy 
of the algorithm.  
  
We get the dataset [(find it here)](https://archive.ics.uci.edu/ml/machine-learning-databases/pima-indians-diabetes/pima-indians-diabetes.data) [and a description here](https://archive.ics.uci.edu/ml/machine-learning-databases/pima-indians-diabetes/pima-indians-diabetes.names)
and split it into the `trainingSet` and the `testSet`.  
  
We then run the algorithm and test the accuracy. If you ran it, you should get ~76% 
 
### Conclusion
These are just a few of the algorithms that can achieve learning. Machine learning can be a
very useful tool in solving problems where predictive modeling can be utilized. It provides
another way to reason about a problem.  

##### Bibliography
[Statsoft](http://www.statsoft.com/Textbook/k-Nearest-Neighbors#classification)

[Saravanan Thirumuruganathan](https://saravananthirumuruganathan.wordpress.com/2010/05/17/a-detailed-introduction-to-k-nearest-neighbor-knn-algorithm/)

[Jason](http://machinelearningmastery.com/naive-bayes-classifier-scratch-python/) [Brownlee](http://machinelearningmastery.com/tutorial-to-implement-k-nearest-neighbors-in-python-from-scratch/)

[DataCamp](http://blog.datacamp.com/machine-learning-in-r/)

[MathWorks](http://www.mathworks.com/help/stats/support-vector-machines-svm.html)

[Tristan Fletcher](http://www.tristanfletcher.co.uk/SVM%20Explained.pdf)

[Harrison Kinsley](https://youtu.be/KTeVOb8gaD4?t=2m38s)

[Ram Narasimhan](http://stackoverflow.com/a/20556654/2229572)

[Queen Mary University](https://www.eecs.qmul.ac.uk/~norman/BBNs/Bayes_rule.htm)

[Dr. Saed Sayad](http://www.saedsayad.com/naive_bayesian.htm)