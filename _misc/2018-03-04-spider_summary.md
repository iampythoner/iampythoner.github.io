---
layout: cnblog_post
title:  "spider_summary"
permalink: '/misc/spider_summary'
date:   2018-03-04 07:34:39
categories: misc
---
```

############################ requests的使用
import requests
# 发送请求， 使用的函数按照HTTP访问的方法，主要：get、post、put、delete、head、options，patch
# 最终都是调用了request函数，这个函数的原型是： def request(method, url, **kwargs):
# kwargs可以填入：headers, params, data, json， proxies
response = requests.get('https://www.baidu.com')
# 返回的对象是 requests.models.Response
# response对象通常有以下几个方法：
response.status_code # 状态码
response.request.headers # 对应请求的请求头
response.headers # 响应头
response.encoding # 默认解码方式，修改之前默认是ISO-8859-1
response.text # 按照response.encoding 解码的str
response.content # bytes
print(response.content.decode('utf-8'))
######## 保持会话进行爬取，每次请求都会带上cookie
session = requests.session() # requests.sessions.Session类型
response = session.get(url,headers)
####### 小技巧
cookie转为字典 requests.util.dict_from_cookiejar
https请求不进行ssl验证 requests.get('https://www.baidu.com', verify=False)
设置超时 requests.get(url, 1)

################################### 正向代理和反向代理
代理为谁服务就隐藏谁的信息。

正向代理：代理为客户端服务，产生的结果是：服务端不知道真正的客户端
反向代理：代理为服务端服务，并对服务集群做负载均衡，将请求均衡分配到空闲服务器，
产生的结果是：客户端不知道真正的服务器。

nginx服务器的作用
1.C-S架构模式解耦，他是一个中间层，可以随时更改绑定的服务器
2.负载均衡，避免请求拥塞，提升并发量
3.反向代理，配合服务器集群提供了按照配置、服务地区的最有服务

CDN都是反向代理的应用，但他们都不代表反向代理
1. 内容分发网络CDN, 通过反向代理把请求分发到世界各地的服务器去, 提升访问速度.
2. 大公司不想暴露自己主服务器的位置.

阿里云CDN配置------
https://help.aliyun.com/document_detail/27112.html
1.到CDN控制台添加域名
指定业务类型：（多选）全站加速、图片小文件、大文件、音视频点播、直播流媒体、移动加速
指定源站类型：OSS域名 IP 源站域名
2.域名管理：配置CNAME
复制CNAME， 到DNS服务商控制台，添加一条CNAME记录

################################  关于反爬
1.拒绝请求：
①User-Agent  反反爬： 准备User-Agent池
②Host、Referer 反反爬：修改Header，复制浏览器访问时的内容
③Cookie：反反爬：使用浏览器Cookie， 保持会话request使用session或直接使用selenium
    需要登录：准备多个账号， 单独使用程序实现登录保存cookie， 爬虫程序读取cookie发送请求
    不需要登录：单独使用程序实现登录保存cookie， 爬虫程序读取cookie发送请求
④ip， 反反爬：使用ip池，使用代理
2.数据干扰
①伪装：如请求的明明是JSON，但返回image，再通过js解密数据,
通常可以通过js代码得到解密方法；如果不能，只能爬取网页。
②加密：如果不能轻易破解接口加密，只能通过爬取网页了。
③自定义字体：通常用来干扰数字数据，下载字体下来，建立对应关系(手动或者fontmeta)，并在爬虫程序中通过对应关系还原。

############################## 数据提取过程， 由容易到复杂
############# 数据提取概述
非结构化的数据：html等
处理方法：正则表达式、xpath----->将提取的信息字符串构造Python类型

结构化数据：json，xml等
处理方法：转化为python数据类型---->利用三方库直接转换为Python类型
############# 具体的数据提取过程
1.如果有ajax请求的JSON数据，可直接使用，通常这种数据有两种形式：
    ①pure JSON形式
    ②带回调形式的JSON，通常是xxxx(JSON)的形式
    可以通过使用正则进一步提取，有的可以直接将请求参数中的callback参数直接去掉，去掉之后直接返回pure JSON
    ③接口请求过来一段字符串，类似于字典格式，但是不是标准JSON，其中某个字段的值是JSON，可以通过re提取。
    手机页面的数据经常使用JSON
    # re.compile（编译）
	# pattern.match（从头找一个
	# pattern.search（找一个）
	# pattern.findall（找所有）
	# pattern.sub（替换
    # 元字符“.”在默认模式下，匹配除换行符外的所有字符。在DOTALL模式下，匹配所有字符，包括换行符

    # 通过正则表达式匹配内涵段子的段子
    #  http://neihanshequ.com/
    # 获取http://36kr.com/网站首页的所有新闻

2.处理HTML页面：
    ①xpath：
        xpath 使用的lxml这个HTML/XML解析器实现，可以使用xpath的语法定位特定元素以及获取节点信息，常用语法
        nodename: 节点名称
        / 从根节点选取
        // 当前节点
        . 选取当前节点
        .. 当前节点的父节点
        @ 选取属性

        //title[@lang] 拥有属性lang的title元素
        //title[@lang='eng'] 拥有值为eng的lang属性的title元素
        ### 数组操作
        /bookstore/book[1] 第一个book元素
        /bookstore/book[last()] 最后一个book元素
        /bookstore/book[position()<3] 前2个
        /bookstore/book[price>35.00] # price标签文本值>35.00的book元素

        # 通配符
        *任何元素节点
        @*任何属性节点
        node() 任何节点 相当于 * 和 @* 之和
        ############## xpath 配合Python使用
        from lxml import etree
        element = etree.HTML(html_str)
        script = element.xpath(
            '//div[@class="scontent" and @id="bofqi"]/script/text()')
        # 1、爬取糗事百科段子，页面的URL是
        #      http://www.qiushibaike.com/8hr/page/1
        # 2、动手把上述的爬虫改为多线程爬虫

3.动态页面处理，通常是根据登录用户的信息不同而提供不同的页面数据。
    这类信息的爬取，首先要进行登录：登录过程分为以下几种：
    ⑴没有验证码：
        情况一：登录字段没有加密或者非常容易看出加密手段：直接向表单提交地址发送请求(GitHub)
        情况二：登录字段加密手段无法破解：selenium
    ⑵验证码刷新不改变：
        主要原理是：获取验证码图片的接口，判断携带的cookie相同就返回相同的验证码
        情况一：requests + 识图工具
        情况二：使用selenium + 识图工具(如云打码）可直接传递验证码图片的url
    ⑶验证码刷新改变：（百度、钱多多 http://qian.sicent.com/Login/code.do ）
        原理：
        如果是动态获取的图片：ajax请求参数或者header会不同，这个可以通过对比参数查看(百度)
        如果是header和参数都相同，两次请求的图片依然不同，那么能够将登陆操作和验证码关联的就只有ip、
        session等信息了。大部分情况是使用了缓存服务器存储了session-id对应的code，但每次获取验证码都会更改它。
        情况一：requests + 识图工具，不能直接使用图片的url，必须下载下来再上传识图
        情况二：selenium + 识图工具，不能直接使用图片的url，必须下载下来再上传识图

登录之后如何保持登录状态：
        ①requests直接使用session保持会话爬取
        ②每次请求都携带登录之后传递的cookie，这样无法使用动态cookie，
        有错误的情况，可能在某个页面请求之后cookie更改
        ③使用Scrapy如何保持登录状态
            下载器中间件CookiesMiddleware
            class scrapy.downloadermiddlewares.cookies.CookiesMiddleware
            该中间件使得爬取需要cookie(例如使用session)的网站成为了可能。
            其追踪了web server发送的cookie，并在之后的request中发送回去， 就如浏览器所做的那样。
https://scrapy-chs.readthedocs.io/zh_CN/1.0/topics/downloader-middleware.html#module-scrapy.downloadermiddlewares.cookies

    ####### selenium使用和注意点
    ①基本使用
        from selenium import webdriver
        driver = webdriver.Chrome()
        tag = driver.find_elements_by_xpath('') # list 或者 selenium.webdriver.remote.webelement.WebElement
        # driver.switch_to.frame()
        # driver.save_screenshot
        # driver.delete_cookie("CookieName")
        # driver.delete_all_cookies()
        2.定位元素
        find_element_by_id(返回一个)
        find_elements_by_xpath （返回一个列表）
        find_elements_by_link_text
        find_elements_by_partial_link_text
        find_elements_by_tag_name
        find_elements_by_class_name
        find_elements_by_css_selector
        3.查看页面相关信息
        # driver.page_source
        # driver.get_cookies()
        # driver.current_url
        4.关闭
        # driver.close() #退出当前页面
        # driver.quit()  #退出浏览器

    ②获取登陆后的cookie
        {cookie[‘name’]: cookie[‘value’] for cookie in driver.get_cookies()}
    ③页面等待
        time.sleep(10)
    ########
    # 1.豆瓣登陆
    # 2.爬取斗鱼直播平台的所有房间信息
    # https://www.douyu.com/directory/all

4.几种常见的页面数据组织形式：
    初始页 + ajax请求的JSON (有道词典)
    初始页 + ajax请求的HTML片段 （百度贴吧首页下拉、京东搜索下拉）
    初始页 + iframe (QQ邮箱登录，网易云音乐)

################################ 数据清洗

数据的完整性----例如人的属性中缺少性别、籍贯、年龄等
数据的唯一性----例如不同来源的数据出现重复的情况
数据的权威性----例如同一个指标出现多个来源的数据，且数值不一样
数据的合法性----例如获取的数据与常识不符，年龄大于150岁
数据的一致性----例如不同来源的不同指标，实际内涵是一样的，或是同一指标内涵不一致

##### 技术实现

################################ 数据去重
url去重的场景：
一次性爬取的数据，短期内同一url数据不会改变，可以通过url去重。
长期爬取的数据，如果网站url本身是静态的，要注意筛选是否是伪静态url的网站，也可以直接通过url去重。

其它数据去重场景：
①数据入库阶段去重：
单个字段：比如url
通过数据本身：建立联合索引（用户id,发帖日期，数据来源）

②数据入库前去重：
    ①请求去重：
        思路一：url去重(使用情况：url对应一条数据不会变的情况):
            方法一：把url地址存在redis的集合中，后续根据url是否存在于redis的集合中，来判断数据是否抓过
            方法二：布隆过滤器(适用于文章内容去重)是一种位操作，使用10个算法给每个url地址生成10个值，每个值代表一个位置，拿到一个url地址，先取10个位置的值，如果全为1，表示url地址存在，如果不全为1，表示url地址不存在，对应的往该位置置位1
        思路二：url+headers去重(Scrapy-redis实现)
    ②数据去重：
        思路一：Hash去重
            1.选择数据中的关键字段，使用hash方法（md5,sha1,）把关键字段进行加密，生成字符串
            2.把字符串存入redis的集合中
                如果抓过，更新对应的数据，没有抓过，插入

########################################## Scrapy 概况

1.「Scrapy Engine(引擎)
总指挥: 负责数据和信号的在不同模块间的传递
scrapy已经实现,
2.Scheduler(调度器)
一个队列，存放引擎发过来的request请求
scrapy 已经实现
3.Downloader (下载器)
下载把引擎发过来的requests请求，并返回给引擎
scrapy 已经实现
4.Spider (爬虫)
处理引擎发来来的response,提取数据，提取url,并交给引擎
需要手写
5.Item Pipeline(管道)
处理引擎传过来的数据，比如存储
需要手写
6.DownloaderMiddlewares(下载中间件)
可以自定义的下载扩展，比如设置代理
一般不用手写
7.(SpiderMiddlewaresSpider(中间件)
可以自定义requests请求和进行response过滤
一般不用手写

################ 请求和数据流向
调度器 ==> 获取request对象(遍历spider通过yield产生的Request对象) ===> 引擎 ===> 下载中间件 ==> 下载器
下载器发送请求，获取响应 ==> 下载中间件 ==> 引擎 ==> 爬虫中间件 ==> 爬虫(交给设定的callback或默认初始请求对应的parse方法)
爬虫 提取数据 ==> 交给引擎(遍历spider通过yield产生的dict、item对象) ==> 管道
爬虫 url 组成发request对象 ==> 爬虫中间件 ==> 引擎 ==> 调度器

################ Scrapy创建项目
1.创建一个项目
scrapy startproject mySpider
2.生成一个爬虫
scrapy genspider tieba "tieba.baidu.com"
3.提取数据，创建新请求，构建数据
完善spider，使用xpath等方法
4.保存数据
pipeline中保存数据

########################################## Scrapy 具体使用
项目结构
├── justspider
│   ├── __init__.py
│   ├── items.py
│   ├── middlewares.py
│   ├── pipelines.py
│   ├── settings.py
│   └── spiders
│       ├── __init__.py
│       └── tieba.py
└── scrapy.cfg

############ settings文件分析
BOT_NAME = 'justspider'
SPIDER_MODULES = ['justspider.spiders']
NEWSPIDER_MODULE = 'justspider.spiders'
### 其他主要配置
1.User-Agent
#USER_AGENT = 'justspider (+http://www.yourdomain.com)'
2.是否遵守robots协议
ROBOTSTXT_OBEY = True
3.最大并发数
#CONCURRENT_REQUESTS = 32 默认是16
4.自动限流配置 autothrottle, 下载延迟
DOWNLOAD_DELAY = 3
5.是否使用cookie 默认是enable
#COOKIES_ENABLED = False
6.是否使用telnet控制台 默认是enable
#TELNETCONSOLE_ENABLED = False
7.重写默认的请求headers
#DEFAULT_REQUEST_HEADERS = {
#   'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
#   'Accept-Language': 'en',
#}
8.配置默认的爬虫中间件
#SPIDER_MIDDLEWARES = {
#    'justspider.middlewares.JustspiderSpiderMiddleware': 543,
#}
9.配置默认的下载中间件
#DOWNLOADER_MIDDLEWARES = {
#    'justspider.middlewares.JustspiderDownloaderMiddleware': 543,
#}
10.是否使用扩展
#EXTENSIONS = {
#    'scrapy.extensions.telnet.TelnetConsole': None,
#}
11.配置item使用的pipeline
#ITEM_PIPELINES = {
#    'justspider.pipelines.JustspiderPipeline': 300,
#}
12.自动限流的扩展，默认不使用
① 打开扩展
#AUTOTHROTTLE_ENABLED = True
②初始下载延迟
#AUTOTHROTTLE_START_DELAY = 5
③最大的下载延迟
#AUTOTHROTTLE_MAX_DELAY = 60
④平均并行请求数
#AUTOTHROTTLE_TARGET_CONCURRENCY = 1.0
⑤是否为每个接收到的响应显示流量状态
#AUTOTHROTTLE_DEBUG = False

13.打开、配置HTTP缓存(默认禁用)
①是否打开配置
#HTTPCACHE_ENABLED = True
②缓存过期的时长 秒为单位
#HTTPCACHE_EXPIRATION_SECS = 0
③ 缓存的目录
#HTTPCACHE_DIR = 'httpcache'
④对哪些状态码的请求不缓存
#HTTPCACHE_IGNORE_HTTP_CODES = []
⑤实现缓存存储后端的类
#HTTPCACHE_STORAGE = 'scrapy.extensions.httpcache.FilesystemCacheStorage'

######################## Spider重写,控制数据提取和继续请求的流程
启动爬虫
start_url地址的响应===>交给parse函数传入response===>parse函数提取数据
--------提取方法
response的选择器(response是一个scrapy.http.response.html.HtmlResponse对象)
①response.xpath()、②response.css()、③response.selector.re()
--------拿到数据列表之后再次提取
extract()提取数据，结果是列表，没有是空列表
extract_first()提取结果中第一个字符串，没有就是None
可以对获取的对象继续使用xpath，css和re
--------继续操作
情况一：需要构造新的请求(下一页，详情页等)，指定callback，以及为response设置的meta
    yield scrapy.Request(item['detail_url'],
                         callback=self.parse_detail,
                         meta={'item': item})
    # dont_filter：默认False，url会被去重;设置为True，可以在当前爬虫中对相同url请求多次
    所有的callback和初始parse方法只有一个response参数，meta会作为response的属性传递
情况二：产生数据，这个数据会通过引擎交给pipeline
    产生的数据可以是Item(scrapy.Item子类)、dict类型，具体代码为：
    item = XXItem()
    item.name = xx
    yield item
    其中XXItem类的实现为Item:
    class SuningItem(scrapy.Item):
        name = scrapy.Field()
-------scrapy.Spider类一些其他注意点
-----1
方法start_requests的默认实现为
def start_requests(self):
    ###......
    for url in self.start_urls:
        yield Request(url, dont_filter=True)
会遍历spider的start_urls这个list，并以此构建Request对象开始爬虫。
有一些场景需要重写这个方法，比如在爬虫开始直接导入用户登录的cookie，
这个时候不要忘记yield一个初始Request，否则爬虫无法开始。(Renren登录)
-----2
allowed_domains要添加可能爬取的所有url的domain，否则将无法爬取
########################  pipeline重写，数据持久化或者消费处理
可以不重写，那么产生的数据不会进行任何处理

通常重写以下几个方法：
1.open_spider(self, spider) 爬虫开始的时候执行一次
2.close_spider(self, spider) 爬虫结束的时候执行一次
可以在open_spdier中建立和数据库的连接，在close_spide关闭和数据库的连接
3.from_crawler(cls, crawler)
如果重写了，这个类方法会在Crawler中创建pipeline实例的时候调用。它必须返回一个新的pipeline实例。
Crawler对象提供了对所有Scrapy核心组件的访问权限，如settings何signals，
对pipeline来说，这个方法是访问和hook Scrapy基础设施的手段。
4.process_item(self, item, spider)
每个item pipeline组件都需要调用该方法，这个方法必须返回一个具有数据的dict，
或是 Item (或任何继承类)对象， 或是抛出 DropItem 异常，被丢弃的item将不会被之后的pipeline组件所处理。

---例子：
import pymongo

class MongoPipeline(object):

    collection_name = 'scrapy_items'

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        self.db[self.collection_name].insert(dict(item))
        return item
--------------另外，可以在数据入库之前添加去重的pipeline
from scrapy.exceptions import DropItem

class DuplicatesPipeline(object):

    def __init__(self):
        self.ids_seen = set()

    def process_item(self, item, spider):
        if item['id'] in self.ids_seen:
            raise DropItem("Duplicate item found: %s" % item)
        else:
            self.ids_seen.add(item['id'])
            return item
----只需要在settings设置去重pipeline的权重小于入库pipeline即可。
######################## 中间件 及 配置
############### 下载中间件
如果您想禁止内置的(在 DOWNLOADER_MIDDLEWARES_BASE 中设置并默认启用的)中间件，
您必须在项目的 DOWNLOADER_MIDDLEWARES 设置中定义该中间件，并将其值赋为 None 。
例如，如果您想要关闭user-agent中间件:
DOWNLOADER_MIDDLEWARES = {
    'myproject.middlewares.CustomDownloaderMiddleware': 543,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
}
##### 经常重写的下载中间件方法
1.process_request(request, spider) 处理请求
返回 None 、返回一个 Response 对象、返回一个 Request 对象或raise IgnoreRequest
2.process_response(request, response, spider)
返回一个 Response 对象、 返回一个 Request 对象或raise一个 IgnoreRequest 异常。
3.process_exception(request, exception, spider)
当下载处理器(download handler)或 process_request() (下载中间件)抛出异常(包括 IgnoreRequest 异常)时， Scrapy调用 process_exception() 。
process_exception() 应该返回以下之一: 返回 None 、 一个 Response 对象、或者一个 Request 对象。
如果其返回 None，Scrapy将会继续处理该异常，接着调用已安装的其他中间件的 process_exception() 方法，直到所有中间件都被调用完毕，则调用默认的异常处理。
如果其返回一个 Response 对象，则已安装的中间件链的 process_response() 方法被调用。Scrapy将不会调用任何其他中间件的 process_exception() 方法。
如果其返回一个 Request 对象， 则返回的request将会被重新调用下载。这将停止中间件的 process_exception() 方法执行，就如返回一个response的那样
----- 常用的内置下载中间件
CookiesMiddleware 默认使用，可以通过settings的COOKIES_ENABLED设置是否禁用
DefaultHeadersMiddleware 指定request的header
DownloadTimeoutMiddleware 依据settings中的DOWNLOAD_TIMEOUT设置超时
HttpAuthMiddleware 该中间件完成某些使用 Basic access authentication (或者叫HTTP认证)的spider生成的请求的认证过程，
HttpCacheMiddleware  该中间件为所有HTTP request及response提供了底层(low-level)缓存支持。 其由cache存储后端及cache策略组成。
HttpCompressionMiddleware 压缩支持 通过settings中的设置COMPRESSION_ENABLED，默认True
HttpProxyMiddleware 代理支持，通过创建Request时传递代理ip+port
RedirectMiddleware 该中间件根据response的状态处理重定向的request。
MetaRefreshMiddleware 该中间件根据meta-refresh html标签处理request重定向。
RetryMiddleware 该中间件将重试可能由于临时的问题，例如连接超时或者HTTP 500错误导致失败的页面 settings:RETRY_TIMES RETRY_HTTP_CODES
RobotsTxtMiddleware 该中间件过滤所有robots.txt eclusion standard中禁止的request。settings: ROBOTSTXT_OBEY
UserAgentMiddleware 用于**覆盖**spider的默认user agent的中间件。
AjaxCrawlMiddleware 根据meta-fragment html标签查找 ‘AJAX可爬取’ 页面的中间件
############### Spider中间件
---- 经常重写的Spider中间件
1.process_spider_input(response, spider)
当response通过spider中间件时，该方法被调用，处理该response。
process_spider_input() 应该返回 None 或者抛出一个异常。
2.process_spider_output(response, result, spider)
当Spider处理response返回result时，该方法被调用。
process_spider_output() 必须返回包含 Request 、dict 或 Item 对象的可迭代对象
3.process_spider_exception(response, exception, spider)
当spider或(其他spider中间件的) process_spider_input() 跑出异常时， 该方法被调用。
4.process_start_requests(start_requests, spider)
该方法以spider 启动的request为参数被调用，执行的过程类似于 process_spider_output() ，
只不过其没有相关联的response并且必须返回request(不是item)。
---- 常用的内置Spider中间件
1.DepthMiddleware DepthMiddleware是一个用于追踪每个Request在被爬取的网站的深度的中间件。
其可以用来限制爬取深度的最大深度或类似的事情。 settings:DEPTH_LIMIT
2.HttpErrorMiddleware
过滤出所有失败(错误)的HTTP response，因此spider不需要处理这些request。
处理这些request意味着消耗更多资源，并且使得spider逻辑更为复杂。
3.OffsiteMiddleware
过滤出所有URL不由该spider负责的Request。
该中间件过滤出所有主机名不在spider属性 allowed_domains 的request。
4.RefererMiddleware
根据生成Request的Response的URL来设置Request Referer 字段。
5.UrlLengthMiddleware
过滤出URL长度比URLLENGTH_LIMIT的request。
#################### Scrapy处理表单请求，如何构造Request用于提交表单(GitHub为例)
1.带上cookie直接请求
    1.构造cookie字典
    2.添加到scrapy.Request(cookies=cookies)
    3.cookie字符串不能够放到headers，没有效果
2.scrapy.FormReuquest
代码：scrapy.FormReuquest（url,formdata,callback）
其中url是要post提交的url地址，formdata，请求体
3.scrapy.FormReuquest.from_response(response,formdata,callback)
根据当前页面的response自动选取表单，formdata只需填入Web页面要填写的字段即可。
注意：如果有多个表单，可以使用formxpath或formcss这两个参数选取

###################### CrawlSpider
1.创建的命令
    scrapy genspider -t crawl spdier_name allow_domain
2.生成的CrawSpider类继承自Spider，增加类属性rules，是一个Rule类型的元组，parse方法变为parse_item
特点：
    1.根据Rule元组定义的规则寻找url，构建请求，如此反复，通过最终的callback做数据提取
    2.callback依然可以yield新的Request，继续使用Rule元组继续筛选url，
缺点：crawlspider的解析函数中间不能传递数据，经常在详情页能够获取全部数据的时候使用crawlspider,
        parse_item就是为解析最终拿数据的页面准备的
3.Rule构建，有几个重要的参数：
①link_extractor链接提取器
LinkExtractor(allow=r'/p/\d{10}') # restrict_xpaths、restrict_css都是元组类型
②callback，回调函数，可用于yield Request或Item
③follow，根据当前提取规则得到response之后，是否继续使用rules下的Rule继续提取

###################### logger
scrapy中使用日志：
settings：LOG_FILE = "./log.log" # 日志存储的位置,终端不会再展示日志
scrapy中使用日志输出信息：
方式①：
    1.import logging
    2. logger = logging.getLogger(__name__)
        # __name__ 用于format,通常显示为提示[__name__]
        能够显示当前的输出来自于那个文件
    3.logger.warning("....")
方式②：
    如果当前类是Spider，直接使用
    self.logger.warning('....')
可以在settings里面通过LOG_LEVEL设置日志级别，还可以配置其他设置：
LOG_FILE、LOG_ENABLED、LOG_ENCODING、LOG_FORMAT、LOG_DATEFORMAT、LOG_STDOUT
########### 一般项目使用系统的logging模块
1.import logging
2.logging.BasicConfig({})
    配置日志数据的样式
3.实例化logger = logging.getLogger(__name__)
4.导入logger，使用logger.info("") <=> logging.log(logging.INFO, "")
################################# Scrapy Shell
怎么使用
    scrapy shell url
能干什么
    观察scarpy中各种对象的方法和属性
        response.text  网页的html字符串str
        response.body  获取网页的响应的bytes
        resposne.request.url
        resposne.url
        response.request.url
    测试xpath
############ Scrapy部署
############ telnet终端
############ feed export
########################################## Scrapy-Redis
scarpy_redis是一个基于redis的scarpy的组件
主要作用：
    1.持久化`待爬取`的request对象，经过序列化存入redis中，可以实现暂停、断点续爬、分布式爬虫
    2.已经看到过的request的指纹存入redis，作为去重的依据
工作流程：
	1.调度器的带爬取的request对象，已经看到过的request的指纹保存在redis中
	2.itempipeline的数据默认保存在redis中
和scrapy的区别：
    settings多了5行
        1.指定了调度器的类
        2.指定了去重的类
        3.指定了redis_url
DUPEFILTER_CLASS = 'scrapy_redis.dupefilter.RFPDupeFilter'
SCHEDULER = 'scrapy_redis.scheduler.Scheduler'
SCHEDULER_PERSIST = True
REDIS_URL = 'redis://192.168.199.131:6379/5'
ITEM_PIPELINES = {
    'book.pipelines.BookPipeline': 300,
    'scrapy_redis.pipelines.RedisPipeline': 400,
}

######  scrapy_reids如何实现--概述
	1.去重
		1.hashlib.sha1()
		2.加密了request对象的请求方法，请求体，url
		3.生成16进制字符串作为当前request的指纹
		4.把指纹存在redis的集合中，后续的request的指纹判断是否存在该集合中
	2.待爬取的request对象入队
		1.url在start_urls中的时候
		2.url地址之前没有见过的时候
		3.dont_fileter=Ture,表示不过滤，会入队
################################# 源码分析

---------------计算指纹的方法：
from scrapy.utils.request import request_fingerprint
这个函数默认会忽略cookie和其他header，如果想让一些header参与指纹计算
使用include_headers参数，它是要包含的请求头列表。
最终参数与计算hash的值按顺序为：
请求method，url，body，include_header和对应的value，使用的是sha1算法
----------------指纹过滤器
RFPDupeFilter (Request Fingerprint duplicates filter) 请求指纹重复过滤器

### scrapy默认的指纹去重过滤器
scrapy.dupefilters.RFPDupeFilter 继承自BaseDupeFilter
1.对象属性fingerprints是set()类型，用于在内存中存储，已经访问过的request指纹
2.如果对象实例化时传入了path参数，那么会在path路径下的生成requests.seen用于持久化存储，只需在settings
指定DUPEFILTER_DEBUG即可，那么会使用下面这个类方法实例化这个类
def from_settings(cls, settings):
    debug = settings.getbool('DUPEFILTER_DEBUG')
    return cls(job_dir(settings), debug)
# job_dir 实现为
def job_dir(settings):
    path = settings['JOBDIR']
    if path and not os.path.exists(path):
        os.makedirs(path)
    return path
可知：requests.seen 最终依赖于settings中JOBDIR这个配置,存储在JOBDIR指定的目录下
3.是否已经爬取过的判断：
def request_seen(self, request):
    fp = self.request_fingerprint(request)
    if fp in self.fingerprints:
        return True
    self.fingerprints.add(fp)
    if self.file:
        self.file.write(fp + os.linesep)
可知：直接返回内存缓存fingerprints这个set里面是否有请求的指纹

### scrapy-redis实现的指纹去重过滤器
scrapy_redis.dupefilter.RFPDupeFilter 继承自scrapy默认的BaseDupeFilter
1.key属性:用于存储redis存储该次请求的指纹的key，
dupefilter:%(timestamp)s % {'timestamp': int(time.time())}
server属性：对redis的链接
2.from_settings实例化对象
def from_settings(cls, settings):
    server = get_redis_from_settings(settings)
    key = defaults.DUPEFILTER_KEY % {'timestamp': int(time.time())}
    debug = settings.getbool('DUPEFILTER_DEBUG')
    return cls(server, key=key, debug=debug)
可知：通过settings的配置创建redis连接
3.是否已经爬取过的判断：
def request_seen(self, request):
    fp = self.request_fingerprint(request)
    added = self.server.sadd(self.key, fp)
    return added == 0
向redis插入指纹失败，则已经爬取过
----------------将要爬取的request放到队列中
----scrapy默认入队实现 scrapy.core.scheduler.Scheduler
def enqueue_request(self, request):
    if not request.dont_filter and self.df.request_seen(request):
        self.df.log(request, self.spider)
        return False
    dqok = self._dqpush(request)
    if dqok:
        self.stats.inc_value('scheduler/enqueued/disk', spider=self.spider)
    else:
        self._mqpush(request)
        self.stats.inc_value('scheduler/enqueued/memory', spider=self.spider)
    self.stats.inc_value('scheduler/enqueued', spider=self.spider)
    return True
----scrapy-redis入队实现 scrapy_redis.scheduler.Scheduler
def enqueue_request(self, request):
    if not request.dont_filter and self.df.request_seen(request):
        self.df.log(request, self.spider)
        return False
    if self.stats:
        self.stats.inc_value('scheduler/enqueued/redis', spider=self.spider)
    self.queue.push(request)
    return True

思路完全一致：
如果 request.dont_filter为True，或者，指纹过滤器验证没有重复，直接入队列

########################################## 其他知识点
代理IP的类型：（透明、匿名、高匿）
①透明代理的意思是客户端根本不需要知道有代理服务器的存在，但是它传送的仍然是真实的IP。你要想隐藏的话，不要用这个。
②普通匿名代理能隐藏客户机的真实IP，但会改变我们的请求信息，服务器端有可能会认为我们使用了代理。
不过使用此种代理时，虽然被访问的网站不能知道你的ip地址，但仍然可以知道你在使用代理，当然某些能够侦测ip的网页仍然可以查到你的ip。
③高匿名代理不改变客户机的请求，这样在服务器看来就像有个真正的客户浏览器在访问它，这时客户的真实IP是隐藏的，服务器端不会认为我们使用了代理。

一些其他情况的反反爬：http://www.shenjianshou.cn/blog/?p=292
```