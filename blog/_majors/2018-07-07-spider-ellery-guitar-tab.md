---
title: 记一次抓取微信公众号上乐谱的经历
tags: [ python, spider, starred ]
---

**需求**

周末去打谱子，问问同学要不要帮忙顺便打一点，L同学直接推了个公众号过来，
叫ElleryGuitarTab。公众号质量很高，可惜没有提供批量下载的方式，交流群也没有。

正好[我之前做ITRClub的项目时做过微信公众号的爬虫][wxpa]，可以说是有经验。

[wxpa]: {% link _majors/2018-03-10-wx-pa-spider.md %}

**目录页分析**

```
<div class="article_list article_list_0">

	<a class="list_item js_post" href="http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&amp;mid=100000229&amp;idx=1&amp;sn=dc36b27899892ec61f8bbd9023971b2d&amp;scene=19#wechat_redirect">
	<div class="cover">
		<img class="img js_img" src="http://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4ibCmK2Ikthgddjyzuicd8atrcNSjAIic1A2rXBibCcqyWUF2b6DD408blg/0" alt="">
	</div>
	<div class="cont">
		<h2 class="title js_title">【TAB】 All Falls Down</h2>
		<p class="desc">Alan Walker ft. Noah Cyrus</p>
	</div>
</a>

<a class="list_item js_post" href="http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&amp;mid=100000438&amp;idx=1&amp;sn=f0f7f06e3d74205efe33e737d54949b9&amp;scene=19#wechat_redirect">
	<div class="cover">
		<img class="img js_img" src="http://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLP9icPmk4ISsQo2XSwK8sCuEES1iclsliaLSmD7JmRo3wjzY3ucvuVwlCbA/0" alt="">
	</div>
	<div class="cont">
		<h2 class="title js_title">【TAB】 Alone</h2>
		<p class="desc">Alan Walker</p>
	</div>
</a>

...

</div>
```

直接用BeautifulSoup处理：
```python
from bs4 import BeautifulSoup
article_list0 = """
<div class="article_list article_list_0">

			<a class="list_item js_post" href="http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&amp;mid=100000229&amp;idx=1&amp;sn=dc36b27899892ec61f8bbd9023971b2d&amp;scene=19#wechat_redirect">
				<div class="cover">
					<img class="img js_img" src="http://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4ibCmK2Ikthgddjyzuicd8atrcNSjAIic1A2rXBibCcqyWUF2b6DD408blg/0" alt="">
				</div>
				<div class="cont">
					<h2 class="title js_title">【TAB】 All Falls Down</h2>
					<p class="desc">Alan Walker ft. Noah Cyrus</p>
				</div>
			</a>

...
"""
soup = BeautifulSoup(article_list0, 'html5lib')

def links_extract(soup):
    dat = []
    for i in soup.find_all('a'):
        dat.append([i.h2.string, i['href']])
    return dat

links_extract(soup)
```

```
[['【TAB】 All Falls Down',
  'http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&mid=100000229&idx=1&sn=dc36b27899892ec61f8bbd9023971b2d&scene=19#wechat_redirect'],
 ['【TAB】 Alone',
  'http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&mid=100000438&idx=1&sn=f0f7f06e3d74205efe33e737d54949b9&scene=19#wechat_redirect'],
 ['【TAB】 Boulevard of Broken Dreams - Green Day',
  'http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&mid=100000752&idx=1&sn=d59233636a3b837371072a5f52c63c56&scene=19#wechat_redirect'],
 ['【TAB】 Careless Whisper',
  'http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&mid=100000787&idx=1&sn=3ddbbb2fb900aa568fa064bd2d591f9b&scene=19#wechat_redirect'],
...
```

**爬单个谱子**

谱子是图片格式的。试试直接解析网页源代码来获取图片：

```python
import selenium
from selenium import webdriver
wd = webdriver.Chrome()
wd.get('http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&mid=100000229&idx=1&sn=dc36b27899892ec61f8bbd9023971b2d&scene=19#wechat_redirect')

soup = BeautifulSoup(wd.page_source, 'html5lib')
soup.find_all('img')
```

```
[<img alt="" class="profile_avatar" id="js_profile_qrcode_img" src=""/>,
 <img class="" data-copyright="0" data-fail="0" data-ratio="1.41484375" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4aynvIQ3VB7pVsicpib3PpysXiaWdID3LUKGKVWcUJia0qpJrxqicdMKWsNw/640?wx_fmt=jpeg" data-type="jpeg" data-w="1280" src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4aynvIQ3VB7pVsicpib3PpysXiaWdID3LUKGKVWcUJia0qpJrxqicdMKWsNw/640?wx_fmt=jpeg&amp;tp=webp&amp;wxfrom=5&amp;wx_lazy=1" style="width: auto !important; height: auto !important; visibility: visible !important;"/>,
 <img class="img_loading" data-copyright="0" data-ratio="1.41484375" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4sicKIpmomgBGWibGsnG6pVEleXTwjXchzcHTBwvfflNExICCz02FWQOQ/640?wx_fmt=jpeg" data-type="jpeg" data-w="1280" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="width: 677px !important; height: 957.849px !important;"/>,
 <img class="img_loading" data-copyright="0" data-ratio="1.41484375" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4x9LbUxoZWAOX6G4qzf8vJKQ2iatiar08w8t3BP0PHciaiabDclnHUJ5zAg/640?wx_fmt=jpeg" data-type="jpeg" data-w="1280" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="width: 677px !important; height: 957.849px !important;"/>,
 <img class="img_loading" data-copyright="0" data-ratio="1.41484375" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4ibJoz5GEiaencoKhx8T8vbZ1EzLI2J9tQeRwEskABBbcTia6Sq3YcAnCQ/640?wx_fmt=jpeg" data-type="jpeg" data-w="1280" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="width: 677px !important; height: 957.849px !important;"/>,
 <img class="img_loading" data-copyright="0" data-ratio="1.41484375" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4VTrUiaxyefSkplJsyJrbnEEicRJIMQBGf6bSUUeTc2NjAqf2xicJk9T8w/640?wx_fmt=jpeg" data-type="jpeg" data-w="1280" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="width: 677px !important; height: 957.849px !important;"/>,
 <img class="img_loading" data-copyright="0" data-ratio="1.41484375" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4QEW5R0bNPEmibotz4qbH6icE6KCafMzjYZcpwHAQR9MHtYYJH8YpictYg/640?wx_fmt=jpeg" data-type="jpeg" data-w="1280" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="width: 677px !important; height: 957.849px !important;"/>,
 <img class="img_loading" data-copyright="0" data-ratio="1.41484375" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4dXFVoODNuakuu4JJVIySr1D4cTesXd5GR1HuaeSfOib09S8muBpdmBw/640?wx_fmt=jpeg" data-type="jpeg" data-w="1280" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="width: 677px !important; height: 957.849px !important;"/>,
 <img class="img_loading" data-copyright="0" data-ratio="1.41484375" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4A59RjLs1WEXFqLiakniaYtWbTfKK3hsiauhSpyv7XzicQna0oicshpFnRQg/640?wx_fmt=jpeg" data-type="jpeg" data-w="1280" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="width: 677px !important; height: 957.849px !important;"/>,
 <img _width="100%" class="img_loading __bg_gif" data-order="0" data-ratio="0.1591696" data-src="https://mmbiz.qpic.cn/mmbiz_gif/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4lqz74a2yFAibDr61n7kBPFwrsWcicuDv9nib8CIKZg9FRZfsXgSa0ibdEg/640?wx_fmt=gif" data-type="gif" data-w="578" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="vertical-align: middle; box-sizing: border-box; width: 609px !important; height: 96.9343px !important;" width="100%"/>,
 <img class="img_loading" data-copyright="0" data-ratio="0.5577689243027888" data-s="300,640" data-src="https://mmbiz.qpic.cn/mmbiz_png/lvZ1HogYhgJo928FIA90vK1dg2wiblJ9ZnI1MI0Sru24fD2j4SKvM8uejwy22rNHOico7UqhicDvvVsibI3Agk0ib3w/640?wx_fmt=png" data-type="png" data-w="502" src="data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==" style="width: 502px !important; height: 280px !important;"/>,
 <img alt="" id="js_preview_reward_author_head" src=""/>,
 <img class="reward_qrcode_img" src="//res.wx.qq.com/mmbizwap/en_US/htmledition/images/pic/appmsg/pic_reward_qrcode.2x3534de.png"/>,
 <img class="qr_code_pc_img" id="js_pc_qr_code_img" src="/mp/qrcode?scene=10000004&amp;size=102&amp;__biz=MzUxOTY3NDYzNQ==&amp;mid=100000229&amp;idx=1&amp;sn=dc36b27899892ec61f8bbd9023971b2d&amp;send_time="/>,
 <img id="js_pc_weapp_code_img"/>]
```

接着问题来了：怎么区分乐谱和其他图片呢？最简单的思路是训练一个
神经网络（笑）。注意到：
1. `<img>`有一个特别的属性data-src，就是图片链接；
2. 所有乐谱的格式都是jpg

```python
def get_img(lnk):
    wd.get(lnk)
    soup = BeautifulSoup(wd.page_source, 'html5lib')
    out = []

    for img in soup.find_all('img'):
        if 'data-src' in img.attrs:
            imglnk = img['data-src']
            if imglnk.endswith('jpeg'):
                out.append(imglnk)

    return out

get_img('http://mp.weixin.qq.com/s?__biz=MzUxOTY3NDYzNQ==&mid=100000316&idx=1&sn=53b569177f7df04e3512f5bc14ed4e9c&scene=19#wechat_redirect')
```

```
['https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLPXKD4kN8ic4VTriapU6m62vOBNJAMOHSwT62GV7GLEaa4ca0ibUvalYUMg/640?wx_fmt=jpeg',
 'https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLP6utZFmMnll2M8UtCxoHn5rUV7Nlq3xEZGpXKHiaVyATcfnScmQhw7TA/640?wx_fmt=jpeg',
 'https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLPd6XAlAqYkrGKhZXdr4V7UWYN9van5vlz0RFhYLlWHICHFVatISaw4A/640?wx_fmt=jpeg',
 'https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLPmOpglZ4wh5ZVTd33RlbxribtQFNJNDRr6t4aUobLeRFd7S3cUpeDRaA/640?wx_fmt=jpeg']
```

很好。再验证几次之后就开始批量爬：

```python
import os
import time
import sys
import selenium
from selenium import webdriver
from bs4 import BeautifulSoup
wd = webdriver.Chrome()

def get_img(lnk):
    wd.get(lnk)
    soup = BeautifulSoup(wd.page_source, 'html5lib')
    out = []

    for img in soup.find_all('img'):
        if 'data-src' in img.attrs:
            imglnk = img['data-src']
            if imglnk.endswith('jpeg'):
                out.append(imglnk)

    return out

for [name, lnk] in dat:
    print(name)
    os.mkdir(name)
    os.chdir(name)
    imgs = get_img(lnk)

    for no, imgsrc in enumerate(imgs):
        cmd = 'wget "{}" -O {}.jpg'.format(imgsrc, no)
        print(cmd)
        os.system(cmd)
        time.sleep(1)

    os.chdir('..')
```

```
【TAB】 All Falls Down
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4aynvIQ3VB7pVsicpib3PpysXiaWdID3LUKGKVWcUJia0qpJrxqicdMKWsNw/640?wx_fmt=jpeg" -O 0.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4sicKIpmomgBGWibGsnG6pVEleXTwjXchzcHTBwvfflNExICCz02FWQOQ/640?wx_fmt=jpeg" -O 1.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4x9LbUxoZWAOX6G4qzf8vJKQ2iatiar08w8t3BP0PHciaiabDclnHUJ5zAg/640?wx_fmt=jpeg" -O 2.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4ibJoz5GEiaencoKhx8T8vbZ1EzLI2J9tQeRwEskABBbcTia6Sq3YcAnCQ/640?wx_fmt=jpeg" -O 3.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4VTrUiaxyefSkplJsyJrbnEEicRJIMQBGf6bSUUeTc2NjAqf2xicJk9T8w/640?wx_fmt=jpeg" -O 4.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4QEW5R0bNPEmibotz4qbH6icE6KCafMzjYZcpwHAQR9MHtYYJH8YpictYg/640?wx_fmt=jpeg" -O 5.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4dXFVoODNuakuu4JJVIySr1D4cTesXd5GR1HuaeSfOib09S8muBpdmBw/640?wx_fmt=jpeg" -O 6.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIdcbBTVV18OkCG4GH4QLx4A59RjLs1WEXFqLiakniaYtWbTfKK3hsiauhSpyv7XzicQna0oicshpFnRQg/640?wx_fmt=jpeg" -O 7.jpg
【TAB】 Alone
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLPdcA56Kd4siak1pOe15Kl7e5icq6pEXEiaiaW458yvq0O3iccCzIBEBic6LhQ/640?wx_fmt=jpeg" -O 0.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLPxK9Em7zwrI0hdFkmzkuORmeu0ZJxJ7iblybCKFUKV0ZPiclky9Psx1GA/640?wx_fmt=jpeg" -O 1.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLPHicYP9TnePGAfhmzELKicm14iaFyY8rzngocHNaox1aA9jbWQDfDmyaXA/640?wx_fmt=jpeg" -O 2.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgJXRpCDKSPxq2Ml0shgTQLP6yzTWBjqaXVGCGC6d5ticdScIHwvaFHqllUorviaxjqkoy2xgichdnafg/640?wx_fmt=jpeg" -O 3.jpg
【TAB】 Boulevard of Broken Dreams - Green Day
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIjvmGdYibhOibbNOTQiap5ykaqJ0oF8ev2icja0TVqxsot6QutqLDTqaDHPnHJzYy2wAwkd7nFDhzhWA/640?wx_fmt=jpeg" -O 0.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIjvmGdYibhOibbNOTQiap5ykat4nyW0v8BeLZfUT7SMic6blLkjjEDPvnuyqicguQYd1trYKOBvy2kfFg/640?wx_fmt=jpeg" -O 1.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIjvmGdYibhOibbNOTQiap5ykaFhLkSqylAoKPiaiaDEYpMr63kQusbxiaNyFr2Ivbicp6CNmiaDxx6cA8A7w/640?wx_fmt=jpeg" -O 2.jpg
【TAB】 Careless Whisper
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIjvmGdYibhOibbNOTQiap5ykaFgiaKUQjHTGxZvK5vyNHPVeOC6MGszHaRuDCCvuIzabUtjw7xrh33zQ/640?wx_fmt=jpeg" -O 0.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIjvmGdYibhOibbNOTQiap5ykamEayxOPm2FQvJhpfLbWThic40piaAcnG1O60AzibpKubn5PJOxmfiaibZow/640?wx_fmt=jpeg" -O 1.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIjvmGdYibhOibbNOTQiap5ykacSdd85aQicq5fYwVADYe32rEqia7gmAQB4UrLKM8sfDmD3k0JVY1dVjg/640?wx_fmt=jpeg" -O 2.jpg
【TAB】 Closer - The Chainsmokers ft. Hasley
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgKibnw9HVdMdjHh3YnsOk3TcDhAD7fWsTXSqQvQIQwWqjZB9c0C4O1htHeqK7LZzV6goPZcLKXu5nw/640?wx_fmt=jpeg" -O 0.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgKibnw9HVdMdjHh3YnsOk3Tcn4Zicupwia8t1tQeWAv8DtwkuicN81Q98YzzdhDiblcz1hjpbVN0U9fjZQ/640?wx_fmt=jpeg" -O 1.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgKibnw9HVdMdjHh3YnsOk3TcT3mn6ByMQrcQJwzvug1Rxu5VExqRVX9qVZHicNDo7vCyAQJg7pwRSog/640?wx_fmt=jpeg" -O 2.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgKibnw9HVdMdjHh3YnsOk3TcIUH8NE4U92g4Z2z57CkmbkNFLZ0Nd3ING2yib2sibL3pszTu8pcfLVNQ/640?wx_fmt=jpeg" -O 3.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgKibnw9HVdMdjHh3YnsOk3TcfUvUuzMDjoqibAhMlxFx4L47ZIicPOB5ibVSLxzIrIjN9pUass3oD7tVg/640?wx_fmt=jpeg" -O 4.jpg
【TAB】 Counting Stars - OneRepublic
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIjvmGdYibhOibbNOTQiap5yka8rPgQNmCVzkAic6NhBaUvRaa3JamgnomqctibNRLgLFZ7kUXz13z1P5A/640?wx_fmt=jpeg" -O 0.jpg
wget "https://mmbiz.qpic.cn/mmbiz_jpg/lvZ1HogYhgIjvmGdYibhOibbNOTQiap5yka1ShaNlXjPcKyfa78GTk4YWefJiaH3eTTN09EjN7p89USFaTSjQZdxuQ/640?wx_fmt=jpeg" -O 1.jpg
...
```

以上即可获取到乐谱。最后用imagestick的convert打包成pdf：

```shell
$ for i in *; do cd $i; convert `ls */*` $i.pdf; cd ..; done
```

注意不要把太多谱子都转换到单独一个PDF里面，不然会出错：

```
convert-im6.q16: DistributedPixelCache '127.0.0.1' @ error/distribute-cache.c/ConnectPixelCacheServer/244.
convert-im6.q16: cache resources exhausted `【TAB】.Revolution.-.Conny.Berghäll/7.jpg' @ error/cache.c/OpenPixelCache/3945.
convert-im6.q16: DistributedPixelCache '127.0.0.1' @ error/distribute-cache.c/ConnectPixelCacheServer/244.
convert-im6.q16: cache resources exhausted `【TAB】.Revolution.-.Conny.Berghäll/8.jpg' @ error/cache.c/OpenPixelCache/3945.
convert-im6.q16: DistributedPixelCache '127.0.0.1' @ error/distribute-cache.c/ConnectPixelCacheServer/244.
convert-im6.q16: cache resources exhausted `【TAB】.Revolution.-.Conny.Berghäll/9.jpg' @ error/cache.c/OpenPixelCache/3945.
...
```

**写在最后**

以上就是这次微信爬虫的经历。

需要掌握的基本技能：
1. Python的基本语法
2. Chrome、Selenium框架基本使用
3. 网络爬虫基本常识

以上代码不完善的点：
1. 这里应对反爬虫机制只是设定了爬取间隙时间`sleep(1)`，但是依然
很有可能被发现并封禁。需要处理wget被ban的情况；
2. 未完全实现自动爬取目录，仍然是半自动；
3. 微信公众号是WebApp，想爬就得运行JS。我这里用Selenium直接操作
浏览器，但是这就引入了很多依赖，比如X。之前ITRClub那个项目本来是放
在服务器上的，因为X的依赖搞得很麻烦。要研究有没有其他后端的选项（
重新编译浏览器，把GUI去掉？）。

说明：
1. 代码中有若干细节问题，欢迎指出讨论。
