# 1€ Filter Explained
## Introduction

Noisy real-time signals is a problem that plagues many systems. Noise causes a reduction of accuracy by introducing an offset between the true and observed values, it can also cause jitter as a result of having many different observations of values for a single true one. We can minimize (or even remove) such signals by introducing filters, such as the 1€ Filter, but that comes with its own set of problems.

## The Jitter-Latency Tradeoff

The jitter-latency (more commonly known as lag) tradeoff is a problem that happens when implementing filters. Any filter introduces lag to a certain extent, where the observations become delayed a bit from their real-time counterparts, but the result of that is more accuracy and precision. This introduces a dilemma, where more complex and computionally heavy filters could result in less jitter, but more lag. This means that we have to take both effects in consideration. To find a good filter it must be accurate, and, to an extent, computationally simple at the same time. If we are talking numbers, then lag should be below 60 ms, and as we said, any filter introduces lag and considering 40:50 ms of inherent lag, we only have 10:20 ms for jitter.

## Noise Visualized

To get a feel of how noise affects observations we can use a great [tool](https://gery.casiez.net/1euro/InteractiveDemo/) made by Jonathan Aceituno:

![Alt Text](https://i.imgur.com/UcBrq3L.gif)

In this gif, we have a black cursor, and some white dots with their corresponding signals with the same colors.

The black cursor is our true value, while the white dots represent our observations. The reason the observations are going all over the place is that Gaussian noise is being added to the true value and the resulting value is our observations. This is also why the observations are going in both directions if we look at the signals.

Now, we can explain the signals that we see on the right. The black signal is our cursor, and as long as it doesn't move, its signal will continue being a flat, smooth one. That is because the value isn't seeing any change. However, the white signal sees a lot of movements, and now we should already know that is because of the noise being added to every observation. This is a clear demonstration of noise and its affect on a value we are observing. This could be considered a small problem if we aren't looking at real-time data, because we can use very complex and computationally heavy algorithms to minimize the noise, but with real-time data we have to take lag into consideration thus making noise a major problem to any real-time data observations.

Admittedly, this is a very small explanation to noise in general, but hopefully, this is all that we will need.

## Low-Pass Filters

Now that we understand noise and how it affects our observations, it is time for us to talk about how we can minimize this effect.

1€ Filter is used for human motion, and since noise in these movements typically forms high frequencies in the signal while actual limb movements cause lower frequencies, it is only logical for us to use a low-pass filter. But, what is exactly a low-pass filter? 

A simple explanation for low-pass filters is that they are filters which are used to only let signals with lower frequencies than a predetermined cut-off value pass, while attenuating higher frequencies.

![lowpassfilter-formula1](https://github.com/MKSharaf/OneEuroFilterExplained/assets/135005981/6193033f-f696-4a43-a8f1-a9164505aaa7)

## Exponential Smoothing

Exponential smoothing is a kind of a low-pass filter that assigns exponentially decreasing weights to past observations over time. As we discussed already, exponential smoothing is a low-pass filter which means that is used to remove high frequencies. 

$y_t = ax_t + (1-a)y_{t-1}&emsp;&emsp;(1)$

This is the overall formula for exponential smoothing, and now let's explain each variable in the formula: -

* $x_t$ is the input data, aka the observed data at time $t$
* $a$ is a smoothing factor, where $0 < a < 1$
* $y_{t-1}$ is the last smoothed observation that is the result of all the previous computations
* $y_t$ is the result of the new smoothed observation 

$a$ has no formal procedure to be chosen with. A large value for $a$ reduces the smoothing effect, while a small value increases it. This is because the closer $a$'s value gets to $1$, the less weight that will be given to the past obeservations to the point where if $a$ becomes $1$ the output will just be the current observation with no smoothing effect. The opposite is also true, the closer $a$'s value gets to $0$, the greater the smoothing effect will be, this also happens because lesser weight will be given to current observations which will make the smoothing effect less responsive to new observations.

This is basically how it also works as a filter. The closer $a$ will be to $0$, the more jitter that would be reduced, but at the same it would result in more lag since the outputs won't respond as quickly to new observations. 

### How does it work?

So, how exactly does exponential smoothing gives exponentially decreasing weights to its past observations? We can see this effect by performing direct substitution to the original formula

$y_t = ax_t + (1-a)y_{t-1}$ <br/>
&emsp; $= ax_t + a(1-a)x_{t-1} + (1-a)^2y_{t-2}$ <br/>
&emsp; $= ax_t + a(1-a)x_{t-1} + a(1-a)^2x_{t-2} + (1-a)^3y_{t-3}$ &emsp; $etc.$

As we can see, $y_{t-1}$ holds the last smoothed observation and it eoncombeses all of the previous observations. This also shows how a past observation gets a smaller and smaller value as a result of the increasing power.

One last thing to do now for us to make sure that we grasp how exponential smoothing works as a function is to see it work from the start

$y_0 = ax_0$ &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;&nbsp; $(2)$<br/>
$y_1 = ax_1 + (1-a)y_0$ &emsp; $(3)$<br/>
$y_2 = ax_2 + (1-a)y_1$ &emsp; $(4)$<br/>

***We substitute*** $(3)$ ***for*** $(4)$

$y_2 = ax_2 + (1-a)(ax_1 + (1-a)y_0)$<br/>

***Now let's simplify*** 

$y_2 = ax_2 + a(1-a)x_1 + (1-a)^2y_0$ &emsp; $(5)$

***We substitute*** $(2)$ ***for*** $(5)$

$y_2 = ax_2 + a(1-a)x_1 + (1-a)^2(ax_0)$<br/>

***Now for our final simplification*** 

$y_2 = ax_2 + a(1-a)x_1 + a(1-a)^2x_0$<br/>

I hope that by now you have a good understanding of how the exponential filter works. I know that the last section had a bit of math about exponential smoothing, but trust me, this will be useful for understanding 1€ Filter since it uses it, and we will see how.

## 1€ Filter

1€ Filter stands out because of its ability of adapting its cutoff frequency to an estimate of the signal's speed, aka derivative value, for each new sample. We will start with the algorithm's formulas, and then explain them afterwards.

$a  = {1 \over {1 + \huge{τ \over {T_e}}}}$ &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp; $(6)$<br/>
$τ  = {1 \over {2πf_c}}$ &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&ensp;&nbsp; $(7)$ <br/> 
$f_c = f_{c_{min}} + β|Y|$ &emsp;&emsp;&emsp;&emsp;&emsp;&emsp; $(8)$ <br/>  
$y_t = (x_t + {τ \over T_e}\normalsize y_{t-1}){1 \over {1 + \huge{τ \over {T_e}}}}$  &emsp; $(9)$ <br/>

### 1€ Filter is an Exponential Smoother

I already mentioned that 1€ Filter is an exponential smoother, but how does the forumla correlate to that of the exponential smoother?

We can derive the 1€ Filter formula by doing direct substitution on $(1)$. ***Let's start by substituting*** $(1)$ ***for*** $(6)$

$y_t = {1 \over {1 + \huge{τ \over {T_e}}}}\normalsize x_t + (1 - {1 \over {1 + \huge{τ \over {T_e}}}}\normalsize )y_{t-1}$ <br/>
&emsp; $= {1 \over {1 + \huge{τ \over {T_e}}}}\normalsize x_t + ({{1 + \huge{τ \over {T_e}}} \over {1 + \huge{τ \over {T_e}}}} - {1 \over {1 + \huge{τ \over {T_e}}}}\normalsize )y_{t-1}$

***Now we simplifiy***

$y_t = {1 \over {1 + \huge{τ \over {T_e}}}}\normalsize x_t + {{\huge{τ \over {T_e}}} \over {1 + \huge{τ \over {T_e}}}}\normalsize y_{t-1}$

***Finally, let's take*** $(6)$ ***as a common factor***

$y_t = (x_t + {τ \over T_e}\normalsize y_{t-1}){1 \over {1 + \huge{τ \over {T_e}}}}$

We successfully managed to derive 1€ Filter from Exponential Smoother, now all we have to do is explain the variables and how they contribute to the algorithm.

### Smoothing Factor

The first thing 1€ Filter tries to address is the challenge of the changing pace at which events can occur with an event-driven system. This can cause problems when it comes to filtering because the filter's response may not be aligned with the occurrence of the events. This could make the filter uncapable to adapt to possible fluctuations. To help with that, 1€ Filter takes into consideration the actual time interval between samples in its smoothing factor, $T_e$.

$T_e = T_t - T_{t-1}$

Now to explain $τ$, we must explain $(8)$ first.

$f_c = f_{c_{min}} + β|Y|$ 

* $f_{c_{min}}$ is the minimum cutoff frequency that is chosen at the start, it is a constant that must be $> 0$.
* $β$ is the speed coefficient that is also chosen at the start, it is responsible for controlling how much the signal's speed is going to effect the resulting $f_c$
* $Y$ is the signal's speed, which can be computed with something simply by calculating the derivative of the signal

$Y = \Huge{x_t - y_{t-1} \over {T_e}}$

Since all of that is out of the way, now we can understand $τ$. $τ$ is simply a time constant that changes with the changing of $f_c$. If $f_c$ increases, $τ$ decreases, and if it decreases, $τ$ increases.

$τ  = \Huge{1 \over {2πf_c}}$ <br/> 

Finally, we can now see how the smoothing factor, $a$, changes. When $f_c$ changes, it directly causes change in $τ$, and when that happens $a$ changes. This seems tiring to us, but it is a couple of simple computations that can be done rather quickly, this in fact, is one of the things that makes 1€ Filter unique.

### The Thought Behind it

It is best to describe this section with the example of the human body movements. The author acknowledges that finding a trade-off between jitter and lag is difficult when you have something that changes its speed regularly that is because the signal is more suspectable to noise at low speeds and not lag, and more sensitive to lag at high speeds and not noise. This means we have to care more about smoothing and not lag when the speed is slow, and we have to care more about the lag and not the noise when the speed is fast. You can try all of this yourself using the aforementioned [tool](https://gery.casiez.net/1euro/InteractiveDemo/) .

This fact of being unable to find a good way to balance between the two when monitoring something this versatile is what caused the need for an adaptive cut-off frequency. $f_c$ has an direct relationship with the smoothing factor, $a$, this means when $f_c$'s value is small, the smoothing effect is increased. As we discussed earlier, $f_c$ decreases when the speed decreases, and when $f_c$'s value is large, the smoothing effect is decreased to decrease lag. See $(8)$ and its explanation. 

