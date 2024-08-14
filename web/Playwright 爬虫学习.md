p### Playwright 爬虫学习

#### 一、基本操作

##### （一）导包

```py
from playwright.sync_api import sync_playwright
from crawlab import save_item
```

sync_playwright 用来打开一个虚拟浏览器，save_item用来把数据保存到crawlab（非必须）。

注意，装crawlab时要使用pip install crawlab-sdk。如果直接用crawlab会报错。



##### （二）记录网页操作

使用命令

`playwright codegen --target python -o <py文件> -b chromium <网址>`

会打开一个浏览器，在上边的操作会被保存在py文件中且直接可执行。

注意，用这种方法生成的脚本只能按text寻找元素且不能识别前进和后退操作。



####  二、脚本结构

```python
def main():
    with sync_playwright() as p:
        # 启动浏览器
        browser = p.chromium.launch()
        context = browser.new_context()
        page = context.new_page()

        # 访问目标网址
        page.goto('https://price.ccement.com/tv/tvmap_world_phone/index.html', timeout=100000)
        
        # 逻辑代码
        
        browser.close()
```



####  三、元素的定位

在playwright中所有的操作都要基于page对象。

按筛选器定位：

```python
# 找所有class为.t-body的div下边的tr标签
page.query_selector_all('.t-body .tr')

# 找row下的第一个子td标签
country = row.query_selector('.td:nth-child(1)')

# 找class为indexinfo下边的，class为num的span标签
page.query_selector('div.indexinfo span.num')
```



按文本定位：

```python
country = "印度"

# 严格查询
country_item = page.get_by_text(country, exact=True)

# 模糊查询
country_item = page.get_by_text(country)
```



#### 四、page.wait_for_timeout的问题

爬虫时受制于网速，需要等待数据渲染出来才能爬到，所以要按需选择wait的时间。



#### 五、把数据保存到crawlab

```python
save_item({
    '洲': continent,
    '国家': country,
    '价格': price,
    '涨跌值': change_value,
    '涨跌幅': change_range,
    '时间': datetime_stamp
})
```

注意，一般数据库无法保存datetime.now()格式的时间，必须转换成时间戳：

```python
now = datetime.now()
today = datetime(now.year, now.month, now.day)
datetime_stamp = int(datetime.timestamp(datetime.now()))
```
