---
layout: post
title: "classification models"
date: 2013-12-15 00:35
comments: true
categories: research
---

During my past 3 years in career, following classifiers are often used for classification tasks.  

##Typcial classifiers comparision
{% img left /images/personal/research/classifiers/classifiers_compare.png 800 %}

##Decision Tree
Decision Tree is not a start-of-art model for classification or regression, and when there are huge features(say millions) it will take a long time for training.
But it may perform very well when the number of distinct features are limited, and the classification/regression task is obviously non-linear.

A typical scenario is multi-model fusion: you have trained multiple models for single task, and you want to generate the final prediction result using all these models.
Based on my past experiments, Decision Tree can out perform linear model(linear regression, logistic regression and so on) on many datasets.

##RDT, random forest, boosting tree
All of these 3 models are ensemble learning method for classification/regression that operate by constructing multiple Decision Tree at training time.
For RDT(random decision tree), only part of total samples are used to training each tree. And all features are considered for splitting.

Similar with RDT, random forest also use part of total sampels to construct each tree, but it also only use subset of features/dimisions for splitting. 
So random forest introduces more 'random' factors for training, and it may perform better when there are more noises in training set.

boosting tree is actually forward stagwise additive modeling with decision tree as base learner. And if you choose exponential loss function, then boosting tree becauses Adaboost with decision tree as base learner.
Here is one [slide](http://www.slideshare.net/guo_dong/additive-model-and-boosting-tree) about additive model and boosting tree. 

##Generalized linear model
One of the most popular generalized linear model is logistic regression, which is generalized linear model with inversed sigmoid function as the link function.
There are multiple different implementation for logistic regression, and here are some often used by me.

####Logistic regression optimized with SGD. 
It's very basic, so I ignore the details here

####OWLQN
It was proposed by Microsoft in paper [Orthant-Wise Limited-memory Quasi-Newton Optimizer for L1-regularized Objectives](http://research.microsoft.com/en-us/downloads/b1eb1016-1738-4bd5-83a9-370c9d498a03/) of ICML 2007. You can also find the source code and executable runner via this link.

This model is optimized by a method which is similar with L-BFGS, but can achieve sparse model with L1 regularizer. I recommend you try this model and compare with other models you are using in your dataset. 
Here are four reasons:

a. It's fast, especially when the dataset is huge; 
b. It can generate start-of-art prediction results on most dataset;
c. It's stable and there are few parameters need to be tried. Actaully, I find only regularization parameters can impact the performance obviously;
d. It's sparse, which is very important for big dataset and real product. (Of course, sparse is due to L1 regularizer, instead of the specific optimization method)

One problem is it's more challenge to implement it by yourself, so you need spend some time to make it support incremental update or online learning.

####FTRL
It was proposed by Google via paper <a href="http://static.googleusercontent.com/media/research.google.com/en/us/pubs/archive/41159.pdf">
Ad Click Prediction: a View from the Trenches</a> in 2013. I tried on my dataset, and this implementation can generate similar prediction performance with OWLQN.
It's quicker than OWLQN for training, and it's also sparse. One advantage is it's very easy to implement, and it support increamental update naturally.
One pain point for me is this model has 3-4 parameters need to be chosen, and most of them impact the prediction performance obviously.

####Ad predictor
This [paper](http://research.microsoft.com/pubs/122779/adpredictor%20icml%202010%20-%20final.pdf) was also proposed by Microsoft in ICML 2009.

One biggest different with upper 3 implementation is it's based on bayesian, so it's generative model. Ad predictor is used to predict CTR of sponsor search ads of Bing, and on my dataset, it could also achieve comparable prediction performance with OWQLN and FTRL.
Ad predictor model the weight of each feature with a gaussian distribution, so it natually supports online learning. And the prediction result for each sample is also a gaussian distribution, and it could be used to handle the exploration and exploitation problem.
See more details of this model in another [post](http://guod08.github.io/me/blog/2013/12/01/bayesian-ctr-prediction-for-bing/).

##Neural Network
ANN is so slow for training, so it's tried only when the dataset is small of medium. Another disadvantage of ANN is it's totally blackbox.

##SVM
SVM with kernel is also slow for training. You can try it with [libsvm](http://www.csie.ntu.edu.tw/~cjlin/libsvm/).
 
