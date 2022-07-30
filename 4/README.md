

#### 4. sm3 merkle树

sm3代码取自gmssl
https://github.com/gongxian-ding/gmssl-python

hashing函数用于生成sm3的hash值
generation函数用于构造merkle树。由底而上进行计算。

```python
def hashing(strlst):
    strlst = bytes(strlst, encoding='UTF-8')    #为对应gmssl的接口，先行转为字节
    hashvalue = sm3_hash(bytes_to_list(strlst))
    return strlst


def generation(msg):
    ret = []
    #叶子
    for i in range(len(msg)):
        ret.append(hashing(msg[i]))
    layer = ret
    while len(layer) > 1:
        tmp = []
        if (len(layer)% 2) == 1:
            layer.append('')
        for i in range(0, len(layer), 2):
            tmp.append(hashing(layer[i] + layer[i+1]))
        layer = tmp
        ret.append(layer)
    return ret
```
代码可以直接运行。
<img width="1129" alt="pic" src="https://user-images.githubusercontent.com/105547492/181918306-bca91d17-920c-4b7e-861a-947490a64e9e.png">

