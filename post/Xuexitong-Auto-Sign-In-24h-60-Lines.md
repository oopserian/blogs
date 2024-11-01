---

title: 60行代码后台24小时《学习通自动签到》

description: 几年前刚学爬虫发的酷安帖子

cover: /images/2020-05-08/IMG_1027.JPG

date: 2020-05-08

tags: [爬虫,Python,Fiddler]

pin: false

hide: false

---
```
抓包工具:Fiddler
语言:Python 库:requests
测试手机:夜神模拟器
运行方式:函数云
```

第一次发长文， #第一行代码# 本人只是个学习一个月的萌新，所有知识源于GitHub，百度，b站，学校，有很多地方不懂，不要痛击我一个萌新🥹本教程只是交流学习，希望有大佬指点错误的地方

## 首先
我们自己在学习通里创建一个课程，方便测试。自己发布一个签到信息，然后抓包自己签到的数据
![签到成功的数据](/images/2020-05-08/IMG_1011.JPG)
有右边那些数据，一个一个看，找到了一个返回success的，其他的只是页面源代码，还有一些我也不知道
![返回success数据](/images/2020-05-08/IMG_1012.JPG)
复制它的URL在浏览器里打开，发现他有这么长的地址，经过删减，有些不需要的
```
https://mobilelearn.chaoxing.com/pptSign/stuSignajax?
activeld=234452314
&uid=79896801&clientip=&useragent=&latitude
=-1&longitude=-1&appType=15&fid=3028
&name=%5%88%98%4%BF%8А%Е9%BE%99
```
删减过后，发现只需要这么一点也能打
```
https://mobilelearn.chaoxing.com/pptSign/stuSignajax?
activeld=234452314
```
![](/images/2020-05-08/IMG_1014.JPG)
后面那串数字就是我们需要的了，activeID翻译过来不就是活动ID吗，我们只需要找到活动ID就好了啊，返回到活动的列表
![活动列表](/images/2020-05-08/IMG_1015.JPG)
抓活动列表的包，找这里面，看哪个里面有活动ID就好了
```
https://mobilelearn.chaoxing.com/ppt/activeAPI/taskactivelist?
courseld=212107435&classld=26413158&uid=79896801&cpi=65351
848&token=4faa8662c59590c6f43ae9fe5b002b42&_time=158892641
7382&inf_enc=750127bd4eaefc6e6473e6c6f6deeeb5
```
发现这个json文件里面有很多东西，(一般都是json文件）发现有个activelist(活动列表)，里面刚好有active的ID，234452314

![活动列表json](/images/2020-05-08/IMG_1017.JPG)

复制刚刚那个活动API接口，打开网页，删减一些也能打开
```
https://mobilelearn.chaoxing.com/ppt/activeAPI/taskactivelist?
courseld=204220874&classld=8371954
```
删减成上面那样就可以了，最后那两个数据是不能删掉的，删了其中一个，就打不开了，那就是需要后面那两个数据了，一个courseID和classID。
返回到课程列表里去找找有没有这两个数据
![Node简介](/images/2020-05-08/IMG_1019.JPG)
抓包发现就几个，一个一个看了后，发现只有其中一个json文件里有我们想要的数据，对应的ID码，刚好和我们的courseID和classID。然后复制URL出来
```
https://mooc1-api.chaoxing.com/mycourse/backclazzdata?
```
发现只有这么长
![Node简介](/images/2020-05-08/IMG_1021.JPG)
浏览器打开看到课程所有信息了，这个课程URL不需要其他参数，那就从这个URL开始吧。
所有想要的参数都有了，就开始码代码吧。
```
课程列表：
https://mooc1-api.chaoxing.com/mycourse/backclazzdata?
获取courseld=******
获取classld=*******

课程任务列表：
https://mobilelearn.chaoxing.com/ppt/activeAPI/taskactivelist?
courseld=204220874&classld=8371954
获取activeld=*******

普通签到成功：
https://mobilelearn.chaoxing.com/pptSign/stuSignajax?
activeld=220960453
```

``` Python
import requests
import json

def main_handler(event, context):
    url = 'http://mooc1-api.chaoxing.com/mycourse/backclazzdata'
    address = "..."  # 用户的地址信息（隐去部分）
    headers = {
        'Cookie': 'uname=2018D2160128; lv=3; fid=3028; _uid=79896801; uf=...',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/...'
    }

    req = requests.get(url, headers=headers)
    data = bytes(req.text, req.encoding).decode("utf-8", "ignore")
    class_id = json.loads(data)
    content = class_id["channelList"]
    class_name = []
    class_id = []
    for i in content:
        try:
            a = (i['content']['course']['data'][0]['name'])
            b = (i['content']['id'])
            class_id.append(b)
            class_name.append(a)
        except Exception:
            pass

    class_course = []
    for j in content:
        try:
            course_id = (j['content']['course']['data'][0]['id'])
            class_course.append(course_id)
        except Exception:
            pass
```
这里要弄一个检测签到的，在活动列表里有两个参数，activetype(活动类型)为2，就是签到活动。status(状态)为1就是正在进行的，状态为2就是已经结束的了
![Node简介](/images/2020-05-08/IMG_1017.JPG)

写一个判断状态方法就好了，状态为1就执行签到，为2就终止就好了
``` Python
for x in range(0, (len(class_id))):
    class_Name = class_name[x]
    courseID = class_course[x]
    class_url = 'https://mobilelearn.chaoxing.com/ppt/activeAPI/taskactivelist?courseId=%s&classId=%s' % courseID
    req = requests.get(class_url, headers=headers)
    data = bytes(req.text, req.encoding).decode("utf-8", "ignore")
    active_all = json.loads(data)
    content = active_all['activeList']
    for y in content:
        try:
            active_type = (y['activeType'])  # 活动类型
            if active_type == 2:
                status = (y['status'])  # 活动状态,1为正在进行,2为已结束
                if status == 1:
                    active_id = (y['id'])
                    active_url = 'https://mobilelearn.chaoxing.com/pptSign/stuSignajax?address=%s&activeId=%s' % (address, active_id)
                    req = requests.get(active_url, headers=headers)
                    print(class_Name)
                    print('已签到')
                    print(active_url)
                else:
                    print('没有')
        except Exception:
            pass
```
然后自己发布一个签到，运行代码，ok～
带进函数云～
![Node简介](/images/2020-05-08/IMG_1026.JPG)
![Node简介](/images/2020-05-08/IMG_1027.JPG)
触发方式设置为每分钟运行一下。
再发布一个签到试一下
![Node简介](/images/2020-05-08/IMG_1028.JPG)
成功！！后台一关，电脑一关，一躺，一睡[酷][酷]，再也不用担心我忘记签到了[睡][睡][睡]24小时后台识别有没有签到，一有签到就自动给我签了[亲亲][亲亲]

唯一缺点就是，三天左右需要换cookies，不能做到一直挂后台，要换cookie，有没有哪位大佬指点[流泪]自动获取cookie，或者自动登录，selenium云函数好像不行。