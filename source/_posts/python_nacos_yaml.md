---
title: python实现拉取nacos实时更新的yaml文件
date: 2021-07-09 15:56:58
tags: python
categories: python
keywords: python, nacos, yaml
top: 
image: https://raw.githubusercontent.com/AshaoSunshine/ashaoimg/master/landscape/landscape_2.jpg
---
在naocs上提交yaml文件，客户端将yaml文件自动转换成python格式。
<!--more-->

## Step 1.调用nacos相关配置和参数

```python
import nacos
import yaml

# 自定义相关参数
SERVER_ADDPESSER = '127.0.0.1:8848'
NAMESPACE = 'public'
USERNAME = 'nacos'
PASSWORD = 'nacos'


client = nacos.NacosClient(SERVER_ADDRESSES, namespace=NAMESPACE, username=USERNAME, password=PASSWORD)
```  

## Step 2.创建代码实例

```python

def nacos_test():
    yaml_dict = yaml.safe_load()
    class_name = list(yaml_dict.keys())

    with open('test.py', 'w') as b:
        for cn in class_name:
            class_dict = yaml_dict[cn]
            b.write(str.format('{} = {}\n', cn, class_dict))
```

## Step 3.主函数调用代码实例

```python
if __name__ == "__main__"

    conf = client.get_config('test', 'DEFAULT_GROUP')
    nacos_test(conf)
    client.add_config_watcher('test', 'DEFAULT_GROUP',call_back)

    while True:
        time.sleep(1)
```

## Step 4.代码总和
```python
import nacos
import yaml
import time


SERVER_ADDPESSER = '127.0.0.1:8848'
NAMESPACE = 'public'
USERNAME = 'nacos'
PASSWORD = 'nacos'


def call_back(args):
    
    write_conf(args['content'])
    Share.count += 1
    Share.content = args["content"]


def nacos_test():
    yaml_dict = yaml.safe_load()
    class_name = list(yaml_dict.keys())

    with open('test.py', 'w') as b:
        for cn in class_name:
            class_dict = yaml_dict[cn]
            b.write(str.format('{} = {}\n', cn, class_dict))


client = nacos.NacosClient(SERVER_ADDRESSES, namespace=NAMESPACE, username=USERNAME, password=PASSWORD)
if __name__ == "__main__"

    # test是配置列表中的Data Id 
    # DEFAULT_GROUP 是Group
    conf = client.get_config('test', 'DEFAULT_GROUP')
    nacos_test(conf)
    client.add_config_watcher('test', 'DEFAULT_GROUP',call_back)

    while True:
        time.sleep(1)