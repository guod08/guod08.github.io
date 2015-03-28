---
layout: post
title: "Bayesian CTR prediction of Bing"
date: 2013-12-01 22:07
comments: true
categories: research
---
Microsoft published a paper in ICML 2009 named 'Web-Scale Bayesian Click-Through Rate Prediction for Sponsored Search Advertising in Microsoft's Bing Search Engine', 
which is claimed won the competition of most accurate and scalable CTR predictor across MS. 
This article shows how to inference this model(let's call it Ad predictor) step-by-step.

##Pros. and Cons.
I like it because it's totally based on Bayesian, and Bayesian is beautiful. Online learning is naturally supported, and the precition accuracy is comparable with FTRL and OWLQN. And both training and prediction is light-weight and fast.
Btw: one shortage of this model is it's not sparse, which may be a big issue when applied on big dataset with huge amount of features.


##Inference using Expectation Propagation step by step
Firstly, following is the factor graph of ad predictor.
{% img left /images/personal/research/ep/adPredictor.png 1200 %}

{% img left /images/personal/research/ep/ep_factor_definition.png 1200 %}

{% img left /images/personal/research/ep/ep_step1.png 1200 %}

{% img left /images/personal/research/ep/ep_step2.png 1200 %}

{% img left /images/personal/research/ep/ep_step3.png 1200 %}

{% img left /images/personal/research/ep/ep_step4.png 1200 %}

{% img left /images/personal/research/ep/ep_step5.png 1200 %}

{% img left /images/personal/research/ep/ep_step6.png 1200 %}

{% img left /images/personal/research/ep/ep_step7_1.png 1200 %}

{% img left /images/personal/research/ep/ep_step7_2.png 1200 %}

{% img left /images/personal/research/ep/ep_step7_3.png 1200 %}

{% img left /images/personal/research/ep/ep_step7_4.png 1200 %}

{% img left /images/personal/research/ep/ep_step8.png 1200 %}

{% img left /images/personal/research/ep/ep_step9.png 1200 %}

{% img left /images/personal/research/ep/ep_step10.png 1200 %}

{% img left /images/personal/research/ep/ep_step11.png 1200 %}

{% img left /images/personal/research/ep/ep_step12.png 1200 %}

{% img left /images/personal/research/ep/ep_step13.png 1200 %}

For each sample, we can use the formula of step 13 to update the posterior parameter of W, which is very easy to be implemented.

##Prediction
After training, we can predict with following formula:
{% img left /images/personal/research/ep/ep_predict.png 1200 %}


##Prediction Accuracy
I compared it with FTRL and OWLQN on one dataset for age&gender prediction. AUC of this model is comparable with OWLQN and FTRL, so I recommend you have a try in your case.

##Insights
1. You can find variance of each feature increases after every exposure, which makes sense. 

2. This model shows samples with more features will have bigger variance, which does not make sense very much. I think the reason is we assume all the features are independent. Any insights from you? 