---
layout:     post
title:      machine learning basic notes
subtitle:   from coursa/xuantianlin
date:       2020-03-26
author:     neverset
header-img: img/post-bg-kuaidi.jpg
catalog: true
tags:
    - machine learning
---

## Linear regression
during linear regression, both Ein and Eout converge to sigma square, so the expected error( square error) between them is show in the pic.
d: vc dimension
N: Number of data points
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200326231657.png)

## logistic regression

### Error definition
the output of logistic regression is generated by sigmoid function
the logistic regression error is defined by maximizing the likelihood the h equal f, which is called crosss entropy error. this error is related to w, xn and yn
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200326234337.png)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200326233017.png)

this is the mathmatical form of cross entropy error for calculation
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200331205714.png)

### Gradient decent and learning rate

the optimal gradient descent direction is opposite direction of Delta-Ein
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200326235248.png)

purple niu is teh learning rate
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200326235441.png)

the training process is described in the chart
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200326235751.png)

## Linear model
### advantages and disadvantages

demenstration of advantages and disadvantages between threee linear models
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327000104.png)

![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327000143.png)

### optimization process

what means stochastic gradient descent??

![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327000806.png)

SGD logistic regression can be seen as a softed PLA
stop criterien during training: 
*iteration times
niu (learning rate )is set as 0.1(experience value)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327001000.png)

### multiclass classification

two methods to deal with multiclass classification
* OVA(one versus all)[not recommended, not clearly seperatable]
* OVO(one versus one )

![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327001634.png)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327001548.png)

## nonlinear transformation

the principle to deal with nonlinear problem is to transform the data from nonlinear space to linear space, and use linear model to deal with it. However this will lead to more complex model, so there are some parameters( C, lambda) to restrict the complexity of the model (regularization???)


![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327002150.png)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327002314.png)

## overfitting

use more complex model is not good expecially when number of data points is small
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327002855.png)

four reasons leading to oversfitting:
* data size
* stochastic noise
* deterministic noise
* excessive power (model complexity)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327003033.png)

suggested ways to avoid overfitting:
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327003304.png)

![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327003444.png)

## regularization

loosing and softing the constraints to deal with the nonlinear problem
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327125121.png)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327125252.png)

augemented error definition and regularizer definition
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327125624.png)

regularization prefers smaller lambda and smaller w step during training
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327125913.png)

more effective transformation using legendre polynomials than naive polynomials
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327130423.png)

regularization confines to VC guarante with somehow enlarged error range
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327130904.png)

the model complexity is reduced due to regularization
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327151644.png)

if lambda increases, the effective vc dimension is reduced
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327151855.png)

principles to design regularizer:
* target dependent
* plausible
* friendly
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327152140.png)

L1 regularizer: time-saving in caculation, solution is sparse but not optimal
L2 regularizer: easy to optimize, precise in solution
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327152856.png)

if noise is large then more regularizer is needed
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327153217.png)

## validation

there are lots of hyper-parameters to choose in model selection, all of them are still guaranteed by hoeffdings rule
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327155416.png)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327155652.png)

the data set should be divided into training data set and validation data set, validation data set should not be used for training purpose except for cross validation
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327160642.png)

the traing data set is used to training all modells (g), the validation set is used to select the best modell according to error level
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327160947.png)

rule of thumb to divide data set into train set and validation set
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327161149.png)

the expected error using validation estimates Eout even better than purely Ein
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327161440.png)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327161738.png)

cross validation:

v-fold validation is preferred over single validation if computation allows
5 and 10 fold are good to use
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327161831.png)

## some good suggestions in machine learning

start with simple model to avoid overfit
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327162110.png)

avoid biased data 
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327162216.png)

avoid manual data snooping (danger of less generazation)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327162419.png)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327162549.png)
![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327162637.png)

### three learning principles

![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327162806.png)

![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327162830.png)

![](https://raw.githubusercontent.com/neverset123/cloudimg/master/Img20200327162856.png)


