---
title: Python中数据库连接的封装
date: 2023-06-21 10:57:34
tags:
    - Python
categories: 
    - Python
description: 自己写的一些数据库连接封装
---

## MySQL connect
```python
class DatabaseHandle(object):
    """MySQL操作类"""
    def __init__(self, db_conf: dict, **kwargs):
        """
        初始化数据库连接
        :param db_conf: 数据库连接
        :param kwargs: 不同数据库参数
        """
        db_conf.update(kwargs) if kwargs else db_conf

        self.conn = pymysql.connect(
            host=db_conf.get("host"),
            user=db_conf.get("user"),
            password=db_conf.get("password"),
            port=db_conf.get("port"),
            db=db_conf.get("db"),
            charset=db_conf.get("charset"),
            cursorclass=pymysql.cursors.DictCursor
        )
        self.cursor = self.conn.cursor()

    def fetch_data(self, sql: str, is_single: bool = False) -> tuple:
        """
        查询数据
        :param sql: SQL语句
        :param is_single: 查询单条或多条
        :return: 返回查询结果
        """
        try:
            self.cursor.execute(sql)
            data = self.cursor.fetchone() if is_single else self.cursor.fetchall()

            return data

        except:
            raise Exception("Error: unable to fetch data")

    def delete_data(self, sql: str):
        """
        删除数据
        :param sql: SQL语句
        :return:
        """
        try:
            self.cursor.execute(sql)
            self.conn.commit()

        except:
            self.conn.rollback()

    def update_data(self, sql: str):
        """
        更新数据
        :param sql: SQL语句
        :return:
        """
        try:
            self.cursor.execute(sql)
            self.conn.commit()

        except:
            self.conn.rollback()

    def close_db(self):
        """
        数据库连接关闭
        :return:
        """
        self.conn.close()
```

## MongoDB connect
```python
class MongoDBClient(object):
    """定义MongoDB操作类"""
    def __init__(self, mongo_conf):
        """
        初始化参数
        :param mongo_conf: mongo配置
        """
        self.client = pymongo.MongoClient(
            "mongodb://{0}:{1}@{2}:{3}".format(
                mongo_conf.get("username"),
                parse.quote(mongo_conf.get("password")),
                mongo_conf.get("host"),
                mongo_conf.get("port")
            )
        )

        self.db = self.client[mongo_conf.get("db_name")]

    def insert_one(self, table_name, insert_data, data_field=None):
        """
        插入单条数据
        :param table_name: 数据表名
        :param insert_data: 插入的数据
        :param data_field: 数据筛选
        :return:
        """
        try:
            self.db[table_name].insert_one(insert_data, data_field)

        except:
            raise Exception("插入错误")

    def insert_many(self, table_name, insert_data):
        """
        插入多条数据
        :param table_name: 数据表名
        :param insert_data: 插入数据
        :return:
        """
        try:
            self.db[table_name].insert_many(insert_data)

        except:
            raise Exception("插入错误")

    def find_one(self, table_name: str, data_field: dict = None, target_field: dict = None) -> dict:
        """
        查找单条数据有
        :param table_name: 数据表名
        :param data_field: 查找数据字段
        :param target_field: 指定返回的字段
        :return:
        """
        try:
            result_data = self.db[table_name].find_one(data_field, target_field)
            return result_data

        except:
            raise Exception("查找错误")

    def find_many(self, table_name: str, data_field: dict = None, target_field: dict = None) -> list:
        """
        查找多条数据
        :param table_name: 数据表名
        :param data_field: 查找数据字段
        :param target_field: 指定返回的字段
        :return:
        """
        try:
            result_data = self.db[table_name].find(data_field, target_field)
            return result_data

        except:
            raise Exception("查找错误")

    def update_one(self, table_name, data_condition, data_set):
        """
        更新单条数据
        :param table_name: 数据表名
        :param data_condition: 原数据
        :param data_set: 新数据
        :return:
        """
        try:
            self.db[table_name].update_one(data_condition, {"$set": data_set})

        except:
            raise Exception("更新错误")

    def update_many(self, table_name, data_condition, data_set):
        """
        更新多条数据
        :param table_name: 数据表名
        :param data_condition: 原数据
        :param data_set: 新数据
        :return:
        """
        try:
            self.db[table_name].update_many(data_condition, {"$set": data_set})

        except:
            raise Exception("更新错误")

    def replace_one(self, table_name, data_condition, data_set):
        """
        替换单条数据
        :param table_name: 数据表名
        :param data_condition: 原数据
        :param data_set: 新数据
        :return:
        """
        try:
            self.db[table_name].replace_one(data_condition, data_set)

        except:
            raise Exception("替换错误")

    def delete_many(self, table_name, delete_data):
        """
        删除多条数据
        :param table_name: 数据表名
        :param delete_data: 删除的数据
        :return:
        """
        try:
            self.db[table_name].delete_many(delete_data)

        except:
            raise Exception("删除错误")

    def delete_one(self, table_name, delete_data):
        """
        删除单条数据
        :param table_name: 数据表名
        :param delete_data: 删除的数据
        :return:
        """
        try:
            self.db[table_name].delete_one(delete_data)

        except:
            raise Exception("删除错误")

    def sum_data(self, table_name: str, match_obj: dict, group_obj: dict) -> list:
        """
        查询数据计算
        :param table_name: 数据表名
        :param match_obj: 过滤条件
        :param group_obj: 计算条件
        :return:
        """
        try:
            data_obj = self.db[table_name].aggregate([
                {"$match": match_obj},
                {"$group": group_obj}
            ])
            return list(data_obj)

        except:
            raise Exception("查询失败")

    def aggregate_query(self, table_name: str, pipeline: list) -> list:
        """
        查询数据计算
        :param table_name: 数据表名
        :param pipeline: 聚合过滤条件
        :return:
        """
        try:
            data_obj = self.db[table_name].aggregate(pipeline)
            return list(data_obj)

        except:
            raise Exception("查询失败")

    def close_db(self):
        """
        关闭数据库
        :return:
        """
        self.client.close()
```

## Redis connect
```python
class RedisClient(object):
    """定义Redis操作类"""
    def __init__(self, redis_conf: dict):
        """
        初始化redis
        :param redis_conf: redis配置
        """
        self.client = redis.ConnectionPool(
            host=redis_conf.get("host"),
            port=redis_conf.get("port"),
            db=redis_conf.get("db"),
            password=redis_conf.get("password")
        )
        self.r = redis.StrictRedis(connection_pool=self.client)

    def redis_pop(self, key):
        """
        移除并返回第一个元素
        :param key:
        :return:
        """
        return self.r.lpop(key)

    def operate_lock(self):
        self.r.set(name="SERVICE_OPERATE_PLAN_20200225", value=1)

    def operate_unlock(self):
        self.r.delete("SERVICE_OPERATE_PLAN_20200225", )

    def get_operate_status(self):
        status = self.r.exists("SERVICE_OPERATE_PLAN_20200225", )
        return status

    def redis_close(self):
        self.r.close()
```
