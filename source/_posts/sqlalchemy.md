---
title: sqlalchemy
date: 2021-07-09 15:56:58
tags: sql
categories: 数据库
keywords: sqlalchemy
top: 
image: images/sql/sql_1.webp
---
SQLAlchemy 是一个功能强大的python ORM工具包。
<!--more-->

## 1.0 模型定义

model等同于数据库中的表。  
```python   
from __future__ import unicode_literals, absolute_import
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String, DateTime
ModelBase = declarative_base()
class User(ModelBase):
    __tablename__ = "auth_info"

    id = Column(Integer, primary_key=True)
    date_joined = Column(DateTime)
    username = Column(String(length=30))
    password = Column(String(length=128))
```

## 2.0 增删改查  

### 在用sqlalchemy时，通常先定义个类，方便之后调用。
```python
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine


class SqlalchemyTool(object):
    def __init__(self,db):
        self.db = db
        self.uri = "mysql+pymysql://{user}:{pwd}@{host}:{port}/{db}?charset=utf8".format(user=config.Mysql.user,host=config.Mysql.server,port=config.Mysql.port,db=self.db,pwd=config.Mysql.pwd)
        __engine = create_engine(self.uri, encoding='utf-8', echo=False)
        Session_class = sessionmaker(bind=__engine)
        self.session = Session_class()


    def __enter__(self):
        return self.session

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.session:
            self.session.close()
```

### 增（add）   
```python
with SqlalchemyTool(这里写数据库) as session:
    sql = UserTable(username='username', password='password')
    session.add(sql)
    session.commit()
```
**session**可以看成一个管理数据库持久连接的对象。  
session.add函数会把model加入当前的持久空间，直到session.commit提交。  

### 删（delete）
```python
with SqlalchemyTool(这里写数据库) as session:
    record = session.query(User).filter(User.username == 'username')
    session.delete(record)
    session.commit()
    # 或者 session.query(User).filter_by(username='username').delete()
```
**filter**相当于where,过滤条件。

### 改（update）
```python
with SqlalchemyTool(这里写数据库) as session:
    session.query(User).filter(User.username == 'username').update(
        {"password": 'password1'}
    )
    session.commit()
```
**update** 进行修改相关数据库内容，在对model的属性进行修改时，session会得到修改对应的内容，下次commit即会提交sql。

### 查（query）
```python
with SqlalchemyTool(这里写数据库) as session:
    session.query(User).filter((User.username == 'username')
    # 和 select * from User  等同
    # session.query(User.username).filter((User.username == 'username')
```

[详情查看官方文档。](https://docs.sqlalchemy.org/en/14/)