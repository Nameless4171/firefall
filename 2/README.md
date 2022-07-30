

#### 2. sm3 rho攻击

sm3代码取自gmssl
https://github.com/gongxian-ding/gmssl-python

首先先找到循环，再找到一对碰撞。

```python
def RhoAtk(n):
    strl = [random.choice(digits) for i in range(32)] #生成32个随机数字组成的字符串
    strlst = "".join(strl)
    x = bytes(strlst, encoding='UTF-8')
    x0 = x
    x_ = x
    while True: #find a loop
        x = sm3_hash(bytes_to_list(x))
        x_ = sm3_hash(bytes_to_list(x_))
        x_ = bytes(x_, encoding='UTF-8')
        x_ = sm3_hash(bytes_to_list(x_))
        if x[:n//4] == x_[:n//4]:
            break
        x = bytes(x, encoding='UTF-8')
        x_ = bytes(x_, encoding='UTF-8')
    x_ = bytes(x, encoding='UTF-8')
    x = x0
    while True: #find a collision
        tx = x; tx_ = x_
        x = sm3_hash(bytes_to_list(x))
        x_ = sm3_hash(bytes_to_list(x_))
        if x[:n//4] == x_[:n//4]:
            print("find a collision")
            print(tx)
            print(tx_)
            break
        x = bytes(x, encoding='UTF-8')
        x_ = bytes(x_, encoding='UTF-8')
```
代码可以直接运行。
最终得到16bit的结果：

