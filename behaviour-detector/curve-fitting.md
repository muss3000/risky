# Curve fitting for a drifting object

!DRAFT only!

The arrival of MEOSAR satellites on the beacon detection scene has brought about new and unexpected difficulties for Search and
Rescue operational staff. 

Without getting into a lot of detail (I don't know it!) the issue is that beacon detections arrive for a single beacon much more frequently (say every 30 seconds
) and each report comes with a 95% error margin value (called *accuracy* in the domain). The 95% error margin of a report may range from 120m for a good quality 
GPS measurement to 15nm for when Doppler measurements are part of the calculation. Search and Rescue staff want to know the best guessed historical
path for the target taking into account all the detections and with so many detections of varying quality it's quite difficult to sort out.

Dave Moten's investigations so far indicate that one viable approach to calculating a best path (and its associated error bounds) is by applying a technique called *non-linear weighted least-squares regression*. 

The problem does need precise definition though particularly as any optimization based on a distance function needs to make sense of distance across space *and* time and needs to consider the expected movement distribution (speed and direction changes) of the target in the domain. 

Below is a start on the problem only. Work on this issue has stopped due to other priorities.

## Definition
Each detection can be described by the tuple *(x, y, t, &delta;)* where 
* *x* and *y* are the position coordinates 
* *t* is the time of the detection (assumed 100% accurate)
* *&delta;* is the 95% error margin on the position 

The domain of values is assumed small enough that cartesian spatial distance calculation can be used instead of great-circle formulae.

So we have a set of tuples (the beacon detections for a target drifting at the ocean surface):

&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://www.codecogs.com/eqnedit.php?latex=\fn_jvn&space;(x_i,y_i,t_i,\delta_i)\&space;for\&space;i=1..n" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\fn_jvn&space;(x_i,y_i,t_i,\delta_i)\&space;for\&space;i=1..n" title="(x_i,y_i,t_i,\delta_i)\ for\ i=1..n" /></a>

In terms of the variance in spatial distance, we have the 95% position error margin *&delta;*. Assuming a normal distribution this suggests

&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://www.codecogs.com/eqnedit.php?latex=\fn_jvn&space;standard\&space;deviation\&space;\sigma&space;=&space;\frac{\delta}{3.92}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\fn_jvn&space;standard\&space;deviation\&space;\sigma&space;=&space;\frac{\delta}{3.92}" title="standard\ deviation\ \sigma = \frac{\delta}{3.92}" /></a>

therefore 

&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://www.codecogs.com/eqnedit.php?latex=\fn_jvn&space;variance\&space;\sigma^2&space;=&space;\frac{\delta^2}{3.92^2}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\fn_jvn&space;variance\&space;\sigma^2&space;=&space;\frac{\delta^2}{3.92^2}" title="variance\ \sigma^2 = \frac{\delta^2}{3.92^2}" /></a>

We want to find a regression function **f** that is smooth (say with continous derivative) that takes a time as input and provides a position (x,y coordinate). This regression function would ideally minimize a cost function:

&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://www.codecogs.com/eqnedit.php?latex=\fn_jvn&space;\newline&space;Define\newline\newline&space;\indent&space;d_i(t)&space;=&space;\sqrt{(f(t)_x-x_i)^2&space;&plus;&space;(f(t)_y-y_i)^2}\newline\newline&space;Define\&space;Cost\&space;function\newline\newline&space;\indent&space;M(f)&space;=&space;\int_{t_1}^{t_n}(\sum_{i=1}^{n}&space;w_i(t)&space;.&space;d_i(t)^2)dt\newline&space;where\newline\newline&space;\indent&space;weight\&space;w_i(t)&space;=&space;w_{i,1}.w_{i,2}(t)\newline\newline&space;\indent&space;weight\&space;due\&space;to\&space;variance\&space;w_{i,1}=\sigma_i^{-2}\newline\newline&space;\indent&space;variance\&space;\sigma_i^2&space;=&space;\frac{\delta_i^2}{{3.92}^2}\newline\newline&space;\indent&space;weight\&space;due\&space;to\&space;time\&space;diff\newline\newline\indent\indent&space;w_{i,2}(t)=&space;\left\{\begin{matrix}&space;1&space;&&space;if\&space;d(i)=0\&space;and\&space;t&space;=&space;t_i\&space;,&space;\\(d_i(t)&space;&plus;&space;s|t-t_i|)^{-2}&space;&&space;otherwise&space;\end{matrix}\right.&space;\newline\newline&space;\indent&space;s=&space;\mu_{speed}&space;&plus;&space;2\sigma_{speed}" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\fn_jvn&space;\newline&space;Define\newline\newline&space;\indent&space;d_i(t)&space;=&space;\sqrt{(f(t)_x-x_i)^2&space;&plus;&space;(f(t)_y-y_i)^2}\newline\newline&space;Define\&space;Cost\&space;function\newline\newline&space;\indent&space;M(f)&space;=&space;\int_{t_1}^{t_n}(\sum_{i=1}^{n}&space;w_i(t)&space;.&space;d_i(t)^2)dt\newline&space;where\newline\newline&space;\indent&space;weight\&space;w_i(t)&space;=&space;w_{i,1}.w_{i,2}(t)\newline\newline&space;\indent&space;weight\&space;due\&space;to\&space;variance\&space;w_{i,1}=\sigma_i^{-2}\newline\newline&space;\indent&space;variance\&space;\sigma_i^2&space;=&space;\frac{\delta_i^2}{{3.92}^2}\newline\newline&space;\indent&space;weight\&space;due\&space;to\&space;time\&space;diff\newline\newline\indent\indent&space;w_{i,2}(t)=&space;\left\{\begin{matrix}&space;1&space;&&space;if\&space;d(i)=0\&space;and\&space;t&space;=&space;t_i\&space;,&space;\\(d_i(t)&space;&plus;&space;s|t-t_i|)^{-2}&space;&&space;otherwise&space;\end{matrix}\right.&space;\newline\newline&space;\indent&space;s=&space;\mu_{speed}&space;&plus;&space;2\sigma_{speed}" title="\newline Define\newline\newline \indent d_i(t) = \sqrt{(f(t)_x-x_i)^2 + (f(t)_y-y_i)^2}\newline\newline Define\ Cost\ function\newline\newline \indent M(f) = \int_{t_1}^{t_n}(\sum_{i=1}^{n} w_i(t) . d_i(t)^2)dt\newline where\newline\newline \indent weight\ w_i(t) = w_{i,1}.w_{i,2}(t)\newline\newline \indent weight\ due\ to\ variance\ w_{i,1}=\sigma_i^{-2}\newline\newline \indent variance\ \sigma_i^2 = \frac{\delta_i^2}{{3.92}^2}\newline\newline \indent weight\ due\ to\ time\ diff\newline\newline\indent\indent w_{i,2}(t)= \left\{\begin{matrix} 1 & if\ d(i)=0\ and\ t = t_i\ , \\(d_i(t) + s|t-t_i|)^{-2} & otherwise \end{matrix}\right. \newline\newline \indent s= \mu_{speed} + 2\sigma_{speed}" /></a>

The weight function was constructed by multiplying a weight due to variance (this is a [standard approach](https://onlinecourses.science.psu.edu/stat501/node/352)) with a weight due to the difference in time (for which I am not aware of a standard approach yet). The weight due to difference in time should be high for timestamped positions that imply a reasonable effective speed, and low for timestamped positions that imply an unreasonable effective speed. 

A minimal solution to this problem would provide the values of f at times T<sub>1</sub>,..,T<sub>n</sub> (the times of detections). A more sophisticated solution would provide a function that could be applied to any t between T<sub>1</sub> and T<sub>n</sub>.

# Implementation
Algorithm-wise looks like the *Levenberg-Marquardt* method could be used to solve the problem and *apache commons-math* is one java implementation of that algorithm. Another implemenation [ddogleg](https://github.com/lessthanoptimal/ddogleg) has an implementation that according to the project author would support the weight customisation proposed above. 

Given the weight due to difference in time may adversely affect the robustness of methods like *Levenberg-Marquardt* it might be advisable to code up a brute force algorithm that doesn't rely on the differentiability of M (the absolute value term could cause problems).
