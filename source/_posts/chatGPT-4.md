---
title: 'OpenAi.'
date: '2023/03/16 14:59:21'
categories:
   - openai
tags:
   - 人工智能
toc_level: 3
version: 3
---


![cover](images/chatGPT-4.jpeg)

众所周知，ChatGPT可以帮助研发人员编写或者Debug程序代码，但是在执行过程中，ChatGPT会将程序代码的一些相关文字解释和代码段混合着返回，如此，研发人员还需要自己进行编辑和粘贴操作，效率上差强人意，本次我们试图将ChatGPT直接嵌入到代码业务中，让ChatGPT生成可以直接运行的代码。

ChatGPT的主观回答问题

![](images/chatgtp.jpeg)

可以看到，就像上文所描述的那样，ChatGPT会将文字描述和代码片段混合着返回，其实对于真正的需求者来说，文字描述本身是不必要的，因为如果提问者不知道什么是布隆过滤器，也就不会提出布隆过滤器相关的代码需求。

再看ChatGPT返回的布隆过滤器代码：

```
import mmh3
from bitarray import bitarray

class BloomFilter:
    def __init__(self, size, hash_count):
        self.size = size
        self.hash_count = hash_count
        self.bit_array = bitarray(size)
        self.bit_array.setall(0)

    def add(self, item):
        for i in range(self.hash_count):
            digest = mmh3.hash(item, i) % self.size
            self.bit_array[digest] = 1

    def __contains__(self, item):
        for i in range(self.hash_count):
            digest = mmh3.hash(item, i) % self.size
            if self.bit_array[digest] == 0:
                return False
        return True
```

大体上，没有毛病。但是主观性太强，什么是主观性？就是ChatGPT其实不是站在需求者的视角来编写代码，而是站在自己的角度上，它没有考虑业务的上下文关系，也就是类和方法命名、方法参数、以及参数类型或者返回值以及类型，这些东西是否符合需求者当前的代码业务。

当然，这并不是ChatGPT的错，主要是输入的问题描述不够明确和详细，但如果每次都需要将代码业务逻辑转化为文字告诉ChatGPT，又有些画蛇添足，狗尾续貂之感。

### [](about:blank#%E5%9F%BA%E4%BA%8E%E4%B8%9A%E5%8A%A1%E9%85%8D%E7%BD%AEChatGPT "基于业务配置ChatGPT")基于业务配置ChatGPT

那么怎样将ChatGPT融入业务代码？首先创建Openai接入函数：

```
import openai

openai.api_key = "apikey"

def generate_code(func, docstring):
    init_prompt = "You are a Python expert who can implement the given function."
    definition = f"def {func}"
    prompt = f"Read this incomplete Python code:\n```python\n{definition}\n```"
    prompt += "\n"
    prompt += f"Complete the Python code that follows this instruction: '{docstring}'. Your response must start with code block '```python'."

    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        temperature=0,
        max_tokens=1024,
        top_p=1,
        messages=[
            {
                "role": "system",
                "content": init_prompt,
            },
            {
                "role": "user",
                "content": prompt,
            },
        ],
    )

    codeblock = response.choices[0].message.content
    code = next(filter(None, codeblock.split("```python"))).rsplit("```", 1)[0]
    code = code.strip()

    return code
```

诀窍就是提前设置好引导词：

```
init_prompt = "You are a Python expert who can implement the given function."
    definition = f"def {func}"
    prompt = f"Read this incomplete Python code:\n```python\n{definition}\n```"
    prompt += "\n"
    prompt += f"Complete the Python code that follows this instruction: '{docstring}'. Your response must start with code block '```python'."
```

这里我们提前设置两个参数func和docstring，也就是函数名和功能描述，要求ChatGPT严格按照参数的输入来返回代码，现在运行函数：

```
if __name__ == '__main__':

    print(generate_code("test","Sum two numbers"))
```

程序返回：

```
➜  chatgpt_write_code /opt/homebrew/bin/python3.10 "/Users/gaolei/chatgpt_write_code/chatgpt_write_code.py"
def test(a, b):
    return a + b
```

如此一来，ChatGPT就不会返回废话，而是直接交给我们可以运行的代码。

### [](about:blank#%E8%A3%85%E9%A5%B0%E5%99%A8%E8%B0%83%E7%94%A8ChatGPT "装饰器调用ChatGPT")装饰器调用ChatGPT

事实上，函数调用环节也可以省略，我们可以使用Python装饰器的闭包原理，直接将所定义函数的参数和描述传递给ChatGPT，随后再直接运行被装饰的函数，提高效率：

```
import inspect
from functools import wraps

def chatgpt_code(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        signature = f'{func.__name__}({", ".join(inspect.signature(func).parameters)}):'
        docstring = func.__doc__.strip()
        code = generate_code(signature, docstring)
        print(f"generated code:\n```python\n{code}\n```")
        exec(code)
        return locals()[func.__name__](*args, **kwargs)

    return wrapper
```

将方法定义好之后，使用基于ChatGPT的装饰器:

```
if __name__ == '__main__':

    @chatgpt_code
    def sum_two(num1,num2):
        """
        Sum two numbers.
        """

    print(sum_two(1,2))
```

程序返回：

```
➜  chatgpt_write_code /opt/homebrew/bin/python3.10 "/Users/gaolei/chatgpt_write_code/chatgpt_write_code.py"
sum_two(num1, num2):
generated code:

def sum_two(num1, num2):
    """
    Sum two numbers.
    """
    return num1 + num2
```

直接将业务逻辑和运行结果全部返回。

那么现在，回到开篇的关于布隆过滤器的问题：

```
if __name__ == '__main__':

    @chatgpt_code
    def bloom(target:str,storage:list):
        """
        Use a Bloom filter to check if the target is in storage , Just use this func , no more class
        """

    print(bloom("你好",["你好","Helloworld"]))
```

程序返回：

```
➜  chatgpt_write_code /opt/homebrew/bin/python3.10 "/Users/gaolei//chatgpt_write_code/chatgpt_write_code.py"
generated code:

def bloom(target, storage):
    # Initialize the Bloom filter with all zeros
    bloom_filter = [0] * len(storage)

    # Hash the target and set the corresponding bit in the Bloom filter to 1
    for i in range(len(storage)):
        if target in storage[i]:
            bloom_filter[i] = 1

    # Check if all the bits corresponding to the target are set to 1 in the Bloom filter
    for i in range(len(storage)):
        if target in storage[i] and bloom_filter[i] == 0:
            return False

    return True

True
➜  chatgpt_write_code
```

丝滑流畅，和业务衔接得天衣无缝，拉链般重合，不需要挑挑拣拣，也不必复制粘贴。

### [](about:blank#%E7%BB%93%E8%AF%AD "结语")结语

毫无疑问，ChatGPT确然是神兵利器，吹毛可断，无坚不摧。但工具虽好，也需要看在谁的手里，所谓工具无高下，功力有高深，类比的话，如果倚天剑握在三岁孩童手中，不仅毫无增益，还可能伤其自身，但是握在峨眉掌门灭绝师太手里，那就可以横扫千军如卷席了，那才能体现大宗匠的手段

  

