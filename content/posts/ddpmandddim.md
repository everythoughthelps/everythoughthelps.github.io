+++
date = '2025-09-02T17:10:51+08:00'
draft = false
title = 'DDPM到DDIM的梳理'
categories = ["扩散模型"]
tags = ["ddpm", "ddim"]
comments = true
+++
生成模型本质上是在求或者拟合概率分布，在拟合的方法上有多种方法，DDPM就是其中一种，下面回忆DDPM中求解$p(x_{t-1}|x_t)$的步骤：

$$
\begin{align}   
    p(x_{t-1}|x_t) &= \frac{p(x_{t-1}, x_t)}{p(x_t)} \\
    &= \frac{p(x_t|x_{t-1}) p(x_{t-1})}{p(x_t)} \quad 因为正向过程中p(x_t|x_{t-1})已知，所以根据贝叶斯公式进行引入 \label{eq:introbeyes} \\
    &= \frac{p(x_t|x_{t-1}, x_0) p(x_{t-1}, x_0)}{p(x_t, x_0)} \quad 引入x_0来确定p(x_{t-1})和p(x_t) \label{eq:introducex0}
\end{align}
$$
稍加整理，因为我们要求$p(x_{t-1})$的分布，所以整理成$x_{t-1}$的二次项形式，按照正态分布的表达式展开可以得到p(x_{t-1}|x_t)的分布表达式：

$$
\begin{align}
    p(x_{t-1}|x_t) \propto exp -\frac{1}{2}[(\frac{\alpha_t}{1-\alpha_t}+\frac{1}{1-\bar{\alpha_{t-1}}})x^2_{t-1}-(\frac{2\sqrt{\alpha_t}}{1-\alpha_t}x_t+\frac{2\sqrt{\bar{\alpha_{t-1}}}}{1-\bar{\alpha_{t-1}}}x_0)x_{t-1}+C(x_t, x_0)] \label{eq:distributionofreverse}
\end{align}
$$
观察$\eqref{eq:distributionofreverse}$, 根据正态分布的公式$\mathcal{N}(x; \mu, \sigma^2) \propto exp[-\frac{(x-\mu)^2}{2\sigma^2}]$，可以得到$x_{t-1}$的均值和方差：

$$
\begin{align}
    \frac{1}{\sigma^2} = \frac{\alpha_t}{1-\alpha_t}+\frac{1}{1-\bar{\alpha_{t-1}}} \\
    \sigma^2 = \frac{(1-\alpha_t)(1-\bar{\alpha_{t-1}})}{1-\bar{\alpha_t}} \label{eq:sigma} \\
    \mu = \frac{\sqrt{\alpha_{t-1}^{-}}\left(1 - \alpha_t\right)}{1 - \bar{\alpha}_t} x_0 + \frac{\left(1 - \alpha_{t-1}^{-}\right)\sqrt{\alpha_t}}{1 - \bar{\alpha}_t} x_t \label{eq:mu}
\end{align}
$$
我们得到了$x_{t-1}$的均值$\eqref{eq:mu}$和方差$\eqref{eq:sigma}$，从这个分布中采样，即可获得$x_{t-1}$，但是式$\eqref{eq:mu}$中含有$x_0$，我们需要把$x_0$消掉，因为我们只是使用$x_0$来推导$x_{t-1}$的分布，在推理过程中，实际上我们只知道$x_t$，于是根据前向采样公式将$x_0$替换成$x_t$：

$$
\begin{align}
x_t = \sqrt{\bar{\alpha_t}}x_0 + \sqrt{1-\bar{\alpha_t}}\epsilon \\
x_0 = \frac{x_t - \sqrt{1-\bar{\alpha_t}}\epsilon}{\sqrt{\bar{\alpha_t}}}
\end{align}
$$
最终得到$x_{t-1}$的均值：

$$
\begin{align}
    \mu &= \frac{\sqrt{\alpha_{t-1}^{-}}\left(1 - \alpha_t\right)}{1 - \bar{\alpha}_t} x_0 + \frac{\left(1 - \alpha_{t-1}^{-}\right)\sqrt{\alpha_t}}{1 - \bar{\alpha}_t} x_t \\
        &= \frac{\sqrt{\alpha_{t-1}^{-}}\left(1 - \alpha_t\right)}{1 - \bar{\alpha}_t} \frac{x_t - \sqrt{1-\bar{\alpha_t}}\epsilon}{\sqrt{\bar{\alpha_t}}} + \frac{\left(1 - \alpha_{t-1}^{-}\right)\sqrt{\alpha_t}}{1 - \bar{\alpha}_t} x_t \\
        &= \frac{1}{\sqrt{\alpha_t}}(x_t-\frac{1-\alpha_t}{\sqrt{1-\bar{\alpha_t}}}\epsilon) \quad 这步推导很麻烦，但不难
\end{align}
$$
最后写出$x_{t-1}$的分布：

$$
\begin{equation}
    p(x_{t-1}|x_t) = \mathcal{N}(\frac{1}{\sqrt{\alpha_t}}(x_t-\frac{1-\alpha_t}{\sqrt{1-\bar{\alpha_t}}}\epsilon), \frac{(1-\alpha_t)(1-\bar{\alpha_{t-1}})}{1-\bar{\alpha_t}}) \label{eq:distribution}
\end{equation}
$$
我们有了$x_{t-1}$的最终分布，但是其均值中有$\epsilon$，这里的$\epsilon$就是我们要消去的噪声，我们只知道$\epsilon$符合标准高斯分布，但是具体的值我们不知道，所以我们根据$x_t$去预测$\epsilon$，根据$\eqref{eq:distribution}$得到$x_{t-1}$，然后再根据$x_{t-1}$去预测这一步的$\epsilon$，循环往复，这里使用神经网络来根据$x_t$预测$\epsilon$，假如一个神经网络可以完美的预测出前向过程的噪声，那么随便给一个标准高斯噪声，我们认为它也是可以进行预测其中的噪声的，于是设计出$\theta$参数化的神经网络和损失函数：

$$
\begin{equation}
    MSELoss = \left\|\boldsymbol{\varepsilon}-\boldsymbol{\varepsilon}_{\theta}\left(\bar{\alpha}_{t} \boldsymbol{x}_{0}+\bar{\beta}_{t} \boldsymbol{\varepsilon}, t\right)\right\|^{2}
\end{equation}
$$
最后进行一下总结，$x_t, x_0, \epsilon$是一个知二求三的过程，我们使用迭代的思想，一步一步从$x_t$去预测噪声得到$x_0$是更合理的，因为每一步的步长很小，但是直接从$x_t$预测$x_0$可不可以呢，答案是可以的，但是效果不好，这样的话模型就退化成GAN的生成器了。

随后我们趁热打铁，解析一下DDIM，我们发现生成模型最重要的就是求解$p(x_{t-1})$，即公式$\eqref{eq:distribution}$，我们通过贝叶斯进行推导的时候发现单纯根据$x_t$无法预测出$x_{t-1}$，于是引入$x_0$加上前向过程求解出了$p(x_{t-1})$。无法跳步的原因就在公式$\eqref{eq:introbeyes}$，你的结果依赖$p(x_t|x_{t-1})$，这一步是完全基于马尔可夫过程一小步一小步来的，所以你就不能跳步。DDIM的创新在于使用了待定系数法求解，没有再用$p(x_t|x_{t-1})$来求解$p(x_{t-1})$，我们知道$x_{t-1}$是由$x_t$和$x_0$按照一定的比例“调和"而成的，于是我们假设：

$$
\begin{align}
    p(x_{t-1}|x_t, x_0) \sim \mathcal{N}(kx_0+mx_t, \sigma^2) \\
    x_{t-1} = kx_0 + mx_t + \sigma\epsilon, \epsilon \sim \mathcal{N}(0,1)
\end{align}
$$
我们把$x_t$用前向公式$x_t = \sqrt{\bar{\alpha_t}}x_0+\sqrt{1-\bar{\alpha_t}}\epsilon$替换掉：

$$
\begin{align}
    x_{t-1} &= kx_0 + m\sqrt{\bar{\alpha_t}}x_0+m\sqrt{1-\bar{\alpha_t}}\epsilon' + \sigma\epsilon \label{eq:distributiont-1} \\
            &= (k+m\sqrt{\bar{\alpha_t}})x_0 + \epsilon', \epsilon' \sim \mathcal{N}(0, m^2(1-\alpha_t)+\sigma^2) 
\end{align}
$$
这里可以将$\epsilon$和$\epsilon'$合并是因为他们都服从标准正态分布。
因为$x_{t-1}=\sqrt{\bar{\alpha_{t-1}}}x_0+\sqrt{1-\bar{\alpha_{t-1}}}\epsilon$，使用待定系数法可以求解$k,m$:

$$
\begin{align*}
    m &= \frac{\sqrt{1-\bar{\alpha_{t-1}-\sigma^2}}}{\sqrt{1-\bar{\alpha_t}}} \\
    k &= \sqrt{\alpha_{t-1}^{-}} - \frac{\sqrt{1 - \alpha_{t-1}^{-} - \sigma^2}}{\sqrt{1 - \bar{\alpha}_t}} \sqrt{\bar{\alpha}_t}
\end{align*}
$$
于是我们将$m,k$带入$\eqref{eq:distributiont-1}$且再次用$x_t$将$x_0$替换掉，得到$x_{t-1}$的表达式，注意$\epsilon'$和$\epsilon$只是分布相同，但是他们并不是一个变量， $\epsilon'$是前向过程中的噪声，仍然使用一个参数化的神经网络来预测$\epsilon'$，而$\epsilon$是我们采样过程中引入的随机噪声：

$$
\begin{equation}
    x_{t-1} = \sqrt{\bar{\alpha}_{t-1}} \left( \frac{x_t - \sqrt{1 - \bar{\alpha}_t} \epsilon_\theta(x_t)}{\sqrt{\bar{\alpha}_t}} \right) + \sqrt{1 - \bar{\alpha}_{t-1} - \sigma^2} \epsilon_\theta(x_t) + \sigma \epsilon \label{sampleddim}
\end{equation}
$$
因为我们只是使用待定系数法求解了另一种$p(x_{t-1})$,所以训练过程与DDPM相同，直接拿过来$\epsilon_\theta(x_t)$来用就行，不同的仅仅是采样过程。同是因为我们抛弃了原来的马尔科夫假设，所以下标的迭代可以跨步。最后还有一个地方要讨论，观察$\eqref{sampleddim}$，其中的所有的量都是已知的，唯独$\sigma$，也就是采样过程中的随机性，我们要进行讨论。
可以发现$\sigma$是可以任意取值的。设置一个超参数$\eta$：

$$
\begin{equation}
    \sigma = \eta \sqrt{\frac{1-\bar{\alpha_{t-1}}}{1-\bar{\alpha_t}}\beta_t}
\end{equation}
$$
对比公式$\eqref{eq:sigma}$和上式，当$\eta=1$时，你会发现这就是DDPM计算的方差。
当$\eta=0$时，此时采样过程中的随机性完全消失，变成确定性采样。
