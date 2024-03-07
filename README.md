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
