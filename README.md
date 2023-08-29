# 爬虫

# 1 系统设计文档 
   该系统是从读书网（https://www.dushu.com/book/1188_1.html）获取书本信息，包括书本的名字和封面图片链接。 
## 1.1 功能模块设计 
   本文所设计的网络爬虫程序是基于 PyCharm 的编程平台建设，整个系统主要由如下四个模块组成：Scrapy爬虫模块、数据库MySql模块、CrawlSpider模块。系统结构图如图1 所示。
![图 1 系统结构图](https://github.com/nanjingzhuyuxuan/PaChong/blob/main/picture/%E5%9B%BE%E7%89%871.png)


# 2 说明文档
## 2.1 scrapy架构组成 
（1）引擎——自动运行，无需关注，会自动组织所有的请求对象，分发给下载器。   
（2）下载器——从引擎处获取到请求对象后，请求数据。   
（3）Spiders——Spider类定义了如何爬取某个(或某些)网站。包括了爬取的动作(例 
如:是否跟进链接)以及如何从网页的内容中提取结构化数据(爬取item)。换句话说，Spider就是您定义爬取的动作及分析某个网页(或者是有些网页)的地方。   
（4）调度器——有自己的调度规则，无需关注。   
（5）管道（Item pipeline）——最终处理数据的管道，会预留接口供我们处理数据。当Item在Spider中被收集之后，它将会被传递到Item Pipeline，一些组件会按照一定的顺序执行对Item的处理。 
每个item pipeline组件(有时称之为“Item Pipeline”)是实现了简单方法的Python类。他们接收到Item并通过它执行。
一些行为，同时也决定此Item是否继续通过pipeline，或是被丢弃而不再进行处理。以下是item pipeline的一些典型应用：   
1. 清理HTML数据。   
2. 验证爬取的数据(检查item包含某些字段)。   
3. 查重(并丢弃)。   
4. 将爬取结果保存到数据库中。  

## 2.2 MySQL介绍
   MySQL是一种开源的关系型数据库管理系统（DBMS），它使用SQL语言进行数据库的管理和操作。由于其开源的特性，MySQL得到广泛的应用和支持，并且具有良好的稳定性和可靠性。
   MySQL可以在多个操作系统上运行，包括Windows、Linux和Mac OS等。它支持大多数编程语言，如Java、Python和PHP等，使得开发人员能够方便地与数据库进行交互。
   MySQL具有强大的性能和灵活的功能。它支持多用户并发访问，可以处理复杂的查询语句，并提供了丰富的索引和优化器来提高查询效率。此外，MySQL还支持事务处理和数据备份等特性，保证了数据的完整性和可靠性。

## 2.3 CrawlSpider介绍 
   CrawlSpider继承自scrapy.Spider，可以定义规则，再解析html内容的时候，可以根据链接规则提取出指定的链接，然后再向这些链接发送请求。所以，如果有需要跟进链接的需求，意思就是爬取了网页之后，需要提取链接再次爬取，使用CrawlSpider是非常合适的。

# 3 使用文档
1.创建项目：scrapy startproject dushuproject。   
2.跳转到spiders路径 cd\dushuproject\dushuproject\spiders。   
3.创建爬虫类：scrapy genspider ‐t crawl read www.dushu.com。   
4.检查items.py。   
5.检查spiders.py。   
6.检查settings.py。   
7.检查pipelines.py。   
8.在命令行中输入scrapy crawl read 运行scrapy爬虫框架。  

数据保存到本地MySQL数据库   
（1）settings配置参数：   
```python
DB_HOST = '192.168.231.128'   
DB_PORT = 3306   
DB_USER = 'root'   
DB_PASSWORD = '1234'   
DB_NAME = 'test'   
DB_CHARSET = 'utf8'//python
```
（2）管道配置
```python
from scrapy.utils.project import get_project_settings 
import pymysql 
class MysqlPipeline(object): 
#__init__方法和open_spider的作用是一样的
#init是获取settings中的连接参数 
def __init__(self): 
   settings = get_project_settings() 
   self.host = settings['DB_HOST'] 
   self.port = settings['DB_PORT'] 
   self.user = settings['DB_USER'] 
   self.pwd = settings['DB_PWD'] 
   self.name = settings['DB_NAME'] 
   self.charset = settings['DB_CHARSET'] 
   self.connect() 
# 连接数据库并且获取cursor对象 
def connect(self): 
   self.conn = pymysql.connect(host=self.host, 
   port=self.port, 
   user=self.user, 
   password=self.pwd, 
   db=self.name, 
   charset=self.charset) 
   self.cursor = self.conn.cursor() 
def process_item(self, item, spider): 
   sql = 'insert into book(image_url, book_name, author, info) values("%s", 
   "%s", "%s", "%s")' % (item['image_url'], item['book_name'], item['author'], item['info']) 
   sql = 'insert into book(image_url,book_name,author,info) values 
   ("{}","{}","{}","{}")'.format(item['image_url'], item['book_name'], item['author'], 
   item['info']) 
   # 执行sql语句 
   self.cursor.execute(sql) 
   self.conn.commit() 
   return item 
def close_spider(self, spider): 
   self.conn.close() 
   self.cursor.close()
```
# 4 测试文档
   首先在使用管理员身份打开命令行，进入项目所在文件夹，输入scrapy crawl read 运行爬虫框架。  
   ![图 2](https://github.com/nanjingzhuyuxuan/PaChong/blob/main/picture/%E5%9B%BE%E7%89%872.png)

   运行完后，在PyCharm中可以看到爬取下的数据的json文件。  
   ![图 3](https://github.com/nanjingzhuyuxuan/PaChong/blob/main/picture/%E5%9B%BE%E7%89%873.png)


   通过运行pipelines将爬取下的数据导入到MySQL数据库中，共8872条数据。  
   ![图 4](https://github.com/nanjingzhuyuxuan/PaChong/blob/main/picture/%E5%9B%BE%E7%89%874.png)

# 5 主要代码节选
```python
class ReadSpider(CrawlSpider)://python
   name = 'read'
   allowed_domains = ['www.dushu.com']
   start_urls = ['https://www.dushu.com/book/1188_1.html']

   rules = (
       Rule(LinkExtractor(allow=r'/book/1188_\d+.html'),
                          callback='parse_item',
                          follow=True),
   )

   def parse_item(self, response):
       img_list = response.xpath('//div[@class="bookslist"]//img')
       for img in img_list:
           name = img.xpath('./@data-original').extract_first()
           src = img.xpath('./@alt').extract_first()
           book = ScrapyReadbook101Item(name=name,src=src)
           yield book

class ScrapyReadbook101Pipeline:
   def open_spider(self,spider):
       self.fp = open('book.json','w',encoding='utf-8')

   def process_item(self, item, spider):
       self.fp.write(str(item))
       return item

   def close_spider(self,spider):
       self.fp.close()

# 加载settings文件
   def open_spider(self,spider):
       settings = get_project_settings()
       self.host = settings['DB_HOST']
       self.port =settings['DB_PORT']
       self.user =settings['DB_USER']
       self.password =settings['DB_PASSWROD']
       self.name =settings['DB_NAME']
       self.charset =settings['DB_CHARSET']
       self.connect()

   def connect(self):
       self.conn = pymysql.connect(
                           host=self.host,
                           port=self.port,
                           user=self.user,
                           password=self.password,
                           db=self.name,
                           charset=self.charset
       )
       self.cursor = self.conn.cursor()

   def process_item(self, item, spider):
       sql = 'insert into book(name,src) values("{}","{}")'.format(item['name'],item['src'])
       # 执行sql语句
       self.cursor.execute(sql)
       # 提交
       self.conn.commit()
       return item

   def close_spider(self,spider):
       self.cursor.close()
       self.conn.close()//python
     
