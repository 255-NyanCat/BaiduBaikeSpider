```python
#-*- coding: utf-8 -*-
from bs4 import BeautifulSoup
import requests
import json
import time
import re
headers={'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36'}
usrin = input('请输入您要查询的东西： ')
pattern = re.compile(usrin)
baidubaikeurl = 'http://baike.baidu.com/search/none?word='+usrin+'&pn=0&rn=10&enc=utf8'
baidubaikeselector = '#body_wrapper > div.searchResult > dl'
print('正在向百度百科发送请求......')
gets = requests.get(baidubaikeurl,headers=headers)
print('正在建立Soup......')
rawsoup = BeautifulSoup(gets.text,'html.parser')
soups = rawsoup.select(baidubaikeselector)
all_a_tag = []
print('开始遍历结果......')
badres = 0
for s in soups:
    for item in s.find_all('a'):
        if re.search(usrin,str(item.get_text())) != None and re.search('；',str(item.get_text())) == None and re.search(';',str(item.get_text())) == None and re.search(',',str(item.get_text())) == None and re.search('去网页搜索',str(item.get_text())) == None:
            if re.search('https://baike.baidu.com',str(item['href']))  == None: 
                all_a_tag.append({"title":item.get_text(),"link":'https://baike.baidu.com'+str(item['href'])})
            else:
                all_a_tag.append({"title":item.get_text(),"link":item['href']})
        else:
            badres = badres + 1
            continue
print('遍历完毕，共有{btag}条结果不符合要求'.format(btag = badres))
f = open(r"tmp\res.json","w",encoding="utf-8")
print('正在写入中......')
f.write('''{
    "allresults":[
        ''')
for item in all_a_tag[0:len(all_a_tag) - 1]:
    json.dump(item,f,ensure_ascii=False)
    f.write(',\n        ')
json.dump(all_a_tag[-1],f,ensure_ascii=False)
f.write('''
    ]
}''')
print('写入完毕')
f.close()
```