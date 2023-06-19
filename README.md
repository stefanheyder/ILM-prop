# Method

To predict the number of hospitalisations we consider the reporting process of both reported COVID-19 cases and reported hospitalisations.
Recall that the reporting date of a COVID-19 case is shared for both the case and its hospitalisation, i.e. the case and hospitalisation are linked through this date.

As hospitalisations are only available as $7$-day rolling sums, see Section \ref{data-parameters}, we use $7$-day rolling sums for daily reported incidences as well. 
To avoid dealing with the double weekday effect of both reporting date of the case and reporting date of the hospitalisation we divide the future hospitalisations we wish to predict into chunks of one week, which gets rid of the weekday effect for the hospitalisations.
This is depicted in the figure below.
Our prediction of each of these weekly chunks then consists of the fraction of hospitalisations of reported cases in the past.

!(Decomposition of the daily reported hospitalisation incidences into the <span style="color:orange">known incidences</span>, i.e. the **reporting triangle**, and <span style="color:green">the future weekly increments</span>. <span style="color:blue">The last increment</span> might not be a weekly one, but we expect few cases to occur for such long delays.)[reptri.png]

More formally, denote by $h_{t,d}$ the number hospitalisations with reporting date $t$ that are known $d$ days later. Unfortunately we only observe $$H_{t,d} = \sum_{s = t - 6}^{t} h_{s,d + (t - s)},$$ i.e. a weekly sum of reported hospitalisations.
On day $T$ our goal is to predict $H_{t,D}$ for large delays $D$ and days $t \leq T$, of course it suffices to predict $H_{t, D} - H_{t, T - t}$ and add the known $H_{t, T - t}$ to this prediction. 
We rewrite this into weekly telescoping sum

$$
H_{t,D} - H_{t,d} = \left(H_{t, d + 7} - H_{t,d}\right) + \left(H_{t, d + 14} - H_{t, d + 7}\right) + \dots + \left(H_{t,D} - H_{t, d + 7 K}\right),
$$

where $K = \lfloor (D -d) / 7 \rfloor$, reducing the task at hand to predict hospitalisations in the $k$-th week ahead, $H_{t, d + 7k} - H_{t, d + 7\cdot(k - 1)}$, $k = 1, \dots, K$.
To leverage known reported incidences, rewrite this as 
$$\underbrace{\frac{H_{t, d + 7k} - H_{t, d + 7\cdot(k - 1)}}{I_{t,d}}}_{=:p_{t,d,k}} I_{t, d}$$
where $I_{t,d}$ is the $7$-day case incidence with reporting date $t$ known at time $t + d$, i.e. the incidenct case analouge of $H_{t,d}$.

Assuming that the proportions $p_{t,d,k}$ change slowly over time $t$ we estimate them by 

$$
\begin{align}
\label{eq:predict_p_tdk}
\widehat {p_{t,d,k}} = \frac{H_{t - 7k, d + 7k} - H_{t - 7k, d + 7\cdot(k - 1)}}{I_{t - 7k,d}} = p_{t - 7k,d,k}
\end{align}
$$

and finally predict

$$
\begin{align}
\label{eq:predict_H_tD}
\widehat{H_{t,D}} = H_{t,d} + I_{t,d} \left(\widehat{p_{t,d,1}} + \dots + \widehat{p_{t,d,K}}\right).
\end{align}
$$

As hospitalisation is affected by age, we perform this procedure for all available age groups separately and finally aggregate over all age groups to obtain a nowcast for all age groups combined. 

This describes our point nowcast for $7$-day hospitalisations. 
To obtain uncertainty intervals we fit a normal (age groups 00-04 and 05-14) or lognormal (all other age groups) distribution to the past performance of our model. 
We chose these distributions based on explorative analysis and believe that these should be seen as heuristics rather than as a matter of fact, which is in line with the philosophy of our model to be as simple as possible.

Denote by $\hat H_{t,D,s}$ the nowcast made for date $t$ on date $s \geq t$. Starting with date $t + D$ the definite $H_{t,D}$ is known and we can estimate the absolute prediction error $\varepsilon_{t,s} = H_{t,D} - \hat H_{t,D,s}$ and the relative prediction error $\eta_{t,s} = \log \left( H_{t,D} - H_{t, s - t}\right) - \log \left( \hat H_{t,D,s} - H_{t, s- t} \right)$.
For the nowcast for date $t$ made on date $s$ we estimate the standard deviation $\hat\sigma$ of $\varepsilon_{t - D - i, s - D - i}$ or $\eta_{t - D - i, s - D - i}$ (age groups 00-04, 05-14 and others respectively), $i = 0, \dots, 27$ by its empirical counterpart.
The estimated predictive distribution which informs our prediction intervals is then $\mathcal N (\hat H_{t,D,s}, \sigma^2)$ (age groups 00-04 and 05-14) or $\mathcal{LN} \left( \log \left(\hat H_{t,D,s} - H_{t, s - t}\right), \sigma^2 \right) + H_{t, s - t}$ (all other agr groups).