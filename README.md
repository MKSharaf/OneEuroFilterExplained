# 1€ Filter Explained
## Introduction

Noisy real-time signals is a problem that plagues many systems. Noise causes a reductoin of accuracy by introducing an offset between the true and observed values, it can also cause jitter as a result of having many different observations of values for a single true one. We can minimize (or even remove) such signals by introducing filters, such as the 1€ Filter, but that comes with its own set of problems.

## The Jitter-Latency Tradeoff

The jitter-latency (more commonly known as lag) tradeoff is a problem that happens when implementing filters. Any filter introduces lag to a certain extent, where the observations become delayed a bit from their real-time counterparts, but the result of that is more accuracy and perception. This introduces a dilemma, where more complex and computionally heavy filters could result in less jitter, but more lag. This means that we have to take both effects in consideration. To find a good filter it must be accurate, and, to an extent, computaionally simple at the same time. If we are talking numbers, then lag should be below 60 ms, and as we said, any filter introduces lag and considering 40:50 ms of inherent lag, we only have 10:20 ms for jitter.

## Noise Visualized

To get a feel of how noise affects observations we can use a great [tool](https://gery.casiez.net/1euro/InteractiveDemo/) made by Jonathan Aceituno:

![Alt Text](https://i.imgur.com/UcBrq3L.gif)

In this gif, we have a black cursor, and some white dots with their corresponding signals with their the same colors.

The black cursor is our true value, while the white dots represent our observations. The reason the observations are going all over the place is that Gaussian noise is being added to the true value and the resulting value is our observations. This is also why the observations are going in both directions if we look at the signals.

Now, we can explain the signals that we see on the right. The black signal is our cursor, and as long as it doesn't move, it's signal will continue being a flat, smooth one. That is because the value isn't seeing any change. However, the white signal sees a lot of movements, and now we should already know that is because of the noise being added to every observation. This is a clear demonstration of noise and it's affect on a value we are observing. This could be considered a small problem if we aren't looking at real-time data, because we can use very complex and computionally heavy algorithms to minimize the noise, but with real-time data we have to take lag into consideration thus making noise a major problem to any real-time data observations.

Admittedly, this is a very small explanation to noise in general, but hopefully, this is all that we will need.

## Low-Pass Filters

Now that we understand noise and how it affects our observations, it is time for us to talk about how we can minimize this effect.

1€ Filter is used for human motion, and since noise in these movements typically forms high frequencies in the signal while actual limb movements cause lower frequencies, it only logical for us to use a low-pass filter. But, what excatly a low-pass filter? 

A simple explanation for low-pass filters is that they are filters which are used to only let signals with lower frequencies than a predetermined cut-off value pass, while attenuuating higher frequencies.

![lowpassfilter-formula1](https://github.com/MKSharaf/OneEuroFilterExplained/assets/135005981/6193033f-f696-4a43-a8f1-a9164505aaa7)

## Exponential Smoothing

Exponential smoothing is a kind of a low-pass filter that assigns exponentially decreasing weights to past observations over time. As we discussed already, exponential smoothing is a low-pass filter which means that is used to remove high frequencies. 

$y_t = ax_t + (1-a)y_{t-1}$

This is the overall formula for exponential smoothing, and now let's explain each variable in the formula: -

* $x_t$ is the input data, aka the observed data at time $t$
* $a$ is a smoothing factor, where $0 < a < 1$
* $y_{t-1}$ is the last smoothed observation that is the result of all the previous computations
* $y_t$ is the result of the new smoothed observation 

$a$ has no formal procedure to be chosen with. A large value for $a$ reduces the smoothing effect, while a small value increases it. This is because the closer $a$'s value gets to $1$, the less weight that will be given to the past obeservations to the point where if $a$ becomes $1$ the output will just be the current observation with no smoothing effect. The opposite is also true, the closer $a$'s value gets to $0$, the greater the smoothing effect will be, this also happens because lesser weight will be given to current observations which will make the smoothing effect less responsive to new observations.

This is basically how it also works as a filter. The closer $a$ will be to $0$, the more jitter that would be reduced, but at the same it would result in more lag since the outputs won't respond as quickly to new observations. 

### How it works?

So, how exactly does exponential smoothing gives exponentially decreasing weights to its past observations? We can see this effect by performing direct substitution to the original formula

$y_t = ax_t + (1-a)y_{t-1}$ <br/>
&emsp; $= ax_t + a(1-a)x_{t-1} + (1-a)^2y_{t-2}$ <br/>
&emsp; $= ax_t + a(1-a)x_{t-1} + a(1-a)^2x_{t-2} + (1-a)^3y_{t-3}$ &emsp; $etc.$

As we can see, $y_{t-1}$ holds the last smoothed observation and it eoncombeses all of the previous observations. This also shows how a past observation gets a smaller and smaller value as a result of the increasing power.

One last thing to do now for us to make sure that we grasp how exponential smoothing is works as a function is to see it work from the start

$y_0 = ax_0$ &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;&nbsp; $(1)$<br/>
$y_1 = ax_1 + (1-a)y_0$ &emsp; $(2)$<br/>
$y_2 = ax_2 + (1-a)y_1$ &emsp; $(3)$<br/>

Now when we substitute $(2)$ for $(3)$

$y_2 = ax_2 + (1-a)(ax_1 + (1-a)y_0)$<br/>
&emsp; $= ax_2 + a(1-a)x_1 + (1-a)^2y_0$ &emsp; $(4)

Finally, we substitute $(1)$
