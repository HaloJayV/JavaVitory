* SELECT DISTINCT price FROM product;    // 去重查询
* truncate与delete的异同：
  * truncate是DDL，操作不会进行存储不能进行事务回滚，而delete是DML，会被回滚
  * truncate是删除整个表
  * truncate事务日志少，速度较快，delete则速度慢，但相对安全许多
  * delete数据不会对数据库的查询效率有所改变，而truncate会使数据库容量被重置，DML速度提高

* 修改表名：rename table [xxx] [old] to [new]; / alter table [xxx] rename [new]
* 模糊查询：
  * like "%x%" // 字符匹配     like "x___"  //前缀匹配     in(2,4,6,8) // 包含查询
  * between 1 and 5   // 范围查询

* 排序：  order by xxx asc/desc
* 聚合函数：sum, avg, max, min 

* 外键：

  * 主表：从表 = 一对多
  * alter table 从表 add constraint 外键名 foreign key(从表主键) references 主表名(主表主键);

  * 主表不能删除从表中已经使用的数据，应该先删除从表，再删除主表数据
  * 可以在创建表时添加外键，如 foreign key(iforeign_goodsid) references goods(goodsid);

* 多表查询：

  * 交叉查询(了解): select * from A  表名，B   表名  where  A.aid = B.a_id
  * 内连接查询:
    * 隐式内连接查询：select distinct cname from category c, product p where c.cateid=p.category_id
    * 显示内连接查询：select * from A 表名 inner join B 表名 on category.cateid=p.category_id
  * 外连接查询
  * 子查询：一条select语句的结果作为另一条select语句的一部分，如
    * select * from product where category_id=(select cateid from category where cname='服装')



##  Spider爬虫

* UserAgentTool.py      存放可用的UA

```
import random
# 存放可用的UA
user_agent_list = [
    {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.163 Safari/537.36"},
    {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:75.0) Gecko/20100101 Firefox/75.0"},
    {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36"},
    {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko"},
    {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.135 Safari/537.36 Edge/12.10240"},
    {
        "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.95 Safari/537.36 OPR/26.0.1656.60"},
    {"User-Agent": "Opera/8.0 (Windows NT 5.1; U; en)"},
    {"User-Agent": "Mozilla/5.0 (Windows NT 5.1; U; en; rv:1.8.1) Gecko/20061208 Firefox/2.0.0 Opera 9.50"},
    {"User-Agent": "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; en) Opera 9.50"},
    {"User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64; rv:34.0) Gecko/20100101 Firefox/34.0"},
    {
        "User-Agent": "Mozilla/5.0 (X11; U; Linux x86_64; zh-CN; rv:1.9.2.10) Gecko/20100922 Ubuntu/10.10 (maverick) Firefox/3.6.10"},
    {
        "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/534.57.2 (KHTML, like Gecko) Version/5.1.7 Safari/534.57.2"},
    {
        "User-Agent": "Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.71 Safari/537.36"},
    {
        "user-agent": "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36 QIHU 360SE/12.1.2812.0"},
    {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/64.0.3282.140 Safari/537.36 Edge/18.17763"}
]
def get_headers():
    index = random.randint(0, len(user_agent_list) - 1)
    return user_agent_list[index]
if __name__ == '__main__':
    headers = get_headers()
    print("随机获得UA:", headers)
```

* ProxyTool.py    动态获取代理

```
# 动态获取代理
import json
import random

def read_proxy_tool_file():
    return json.load(open("./ipfile/ProxyTool.json", "r"))

def get_proxy():
    proxy_value_list = read_proxy_tool_file()

    index = random.randint(0, len(proxy_value_list) - 1)
    return proxy_value_list[index]

if __name__ == "__main__":
    proxy = get_proxy()
    print("动态获取ip为：", proxy)
```

* SpecialSpider.py    从指定网址中爬取数据并以json格式保存到文件中

```
import requests
import UserAgentTool
import ProxyTool
import json
import time
import random

class SpecialSpider(object):
    def __init__(self):
        self.special_url = "https://api.eol.cn/gkcx/api/?access_token=&keyword=&level1=&level2=&page={}&signsafe=&size=20&uri=apidata/api/gk/special/lists"

    # 拼接网址url列表
    def get_special_url_list(self):
        url_list = []
        for page in range(1, 73):
            temp_url = self.special_url.format(page)
            url_list.append(temp_url)
        return url_list

    def parse_special_url(self, temp_url):
        special_response = requests.get(url=temp_url,
                     headers=UserAgentTool.get_headers(),
                     proxies=ProxyTool.get_proxy())
        html_content = special_response.content.decode("utf-8")
        special_results = json.loads(html_content)
        return special_results

    def run(self):
        special_datas = []
        special_url_list = self.get_special_url_list()
        for special_url in special_url_list:
            special_page_infos = self.parse_special_url(special_url)
            print(special_page_infos)
            # 获取相应数据并保存
            item = special_page_infos["data"]["item"]
            special_datas.extend(item)
            # 保存
            json.dump(special_datas,
                      open("./specialfile/special.json", "w", encoding="utf-8"),
                      ensure_ascii=False,
                      indent=2)
            wait_time = random.randint(0, 10)
            time.sleep(wait_time)
        print("数据已保存成功")

def main():
    spider = SpecialSpider()
    spider.run()

if __name__ == '__main__':
    main()
```

* specialtypedata.py    从json文件中获取数据并保存到mysql中

```
# _*_ coding:utf-8 _*_
__author__ = 'xiejianyu'
__date__ = '2020-6-30 11:26'

import json
import sqlhelpertool

def read_special_data_file():
    special_results = json.load(open("./specialfile/special.json", "r", encoding="utf-8"))
    return special_results

def save_types_to_table_specialtype(types):
    # 获得链接对象
    sql_helper = sqlhelpertool.MySQLHelper(host="localhost",
                                           port=3306,
                                           user="root",
                                           password="root",
                                           db="db_special")
    for typename in types:
        sql = "insert into tbl_special_type(typename) values(%s);"
        params = [typename]
        sql_helper.insert(sql, params)
        print("正在存入到专业类别表，名称为：%s",typename)
    print("所有专业列表已经存入到数据库中")

def main():
    # 1。 获取json文件中所有数据内容
    # 2. 提取专业类别名称
    # 3. 去掉重复名称
    # 4。
    special_datas = read_special_data_file()
   # print(special_data)

    types_lists = []
    for special_element in special_datas:
        typename = special_element["level3_name"]
        # print(typename)
        types_lists.append(typename)

    # 去掉重复值
    types_lists = list(set(types_lists))
    save_types_to_table_specialtype(types_lists)

if __name__ == '__main__':
    main()
```

* sqlhelpertool.py    保存到mysql工具类

```
# coding:utf-8
import pymysql

class MySQLHelper(object):
    def __init__(self, host, port, user, password, db, charset="utf8"):
        # 初始化数据
        self.host = host
        self.user = user
        self.password = password
        self.port = port
        self.db = db
        self.charset = charset

    def open(self):
        self.conn = pymysql.connect(host=self.host,
                                    port=self.port,
                                    user=self.user,
                                    password=self.password,
                                    db=self.db,
                                    charset=self.charset)
        self.data_cursor = self.conn.cursor()

    def insert(self, sql, params=[]):
        """插入数据"""
        try:
            self.open()
            # 执行语句
            self.data_cursor.execute(sql, params)
            # 提交事务
            self.conn.commit()
            # 关闭
            self.close()
            print("本次数据操作完成!")
        except Exception as msg:
            print("数据操作时,提示错误信息:", msg)

    def close(self):
        self.data_cursor.close()
        self.conn.close()
```

