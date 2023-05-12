---
title: chatGPT with confluence
author: Ray
top: true
cover: false
date: 2023-05-12 15:44:05
categories: 技术
tags:
  - python
  - chatGpt
  - 算法
---

### Target:

由于工作时Confluence中的文件太多，也比较杂乱，难以阅读，所以希望基于chatGPT能够帮助我快速从文件中获取我想要的知识

<!-- more -->

以下是一个demo的代码，参考了GPT官网的做法


```python
##爬虫
import requests
import re
import urllib.request
from bs4 import BeautifulSoup
from collections import deque
from html.parser import HTMLParser
from urllib.parse import urlparse
import os
import html2text
from atlassian import Confluence
import tiktoken
import pandas as pd
import openai
from openai.embeddings_utils import distances_from_embeddings
import numpy as np
from openai.embeddings_utils import distances_from_embeddings, cosine_similarity
import time


# public 的账号 和confluence空间配置
domain = "confluence.xxxxx.com"
full_url = "https://confluence.xxxxx.com/"
login_url = "https://confluence.xxxxx.com/login.action?os_destination=%2Fdologin.action"
page_url = "https://confluence.xxxxx.com/display/41JTSP/"
user_name = "xxxx"
password = "xxxx"
# 定义要爬取的空间key，这是目前我们的空间
space_key = "xxxx"
##数据保存地址，请自定义
filePath = ""
processPath = ""

##分隔符和替换符，主要用于文件名生成和标题还原
splitFlag = "$"
replaceFlag = "_"




##设置openai环境
openai.organization = ""
openai.api_key = ""
```


```python
def crawler(base_url,username,password,space_key,totalSpace = False):
    confluence = Confluence(url=base_url, username=username, password=password)
    ##待实现，爬取所有的space
    ##获取对应空间
    space = confluence.get_space(space_key, expand='description.plain,homepage')
    ##获取space页面id
    page_id = space["homepage"]["id"]
    
        # Create a directory to store the text files
    if not os.path.exists(filePath):
            os.mkdir(filePath)

    # Create a directory to store the csv files
    if not os.path.exists(processPath):
            os.mkdir(processPath)
    
    ##子页面
    child = confluence.get_page_child_by_type(page_id, type='page', start=None, limit=None, expand=None)
    
    ##初始化队列
    queue = deque()
    for i in child:
        queue.append(i)
    
    while queue:
        # Get the next URL from the queue
        childPage = queue.pop()
        ##拿到页面id
        html = confluence.get_page_by_id(childPage["id"], expand="body.storage")
        # 调用方法，将html转为纯文本
        content = html["body"]["storage"]["value"]
        content_text = html2text.html2text(content)
        
        ##文本不为空写入
        if content_text.lstrip() != "":
            title = str(html["title"]).replace("/",replaceFlag)
    #         if not os.path.exists("/Users/lei.zhou/text/"+html["title"]):
    #             os.mkdir("/Users/lei.zhou/text/")
            with open(filePath+ childPage["id"]+splitFlag+title+ ".txt", "w") as f:
                f.write(content_text)

        ##加入子节点‘
        for i in confluence.get_page_child_by_type(childPage["id"], type='page', start=None, limit=None, expand=None):
            queue.append(i)

```


```python
max_tokens = 500

def remove_newlines(serie):
    serie = serie.str.replace('\n', ' ')
    serie = serie.str.replace('\\n', ' ')
    serie = serie.str.replace('  ', ' ')
    serie = serie.str.replace('  ', ' ')
    return serie
def create_context(
    question, df, max_len=1800, size="ada"
):
    """
    寻找最相似的文本段
    """
    # Get the embeddings for the question
    q_embeddings = openai.Embedding.create(input=question, engine='text-embedding-ada-002')['data'][0]['embedding']
    # 使用余弦算法计算最相似的文本
    df['distances'] = distances_from_embeddings(q_embeddings, df['embeddings'].values, distance_metric='cosine')


    returns = []
    cur_len = 0

    # 不断添加文本到上限
    for i, row in df.sort_values('distances', ascending=True).iterrows():
        
        # 文本创建
        cur_len += row['n_tokens'] + 4
        
        # 超出上限退出
        if cur_len > max_len:
            break
        
        # 增加文本
        returns.append(row["text"])

    # 返回
    return "\n\n###\n\n".join(returns)


# token分割
def split_into_many(text, max_tokens = max_tokens):

    # 定义分割符号，可以允许自定义
    sentences = re.split('[.。！？!?]',text)

    # 获取每段的token
    n_tokens = [len(tokenizer.encode(" " + sentence)) for sentence in sentences]
    
    chunks = []
    tokens_so_far = 0
    chunk = []

    # 遍历
    for sentence, token in zip(sentences, n_tokens):

        # 如果到目前为止的标记数量加上当前句子中的标记数量大于,大于最大标记数，则将该块添加到块的列表中，并重置到目前为止的块和标记数
        if tokens_so_far + token > max_tokens:
            chunks.append(". ".join(chunk) + ".")
            chunk = []
            tokens_so_far = 0

    
        if token > max_tokens:
            continue

        # 添加
        chunk.append(sentence)
        tokens_so_far += token + 1

    return chunks


##数据，模型，问题，长度，
def answer_question(
    df,
    model="text-davinci-003",
    question="你有什么问题",
    max_len=1800,
    size="ada",
    debug=False,
    max_tokens=1800,
    stop_sequence=None,
    use_GPT=False
):
    """
    回答问题
    """
    context = create_context(
        question,
        df,
        max_len=max_len,
        size=size,
    )
    
    # If debug, print the raw model response
    if debug:
        print("Context:\n" + context)
        print("\n\n")
        print(f"Answer the question based on the context below, and if the question can't be answered based on the context, say \"I don't know\"\n\nContext: {context}\n\n---\n\nQuestion: {question}\nAnswer:")
    if use_GPT:
        completion = openai.ChatCompletion.create(model="gpt-3.5-turbo",messages=[
    {"role": "user", "content": f"Answer the question based on the context below, and if the question can't be answered based on the context, say \"I don't know\"\n\nContext: {context}\n\n---\n\nQuestion: {question}\nAnswer:"}])
        return completion.to_dict()["choices"][0]["message"]["content"]
    try:
        # Create a completions using the question and context
        response = openai.Completion.create(
            prompt=f"Answer the question based on the context below, and if the question can't be answered based on the context, say \"I don't know\"\n\nContext: {context}\n\n---\n\nQuestion: {question}\nAnswer:",
            temperature=0,
            max_tokens=max_tokens,
            top_p=1,
            frequency_penalty=0,
            presence_penalty=0,
            stop=stop_sequence,
            model=model,
        )
        return response["choices"][0]["text"].strip()
    except Exception as e:
        print(e)
        return ""

    
```


```python
crawler(base_url,username,password,space_key)
```

```python
#原始文本
texts=[]

# 遍历
for file in os.listdir(filePath):
    # 文件读取
    with open(filePath+file, "r") as f:
        titles = file.split(splitFlag)
        if len(titles) <= 1:
            continue
        title = titles[1]
        text = f.read()
        # 标题还原，把_替换为空格插入
        texts.append((title.replace(replaceFlag," "), text))
        
# pd创建
df = pd.DataFrame(texts, columns = ['fname', 'text'])

# 按行分段
df['text'] = df.fname + ". " + remove_newlines(df.text)
df.to_csv('processed/scraped.csv')
df.head()
```

</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>fname</th>
      <th>text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-12-15【VWASP】【IDP】Scan qrcode login HU to ...</td>
      <td>2022-12-15【VWASP】【IDP】Scan qrcode login HU to ...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-12-19   【VWASP】【StartupBroadcasting】Conte...</td>
      <td>2022-12-19   【VWASP】【StartupBroadcasting】Conte...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>bak.B1.1 Payment Center Integration Test Repor...</td>
      <td>bak.B1.1 Payment Center Integration Test Repor...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-03-15  [VWASP] 【ULH】DP token exchange wit...</td>
      <td>2022-03-15  [VWASP] 【ULH】DP token exchange wit...</td>
    </tr>
    <tr>
      <th>4</th>
      <td>OnlineRadio and OlineMusic B0.2 testcase revie...</td>
      <td>OnlineRadio and OlineMusic B0.2 testcase revie...</td>
    </tr>
  </tbody>
</table>




```python
tokenizer = tiktoken.get_encoding("cl100k_base")

df = pd.read_csv('processed/scraped.csv', index_col=0)
df.columns = ['title', 'text']

df['n_tokens'] = df.text.apply(lambda x: len(tokenizer.encode(x)))

df
```



</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>title</th>
      <th>text</th>
      <th>n_tokens</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-12-15【VWASP】【IDP】Scan qrcode login HU to ...</td>
      <td>2022-12-15【VWASP】【IDP】Scan qrcode login HU to ...</td>
      <td>1423</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-12-19   【VWASP】【StartupBroadcasting】Conte...</td>
      <td>2022-12-19   【VWASP】【StartupBroadcasting】Conte...</td>
      <td>1355</td>
    </tr>
    <tr>
      <th>2</th>
      <td>bak.B1.1 Payment Center Integration Test Repor...</td>
      <td>bak.B1.1 Payment Center Integration Test Repor...</td>
      <td>1106</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-03-15  [VWASP] 【ULH】DP token exchange wit...</td>
      <td>2022-03-15  [VWASP] 【ULH】DP token exchange wit...</td>
      <td>1429</td>
    </tr>
    <tr>
      <th>4</th>
      <td>OnlineRadio and OlineMusic B0.2 testcase revie...</td>
      <td>OnlineRadio and OlineMusic B0.2 testcase revie...</td>
      <td>2736</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>412</th>
      <td>B1 OnePortal Qulification Test Report.txt</td>
      <td>B1 OnePortal Qulification Test Report.txt.   #...</td>
      <td>966</td>
    </tr>
    <tr>
      <th>413</th>
      <td>Detailed Solution Architecture.txt</td>
      <td>Detailed Solution Architecture.txt. 250</td>
      <td>8</td>
    </tr>
    <tr>
      <th>414</th>
      <td>B1.3 Release.txt</td>
      <td>B1.3 Release.txt. true</td>
      <td>9</td>
    </tr>
    <tr>
      <th>415</th>
      <td>04  B1验收Charging&amp;RBC.txt</td>
      <td>04  B1验收Charging&amp;RBC.txt.  L1| L2| L3| | | L4|...</td>
      <td>1492</td>
    </tr>
    <tr>
      <th>416</th>
      <td>ULH - Integration Outline and API-Specificatio...</td>
      <td>ULH - Integration Outline and API-Specificatio...</td>
      <td>38</td>
    </tr>
  </tbody>
</table>
<p>417 rows × 3 columns</p>

</div>




```python
# Tokenize the text and save the number of tokens to a new column
df['n_tokens'] = df.text.apply(lambda x: len(tokenizer.encode(x)))

# Visualize the distribution of the number of tokens per row using a histogram
df.n_tokens.hist()
```




    <AxesSubplot:>




![png](chatGPT-with-confluence/output_7_1-3877587.png)
​    



```python
shortened = []

# 循环减少文本
for row in df.iterrows():
    print(row)

    if row[1]['text'] is None:
        continue

    if row[1]['n_tokens'] > max_tokens:
        shortened += split_into_many(row[1]['text'])
    else:
        shortened.append( row[1]['text'] )
df = pd.DataFrame(shortened, columns = ['text'])
df['n_tokens'] = df.text.apply(lambda x: len(tokenizer.encode(x)))
df.n_tokens.hist()
```





![png](chatGPT-with-confluence/output_8_1-3877587.png)
​    


```python
##由于官方的限制，1分钟最多发起60个请求，所以为了防止报错此处主动休眠
##由于数据量过大，如果无法运行，可以在上面一栏    截取部分数据df = df[0:x]  x为截取长度
def cal(x,waittime = 0.6):
    res = openai.Embedding.create(input=x, engine='text-embedding-ada-002')['data'][0]['embedding']
    time.sleep(waittime)
    return res
df['embeddings'] = df.text.apply(lambda x: cal(x))

df.to_csv('processed/embeddings.csv')
df.head()
```



</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>text</th>
      <th>n_tokens</th>
      <th>embeddings</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-12-15【VWASP】【IDP】Scan qrcode login HU to ...</td>
      <td>472</td>
      <td>[-0.0035752991680055857, 0.015155627392232418,...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>| 11 incomplete | /  6 | 用例设计是否包含充分的正面、负面异常测试...</td>
      <td>343</td>
      <td>[0.012331507168710232, 0.002946529071778059, -...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-12-19   【VWASP】【StartupBroadcasting】Conte...</td>
      <td>454</td>
      <td>[0.004054947756230831, -0.0028905682265758514,...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>| 203 complete OK |  |   **专属部分** |  | 检查人：Zh...</td>
      <td>492</td>
      <td>[0.019448528066277504, 0.00747803645208478, 0....</td>
    </tr>
    <tr>
      <th>4</th>
      <td>bak. B1. 1 Payment Center Integration Test Rep...</td>
      <td>481</td>
      <td>[0.0010055731981992722, -0.013514618389308453,...</td>
    </tr>
  </tbody>
</table>

</div>




```python
##读取token数据
df=pd.read_csv('processed/embeddings.csv', index_col=0)
df['embeddings'] = df['embeddings'].apply(eval).apply(np.array)
df.head()
```



</style>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>text</th>
      <th>n_tokens</th>
      <th>embeddings</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-12-15【VWASP】【IDP】Scan qrcode login HU to ...</td>
      <td>472</td>
      <td>[-0.0035752991680055857, 0.015155627392232418,...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>| 11 incomplete | /  6 | 用例设计是否包含充分的正面、负面异常测试...</td>
      <td>343</td>
      <td>[0.012331507168710232, 0.002946529071778059, -...</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-12-19   【VWASP】【StartupBroadcasting】Conte...</td>
      <td>454</td>
      <td>[0.004054947756230831, -0.0028905682265758514,...</td>
    </tr>
    <tr>
      <th>3</th>
      <td>| 203 complete OK |  |   **专属部分** |  | 检查人：Zh...</td>
      <td>492</td>
      <td>[0.019448528066277504, 0.00747803645208478, 0....</td>
    </tr>
    <tr>
      <th>4</th>
      <td>bak. B1. 1 Payment Center Integration Test Rep...</td>
      <td>481</td>
      <td>[0.0010055731981992722, -0.013514618389308453,...</td>
    </tr>
  </tbody>
</table>
</div>


```python
##输入最大token，返回长度进行提问
answer_question(df, question="测试用例需要满足那些要求?", debug=False,use_GPT=True,max_len=1800,max_tokens = 1800)
```


    '测试用例需要满足许多要求，包括但不限于：前提条件、输入数据和期待结果清晰、明确；包含充分的正面、负面异常测试用例；是否从用户层面来设计用户使用场景和使用流程的测试用例；是否简洁，复用性强等。具体要求见上文中的各项检查项。'


```python
answer_question(df, question="一份DD文档或是AD文档需要满足那些要求?,请用中文回答", debug=False,use_GPT=True,max_len=1800,max_tokens = 1800)
```


    'DD文档或AD文档需要满足以下要求：\n1. 它应该清晰明了，包括应用的功能、性能、界面、安全等方面。\n2. 它应该易于理解，包括适当的说明、流程图、状态图等。\n3. 它应该满足可复用性、易维护性等软件工程的基本原则。\n4. 它应该包括可靠性需求，描述系统的可用性、健壮性等需求。\n5. 它应该考虑约束和假设，包括客户和MA的约束和假设。\n6. 它应该包括详细设计，以阐明更改的原因和过程，并进行一致性评估。\n7. 它应该考虑软件单元的互操作性、交互、关键性、技术复杂性、风险和可测试性等方面。 \n8. 它应该包括接口定义，以使接口清楚明了。\n9. 历史记录应该得到正确维护。'







