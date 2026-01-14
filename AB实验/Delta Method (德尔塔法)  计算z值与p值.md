在电商 A/B 实验中，GMV 相关指标常表现为比例类指标（Ratio Metrics），如客单价（GMV/订单数）或人均 GMV（GMV/活跃用户数）。由于分母本身也是随机变量且与分子相关，传统的 t 检验（假设分母固定）会低估方差。Delta Method（德尔塔法） 通过泰勒展开的一阶近似来估算这种复杂随机变量的方差，从而准确计算 Z 值和 P 值。 

以下是基于 2026 年主流实验平台（如 Statsig、eBay）所采用的标准计算流程：   
### 1. 指标定义 
设 X 为分子（如总 GMV），Y 为分母（如总用户数或订单数）。比率指标为 R = mu_x/mu_y。   
### 2. 计算比率指标的方差 (Delta Variance) 
根据 Delta Method，比率指标 R 的方差估计公式如下：Var(R) 约等于 Var(X)/(mu_y^2) + Var(Y) * (mu_x^2) / (mu_y^4) - 2 * mu_x * Cov(X,Y) / (mu_y^3)  
其中：   
* Var(X) 和 Var(Y) 分别是分子和分母的样本方差。
* Cov(X,Y) 是分子和分母的协方差。
* 在实际计算均值差异时，需将上述方差除以样本量 n 得到标准误的平方。
### 3. 计算 Z 值 (Z-score) 
计算实验组（Treatment, t）与对照组（Control, c）的差异显著性：Z = (R_t-R_c)/sqrt(SE_t^2 + SE_c^2)  
其中 SE = sqrt (Var(R)/n) 为通过 Delta Method 计算出的标准误。 
### 4. 计算 P 值 (p-value) 
基于正态分布假设，对于双侧检验：P=2 * (1-Phi(|Z|))  
* Phi 是标准正态分布的累积分布函数（CDF）。
* 显著性判定： 若 P<0.05，则在 95% 置信水平下拒绝原假设，认为实验结果显著。 

Python 参考实现 
```
import numpy as np
from scipy.stats import norm

def delta_method_z_test(x, y):
    """
    x: 分子向量 (如每个用户的GMV)
    y: 分母向量 (如每个用户的订单数, 至少为1)
    """
    n = len(x)
    mu_x, mu_y = np.mean(x), np.mean(y)
    var_x, var_y = np.var(x, ddof=1), np.var(y, ddof=1)
    cov_xy = np.cov(x, y)[0][1]
    
    # Delta Method 计算方差
    var_r = (var_x / mu_y**2) + (var_y * mu_x**2 / mu_y**4) - (2 * mu_x / mu_y**3 * cov_xy)
    return mu_x / mu_y, var_r / n

# 分别计算对照组和实验组的 mean 和 se_sq
mean_c, se_sq_c = delta_method_z_test(gmv_c, count_c)
mean_t, se_sq_t = delta_method_z_test(gmv_t, count_t)

# 计算 Z 和 P
z_score = (mean_t - mean_c) / np.sqrt(se_sq_c + se_sq_t)
p_value = 2 * (1 - norm.cdf(abs(z_score)))
```
### 核心优势 
* 消除相关性偏置： 准确处理了用户行为中“买得多的人通常订单也多”这种分子分母正相关的情况。
* 计算高效： 相比 Bootstrap（自助法）需要成千上万次模拟，Delta Method 只需一次闭式计算，适合大数据量电商场景。 
