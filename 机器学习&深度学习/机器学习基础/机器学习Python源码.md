# 监督学习
## 代价函数
```python
def cost_function(x, y, w, b):  
    """  
    Computes the cost function for linear regression.  
    Args:
		x (ndarray (m,)): 数据，m个样本
		y (ndarray (m,)): 目标值
		w,b (scalar)    : 模型参数 
    Returns:
		total_cost (float): 使用 w，b 作为线性回归参数
		以拟合 x 和 y 中的数据点的成本
	""" 
	# number of training examples
	m = x.shape[0]  
		
    cost_sum = 0  
    for i in range(m):  
        f_wb = w * x[i] + b  
        cost = (f_wb - y[i]) ** 2  
        cost_sum = cost_sum + cost  
    total_cost = (1 / (2 * m)) * cost_sum  
		
    return total_cost
```

# 向量化
对比向量化和非向量化的效率
```python
import numpy as np
import time

def my_dot(a, b): 
	"""
	计算向量点乘
 
    Args:
      a (ndarray (n,)):  输入向量 
      b (ndarray (n,)):  输入向量
    
    Returns:
      x (scalar): 
    """
    x=0
    for i in range(a.shape[0]):
        x = x + a[i] * b[i]
    return x

np.random.seed(1)
a = np.random.rand(10000000)  # very large arrays
b = np.random.rand(10000000)

tic = time.time()  # capture start time
c = np.dot(a, b)
toc = time.time()  # capture end time

print(f"np.dot(a, b) =  {c:.4f}")
print(f"Vectorized version duration: {1000*(toc-tic):.4f} ms ")

tic = time.time()  # capture start time
c = my_dot(a,b)
toc = time.time()  # capture end time

print(f"my_dot(a, b) =  {c:.4f}")
print(f"loop version duration: {1000*(toc-tic):.4f} ms ")

del(a);del(b)  #remove these big arrays from memory
```

```
np.dot(a, b) =  2501072.5817
Vectorized version duration: 9.5129 ms 
my_dot(a, b) =  2501072.5817
loop version duration: 1729.9936 ms
```

# 梯度下降
## 计算梯度
```python
def gradient_function(x, y, w, b): 
    """
    计算线性回归梯度
    Args:
	    x (ndarray (m,)): 数据，m个样本
		y (ndarray (m,)): 目标值
		w,b (scalar)    : 模型参数
    Returns
		dj_dw (scalar): 关于参数w的代价的梯度
	    dj_db (scalar): 关于参数w的代价的梯度    
    """
    
    # Number of training examples
    m = x.shape[0]    
    dj_dw = 0
    dj_db = 0
    
    for i in range(m):  
        f_wb = w * x[i] + b 
        dj_dw_i = (f_wb - y[i]) * x[i] 
        dj_db_i = f_wb - y[i] 
        dj_db += dj_db_i
        dj_dw += dj_dw_i 
    dj_dw = dj_dw / m 
    dj_db = dj_db / m 
        
    return dj_dw, dj_db
```

## 梯度下降
```python
def gradient_descent(x, y, w_in, b_in, alpha, num_iters): 
    """
    执行梯度下降以拟合 w，b
    通过采用具有学习率 alpha 的梯度步长梯度下降num_iters次来更新 w，b
    
    Args:
	    x (ndarray (m,)): 数据，m个样本
		y (ndarray (m,)): 目标值
	    w_in,b_in (scalar): 初始模型参数  
	    alpha (float):     学习率
	    num_iters (int):   梯度下降迭代次数
      
    Returns:
	    w (scalar): 梯度下降后的参数更新值
	    b (scalar): 梯度下降后的参数更新值
    """
    
    w = copy.deepcopy(w_in) # 避免改变全局变量w_in
    b = b_in
    
    for i in range(num_iters):
        # 使用梯度下降计算梯度并更新参数
        dj_dw, dj_db = gradient_function(x, y, w , b)     

        # 更新参数
        b = b - alpha * dj_db                            
        w = w - alpha * dj_dw                            
        
        # 每隔10次打印一次，如果小于10次，则以同样多的迭代次数打印成本
        if i% math.ceil(num_iters/10) == 0:
            print(f"Iteration {i:4}:\ 
		            Cost {cost_function(x, y, w, b):0.2e} ",
                  f"dj_dw: {dj_dw: 0.3e}, dj_db: {dj_db: 0.3e}  ",
                  f"w: {w: 0.3e}, b:{b: 0.5e}")
 
    return w, b
```