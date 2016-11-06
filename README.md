=======================================================
[PyDLM](https://pydlm.github.io/)
=======================================================

Welcome to [pydlm](https://pydlm.github.io/), a flexible, user-friendly and rich functionality time series modeling library for python. This library implementes the Bayesian dynamic linear model (Harrison and West, 1999) for time series data. Time series modeling is easy with `pydlm`. 
------------------------
Updates in Version 0.1.1
------------------------
* Fix bugs in latent states retrieval
* Rewrite all the get methods (simpler and concise). Allows easy fetching individual component.
* Add a longSeason component
* Add more plot functionalities
* Add the ribbon confidence interval
* Add a simple example in documentation for using pydlm

------------
Installation
------------
You can currently get the package from `pypi` by

      $ pip install pydlm

You can also get the latest from [github]
(https://github.com/wwrechard/PyDLM)

      $ git clone git@github.com:wwrechard/pydlm.git pydlm
      $ cd pydlm
      $ sudo python setup.py install

`pydlm` depends on the following modules,

* `numpy`     (for core functionality)
*  `matplotlib` (for plotting results)
* `Sphinx`    (for generating documentation)
* `unittest`  (for testing)

-----------------
A simple example
-----------------
we give a simple example on linear regression to illustrate how to use the `pydlm` for analyzing data. The data is generated via the following process
```
  >>> import numpy as np
  >>> n = 100
  >>> a = 1.0 + np.random.normal(0, 5, n) # the intercept
  >>> x = np.random.normal(0, 2, n) # the control variable
  >>> b = 3.0 # the coefficient
  >>> y = a + b * x
```
In the above code, `a` is the baseline random walk centered around 1.0 and `b` is the coefficient for a control variable. The goal is to
decompose `y` and learn the value of `a` and `b`. We first build the model
```
  >>> from pydlm import dlm, trend, dynamic
  >>> mydlm = dlm(y)
  >>> mydlm = mydlm + trend(degree=1, discount=0.98, name='a')
  >>> mydlm = mydlm + dynamic(features=[[i] for i in x], discount=1, name='b')
```
In the model, we add two components `trend` and`dynamic`. The trend `a` is one of the systematical components that can be used to characterize the intrisic property of a time series, and trend is particularly suitable for our case. The dynamic component `b` is modeling the regression component. We specify its discounting factor to be 1.0 means that we believe `b` should be a constant. For `a` we use 0.98 as we believe baseline can be gradually shift overtime. Then we fit the model
```
  >>> mydlm.fit()
```
After some information printed by the system, we are done (yeah! :p) and we can fetch and examine our results. We can first visualize the fitted results and see how well the model fits the data
```
  >>> mydlm.plot()
```
The result shows
<p align="center">
<img src="/doc/source/img/example_plot_all.png" width=70%/>
</p>

It looks pretty nice for the one-day ahead prediction accuracy. We can also plot the two coefficients `a` and `b` and see how they
change when more data is added
```
  >>> mydlm.turnOff('predict')
  >>> mydlm.plotCoef(name='a')
  >>> mydlm.plotCoef(name='b')
```
and we have
<p align="center">
<img src="/doc/source/img/example_plot_a.png" width=49%/>
<img src="/doc/source/img/example_plot_b.png" width=49%/>
</p>

We see that the latent state of `b` quickly shift from 0 (which is our initial guess on the parameter) to around 3.0 and the confidence
interval explodes and then narrows down as more data is added.

Once we are happy about the result, we can fetch the results:::
```
  >>> # get the smoothed results
  >>> smoothedResult = mydlm.getMean(filterType='backwardSmoother')
  >>> smoothedVar = mydlm.getVar(filterType='backwardSmoother')
  >>> smoothedCI = mydlm.getInterval(filterType='backwardSmoother')
  >>>
  >>> # get the coefficients
  >>> coef_a = mydlm.getLatentState(filterType='backwardSmoother', name='a')
  >>> coef_a_var = mydlm.getLatentCov(filterType='backwardSmoother', name='a')
  >>> coef_b = mydlm.getLatentState(filterType='backwardSmoother', name='b')
  >>> coef_b_var = mydlm.getLatentCov(filterType='backwardSmoother', name='b')
```
We can then use `coef_a` and `coef_b` for further analysis.

-------------------
Quick guide through
-------------------
Complex models can be constructed via simple operations in `pydlm`.
```
  >>> #import dlm and its modeling components
  >>> from pydlm import dlm, trend, seasonality, dynamic, autoReg, longSeason
  >>>
  >>> #randomly generate data
  >>> data = [0] * 100 + [3] * 100
  >>>
  >>> #construct the base
  >>> myDLM = dlm(data)
  >>>
  >>> #adding model components
  >>> myDLM = myDLM + trend(2, name='lineTrend') # add a second-order trend (linear trending)
  >>> myDLM = myDLM + seasonality(7, name='7day') # add a 7 day seasonality
  >>> myDLM = myDLM + autoReg(degree=3, data=data, name='ar3') # add a 3 step auto regression
  >>>
  >>> #show the added components
  >>> myDLM.ls()
  >>>
  >>> #delete unwanted component
  >>> myDLM.delete('7day')
  >>> myDLM.ls()
```
Users can then analyze the data with the constructed model
```
  >>> myDLM.fitForwardFilter()
  >>> myDLM.fitBackwardSmoother()
```

and plot the results easily
```
  >>> #plot the results
  >>> myDLM.plot()
  >>>
  >>> #plot only the filtered results
  >>> myDLM.turnOff('smoothed plot')
  >>> myDLM.plot()
  >>>
  >>> #plot in one figure
  >>> myDLM.turnOff('multiple plots')
  >>> myDLM.plot()
```
The three images show
<p align="center">
<img src="/doc/source/img/intro_plot_all.png" width=32%/>
<img src="/doc/source/img/intro_plot_wo_smooth.png" width=32%/>
<img src="/doc/source/img/intro_plot_in_1.png" width=32%/>
</p>
User can also plot the mean of a component (the time series value that
attributed to this component)
```
  >>> # plot the component mean of 'lineTrend'
  >>> myDLM.turnOn('smoothed plot')
  >>> myDLM.turnOff('predict')
  >>> myDLM.plot(name='lineTrend')
```
and also the latent states for a given component
```
  >>> # plot the latent states of the 'ar3'
  >>> myDLM.plotCoef(name='ar3')
```
which result in
<p align="center">
<img src="/doc/source/img/intro_plot_comp_mean.png" width=49%/>
<img src="/doc/source/img/intro_plot_state.png" width=49%/>
</p>

`pydlm` supports missing observations and also includes the discounting factor, which can be used to control how rapid the model should adapt to the new data (More details will be provided in the documentation)
```
  >>> data = [0] * 100 + [3] * 100
  >>> myDLM = dlm(data) + trend(2, discount = 1.0)
  >>> myDLM.fit()
  >>> myDLM.plot()
  >>>
  >>> myDLM.delete('trend')
  >>> myDLM = myDLM + trend(2, discount = 0.8)
  >>> myDLM.fit()
  >>> myDLM.plot()
```
The two different settings give different adaptiveness
<p align="center">
<img src="/doc/source/img/intro_discount_1.png" width=49%/>
<img src="/doc/source/img/intro_discount_09.png" width=49%/>
</p>

The filtered results and latent states can be retrieved easily
```
  >>> # get the filtered results
  >>> filteredMean = myDLM.getMean(filterType='forwardFilter')
  >>> filteredVar = myDLM.getVar(filterType='forwardFilter')
  >>> filteredCI = myDLM.getInterval(filterType='forwardFilter')
  >>>
  >>> # get the filtered mean for a given component
  >>> filteredTrend = myDLM.getMean(filterType='forwardFilter', name='lineTrend')
  >>>
  >>> # get the filtered latent states
  >>> allStates = myDLM.getLatentState(filterType='forwardFilter')
  >>> trendStates = myDLM.getLatentState(filterType='forwardFilter', name='lineTrend')
```
For online updates
```
  >>> myDLM = dlm([]) + trend(2) + seasonality(7)
  >>> for t in range(0, len(data)):
  ...     myDLM.append([data[t]])
  ...     myDLM.fitForwardFilter()
  >>> filteredObs = myDLM.getFilteredObs()
```  

Documentation
-------------
Detailed documentation is provided in [PyDLM](https://pydlm.github.io/) with special attention to the [User manual](https://pydlm.github.io/#dynamic-linear-models-user-manual).
