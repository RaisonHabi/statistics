## t-test 
The t-test is any statistical hypothesis test in which the test statistic follows a Student's t-distribution under the null hypothesis.

A t-test is most commonly applied when the test statistic would follow a normal distribution if the value of a scaling term in the test statistic were known. When the scaling term is unknown and is replaced by an estimate based on the data, the test statistics (under certain conditions) follow a Student's t distribution. The t-test can be used, for example, to determine if the means of two sets of data are significantly different from each other.

也称为 Student's T Test，用来比较两样本平均值之间是否具有显著性差异；  
换句话说，t-test 让你知道这些差异是不是偶然发生的。

&nbsp;
## t-test 的类型
> **one-sample t-test，用来比较单个样本平均值和一个给定的平均值（理论值）**；  
**independent samples t-test（ unpaired two sample t-test），用来比较两组独立样本平均值**；  
**paired t-test，用来比较两个相关样本组之间的平均值**；

### 1.单样本T检验（one-sample t-test）
t=(m-μ)/(s/n(1/2))详见参考

m:样本的均值  
μ:一个已知的均值（理论值）  
s:样本的标准差（standard deviation）  
n:样本量
 
R 中的单样本 t-test：  
```
假如我们使用机器制造直径为 10.01mm 的零件，最近制造的一批零件有20个，我们想要知道这批零件的
均值（m）和理论均值μ=10.01是否有明显差异，也就是说生成的这批零件是否符合规格。

#R中使用 rnorm 函数生成符合正态分布的一组随机数
rnorm(n, mean, sd)
n:要生成随机数的数量
mean:要生成该组随机数的平均值
sd:要生成改组随机数的标准差

set.seed(233)
sample <- rnorm(30, mean = 10, sd = 0.05 )
head(sample)
[1] 10.044807 10.036622  9.984592  9.952429  9.982188 10.065807

原假设（Null hypothesis）：
H0: m = μ 该批零件均值m等于理论均值μ。
备择假设（Alternative hypothesis）：
H1: m ≠ μ 该批零件均值不等于理论均值μ；
H1: m > μ 该批零件均值大于理论均值μ；
H1: m < μ 该批零件均值小于理论均值μ。
```
#### 1) H0 : m = μ；H1 : m ≠ μ（双尾）
```
t.test(sample, mu = 10.01)  
#结果图见参考
```
从结果中可以看出 t = 0.47，自由度 df = n-1 =29，因为 p-value = 0.64 大于 0.05，所以差异不显著，所以接受原假设，认为该批零件直径的均值与理论均值相等，生产的零件符合标准。
#### 2）H0 : m = μ；H1 : m > μ（单尾）
从上面的双尾 t-test 看出，如果理论均值 μ = 10.01，那么该批零件符合标准。这里我们进行单尾 t-test，备择假设该批零件直径的均值 m 大于理论均值 μ，设置理论均值为 μ = 9.9。
```
t.test(sample, mu = 9.9, alternative = "greater")
#结果图见参考
```
结果中 p-value < 0.05，所以拒绝原假设，认为该批零件直径均值显著大于理论均值 9.9。
#### 3) H0 : m = μ；H1 : m < μ（单尾）
这里的备则假设该批零件直径的均值 m 小于理论均值 μ， 设置理论均值为 μ = 10.1。
```
t.test(sample, mu = 10.1, alternative = "less")
#结果图见参考
```
p-value < 0.05，拒绝原假设，接受备择假设认为该批零件直径的均值 m 小于理论均值 10.1，该批零件不合格。

### 2.独立样本T检验（independent samples t-test）
**独立样本 T 检验，首先要服从正态分布。根据两组数据的方差是否相同，分为 pooled variances 和 separate variances**。

#### 1）同方差（pooled variances）t-test
R 中的独立样本 t-test（同方差）  
```
假如我们使用两台机器生产零件，要比较两台机器生产的零件直径的平均值有没有差异，用 R 随机生成
符合正态分布的两组数据 A，B，分别表示 A 机器，B 机器。

set.seed(233)
sample_A <- rnorm(30, mean = 10, sd = 0.05 )
head(sample_A)
[1] 10.044807 10.036622  9.984592  9.952429  9.982188 10.065807

set.seed(333)
sample_B <- rnorm(30, mean = 10.01, sd = 0.05)
head(sample_B)
[1] 10.005859 10.106734  9.907436 10.023887  9.933702  9.996542

我们在这里只进行双尾检验，单尾检验可参考单样本 T 检验，假设如下：
原假设（H0）：两台机器生产的零件直径相同
备则假设（H1）：两台机器生产的零件直径不同
```
H0 : mA = mB；H1 : mA!=mB(同方差)
```
t.test(sample_A, sample_B, var.equal = T)
```
自由度 df=na+nb-2 = 58，因为 p-value > 0.05，所以不能拒绝原假设，两台机器生产的零件直径相同。
#### 2)R 中的独立样本 t-test（异方差）
```
我们随机产生服从正态分布的方差不同的两组数据：

set.seed(233)
sample_A <- rnorm(30, mean = 10, sd = 0.05 )
head(sample_A)
[1] 10.044807 10.036622  9.984592  9.952429  9.982188 10.065807

set.seed(333)
sample_B <- rnorm(30, mean = 10.01, sd = 1)
head(sample_B)
[1]  9.927188 11.944681  7.958710 10.287739  8.484039  9.740836
```
H0 : mA = mB；H1 : mA!=mB（异方差）
```
t.test(sample_A, sample_B, var.equal = F)
```
异方差独立样本 t-test 的自由度是一个估计值。

p-value > 0.05，不能拒绝原假设，两台机器生成的零件直径均值相同。
### 3.配对样本 T 检验（paired t-test）
**如果对相同的人或事，有两个测量值（before/after）选择配对 T 检验**。

要比较配对样本的均值，首先要计算出所有配对的差值 d，下面两个公式是等价的。
……详见参考

&nbsp;
## 总结
t -test 首先要服从正态分布，如果不服从正态分布，可以使用非参数检验，同时对于独立样本还要检验方差是否相同，对于配对样本，方差一般是不同的，这就涉及到了从如何检验服从正态分布和方差同质性了。

流程图详见参考1

&nbsp;
## reference
[t-test](https://zhuanlan.zhihu.com/p/38243421)  
