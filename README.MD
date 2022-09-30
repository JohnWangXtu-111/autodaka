> 一个湘大校园自动打卡的程序。思路是很明确的，因为湘大校园健康打卡的本质就是一个表单的提交，这个表单数据在客户端和服务器之间的交互方式必然是通过`HTTP`。那么我的思路是，通过`python`里面爬虫相关的库来模拟这个交互过程。还有一种思路是使用点击精灵或者是`autojs`来模拟点击。

实现自动打卡，大致分为如下几步：

1. 抓包。这一步获取接口地址、鉴权方式、参数等。😃如果这个打卡是一个网页多好，直接`F12`抓包舒服得一批。试了`Burp`、`Fiddler`等抓包工具，都没有用......
2. 编写函数模拟请求
3. 定期执行第二步编写的函数
   1. 云函数实现
   2. `linux`定时服务实现

## 抓包

首先，你需要安装一个小黄鸟软件。

[百度网盘下载](https://pan.baidu.com/s/1Ov-OltymG1_pToNPOh9EuQ?pwd=t7f6)

打开安装好的软件，点击右下角那个蓝色的东东，安装一下CA证书。（出现的一切弹框都选确定就行了）

![image-20221001032555617](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221001032555617.png)

这个时候，退出黄色的这个软件，打开湘大校园，进入如下页面，这个时候先不提交（一定要先打开湘大校园，不然会被检测抓包，直接进不去软件）

![image-20221001032754815](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221001032754815.png)

然后回到黄色的软件，点击蓝色的按钮，开始抓包（同样，所有的弹框都选择确定）

![image-20221001032845177](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221001032845177.png)

进入湘大校园，提交健康打卡。然后，又回到小黄鸟，此时，抓包已经完成了。找到下图这一条数据，点击进去。现在需要拿到两个数据，一个是cookie，这个让服务端知道你是谁，一个是表单数据的json，这两个都要复制下来，替换到后面的代码中。

![image-20221001033053328](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221001033053328.png)

第一步，复制cookie

![image-20221001033306899](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221001033306899.png)

第二步，复制json

![image-20221001033342931](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20221001033342931.png)

## 代码

```python
from email import header
import requests
import json

def autoClockin():
    headers = {
        'Host': 'app.xiaoyuan.ccb.com',
        'Connection': 'keep-alive',
        'Content-Length': '1185',
        'Accept': 'application/json, text/plain, */*',
        'User-Agent': 'Mozilla/5.0 (Linux; Android 10; YAL-AL10 Build/HUAWEIYAL-AL10; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/86.0.4240.185 Mobile Safari/537.36jianronghuixue/1.2.2',
        'Content-Type': 'application/json;charset=UTF-8',
        'Origin': 'https://app.xiaoyuan.ccb.com',
        'X-Requested-With': 'com.ccb.xiangdaxiaoyuan',
        'Sec-Fetch-Site': 'same-origin',
        'Sec-Fetch-Mode': 'cors',
        'Sec-Fetch-Dest': 'empty',
        'Referer': 'https://app.xiaoyuan.ccb.com/LHECRESM/DZK/index2020110601.html',
        'Accept-Encoding': 'gzip, deflate',
        'Accept-Language': 'zh-CN,zh;q=0.9,en-CN;q=0.8,en-US;q=0.7,en;q=0.6',
        'Cookie': 'IN047=47001|YxgSs; IN031=31002|YymhM; sid=6055cf31-34f4-4887-89cd-8782b74aff9a; tgw_l7_route=d73149ab3f086e8d0cf04b5f79c1bd9e; IN001=1006|Yzc5O; IN010=10003|Yzc5O' #用你复制的cookie，替换掉冒号后面的内容
    }
    #用你复制的json，替换掉=后面的内容
    data = {
        "stuClass": "9999",
        "schoolId": "10530",
        "schoolName": "湘潭大学",
        "stId": "202005710517",
        "userId": "A3231002159826522854162504",
        "stName": "王石鑫",
        "locationAddr": "湖南省湘潭市雨湖区",
        "id": "E9E2C53F48539674E0539349FD0A3028",
        "departments": "",
        "isContact": "N",
        "isFever": "0",
        "isWuhan": "N",
        "nowArea": "湖南省湘潭市雨湖区",
        "familyaddress": "湘潭大学",
        "familyStatus": "0",
        "diagnosisTreatment": "",
        "nowStatus": "0",
        "healthStatus": "3",
        "isLevel": "N",
        "isbackLive": "N",
        "trafficTool": "",
        "backTrafficTool": "",
        "levelDate": "",
        "backtime": "",
        "arriveAddr": "",
        "trafficNo": "",
        "backTrafficNo": "",
        "professional": "",
        "personType": "",
        "personCategory": None,
        "temperature": 36.9,
        "remarks": None,
        "timeToLeaveHuBei": "",
        "dateOfDisengagement": "",
        "otherSymptoms": "",
        "nowStatusStartTime": "",
        "familyStatusStartTime": "",
        "feverStartTime": "",
        "coughStartTime": "",
        "fatigueStartTime": "",
        "diarrheaStartTime": "",
        "coldStartTime": "",
        "headacheStartTime": "",
        "noseStartTime": "",
        "runnyStartTime": "",
        "throatStartTime": "",
        "conjunctivaStartTime": "",
        "isAppearDiagnosis": "N",
        "isVaccinate": "1",
        "vaccineType": "2",
        "injectTimes": "3",
        "otherDesc": None,
        "isContactWithDiagnosis": "N",
        "isInSchool": "",
        "stMobile": "17377966179"
    }

    url = 'https://app.xiaoyuan.ccb.com/channelManage/outbreak/addOutbreak'
    res = requests.post(url=url, headers=headers, data=json.dumps(data))
    print(res.json())


autoClockin()
```

## 云函数或者定时任务

1. 腾讯云有对python云函数的支持，百度一下就能找到教程
2. 我用的是centos发行版linux，所以可以用crontab做定时任务

[云函数](https://blog.csdn.net/python_kele/article/details/119698462)

[定时任务](https://www.cnblogs.com/yangjisen/p/13171918.html)