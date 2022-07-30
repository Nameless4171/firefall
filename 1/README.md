

#### 1. sm3生日攻击

sm3代码取自gmssl
https://github.com/gongxian-ding/gmssl-python

使用一个字典存放此前所计算出的hash值，这样可以通过与字典中的键进行匹配，得到一对碰撞。

```python
def BirthdayAtk(n):
    note = {}   #空字典，键值分别为前n比特的hash与对应的原串
    for i in range(0xffffffff):
        strl = [random.choice(digits) for i in range(32)] #生成32个随机数字组成的字符串
        strlst = "".join(strl)
        #print(strlst)
        strlst = bytes(strlst, encoding='UTF-8')    #为对应gmssl的接口，先行转为字节
        hashvalue = sm3_hash(bytes_to_list(strlst))
        #print(hashvalue)
        cmp = hashvalue[:n//4]
        if cmp in note: #如果找到了碰撞
            print("find a collision")
            print(note[cmp])
            print(strlst)
            break
        else:
            note[cmp] = strlst #否则将其写入字典
```
最终得到16bit的结果：
<img width="499" alt="pic" src="https://user-images.githubusercontent.com/105547492/181917677-9965960a-eab7-4141-b451-4204887f1f60.png">

