---
layout: post
title: "Gaussian and Truncated Gaussian"
date: 2013-12-02 18:30
comments: true
categories: 
---

Everybody knows about Gaussian distribution, and Gaussian is very popular in Bayesian world and even in our life. This article summaries typical operation of Gaussian, and something about Truncated Guassian distribution.

##Gaussian

###pdf and cdf
{% img left /images/personal/research/gaussian/gaussian_pdf.png 350 'Gaussian pdf' %}
{% img right /images/personal/research/gaussian/gaussian_cdf.png 350 'Gaussian cdf' %}
{% img /images/personal/research/gaussian/gaussian_1.png 1200 %}

###Sum/substraction of two independent Gaussian random variables
{% img /images/personal/research/gaussian/gaussian_plus2.png 1200 %}
Please take care upper formula only works when x1 and x2 are independent. And it's easy to get the distribution for variable x=x1-x2
See [here](http://www.dartmouth.edu/~chance/teaching_aids/books_articles/probability_book/Chapter7.pdf) for the detils of inference

###Product of two Gaussian pdf
{% img /images/personal/research/gaussian/gaussian_multiple2.png 1200 %}
Please take care x is no longer a gaussian distribution. And you can find it's very elegant to use 'precision' and 'precision adjusted mean' for Gaussian operation like multiply and division.
See [here](http://www.tina-vision.net/docs/memos/2003-003.pdf) for the detils of inference

###Division of two Gaussian pdf
{% img /images/personal/research/gaussian/gaussian_divide2.png 1200 %}

###Intergral of the product of two gaussian distribution
{% img /images/personal/research/gaussian/gaussian_integral.png 1200 %}


##Truncated Gaussian
{% img left /images/personal/research/gaussian/tg_pdf.png 350 'Gaussian pdf' %}
{% img right /images/personal/research/gaussian/tg_cdf.png 350 'Gaussian cdf' %}

Truncated Gaussian distribution is very simple: it's just one conditional (Gaussian) distribution. 
Suppose variable x belongs to Gaussian distribution, then x conditional on x belongs to (a, b) has a truncated Gaussian distribution.
{% img /images/personal/research/gaussian/tg_def.png 1200 %}

###Calculate expectation of Truncated Gaussian
{% img /images/personal/research/gaussian/tg_e1.png 1200 %}
{% img /images/personal/research/gaussian/tg_e2.png 1200 %}

###Calculate variance of Truncated Gaussian
{% img /images/personal/research/gaussian/tg_v1.png 1200 %}
{% img /images/personal/research/gaussian/tg_v2.png 1200 %}
{% img /images/personal/research/gaussian/tg_v3.png 1200 %}
