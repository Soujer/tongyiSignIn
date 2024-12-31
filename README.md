![](https://cdn.java.pet/img/2024/12/1734491575-f30cbdae6f604a0.png)
## 前言
晓杰平时也用阿里这个AI，看到推送这个活动就上了，可以撸实物！就写了个脚本自动每天签到，需要的自取。没有青龙面板的可以直接安装python环境双击运行即可。（删掉注释的青龙面板通知代码）
![](https://cdn.java.pet/img/2024/12/1734491728-f10af623652ef68.jpg)
活动地址： https://t.aliyun.com/U/eNiPVf （官方链接无任何推广）
![](https://cdn.java.pet/img/2024/12/1734492297-ef4c4f0c58381f4.jpg)
能不能撸到实物看大家运气啦！
## 活动图片
![](https://cdn.java.pet/img/2024/12/1734500022-ce2aca57377227b.jpg)
## 步骤
1.先下载APP随便发个消息给AI，抓包找到这个请求包：
```
qianwen.biz.aliyun.com/dialog/conversation
```
（抓包工具：[https://www.java.pet/code/3853.html](https://www.java.pet/code/3853.html "https://www.java.pet/code/3853.html")）
2.然后拿去下列配置进行配置即可，还有日期改成你要签到的起始日期
```
#公众号推送token 关注公众号：软件接口平台 回复 token 获取
token="xxxxxxx"

requestId="xxxxxx"
sessionId="xxxxxx"
parentMsgId="xxxxxx"
agentId="xxxxxx"
cookie="xxxxxx"
DeviceId="xxxxxx"
UmidToken="xxxxxx"
start_date = datetime.date(2023, 12, 17)
```
## 代码
```python
import requests
import json
from urllib.parse import unquote, urlparse, parse_qs
import datetime
#⬇️⬇️配置区⬇️⬇️⬇️
#公众号推送token 关注公众号：软件接口平台 回复 token 获取
token="xxxxxxx"

requestId="xxxxxx"
sessionId="xxxxxx"
parentMsgId="xxxxxx"
agentId="xxxxxx"
cookie="xxxxxx"
DeviceId="xxxxxx"
UmidToken="xxxxxx"
# 设置起始日期（例如2023年12月17日）这里写你开始第一天签到的时间 如果是当天你就写当天的日期 保存后固定不用再次修改
start_date = datetime.date(2023, 12, 17)
#⬆️⬆️配置区⬆️⬆️⬆️
def send_message(title, content):
    params = {
        'token': f"{token}",
        'title': title,
        'content': content
    }
    url = 'https://hayo.svip8.vip/send'
    resp = requests.get(url, params=params)
    print(f"Request URL: {resp.url}")
    print(f"Response Status Code: {resp.status_code}")
    try:
        response_json = resp.json()
        message = response_json.get('message')
        print(f"公众号推送结果: {message}")
        return message
    except ValueError:
        print("公众号通知失败")
        return None


url = "https://qianwen.biz.aliyun.com/growth/activity/reading/challenge/query"
headers = {
    "X-App-Version": "3.22.0",
    "Connection": "keep-alive",
    "Accept": "*/*",
    "X-UmidToken":f"{UmidToken}",
    "Accept-Encoding": "br;q=1.0, gzip;q=0.9, deflate;q=0.8",
    "Accept-Language": "zh-Hans-JP;q=1.0",
    "Content-Type": "application/json",
    "User-Agent": "TONGYI/3.22.0 (com.aliyun.ios.tongyi; build:42271522; iOS 16.6.0) Alamofire/5.8.0 DeviceModel/iPhone14,5 AppType/Release",
    "X-DeviceId": f"{DeviceId}",
    "X-NetworkQuality": "wifi, rtt: 3935ms",
    "Referer": "https://qianwen-mobile.aliyun.com/",
    "X-LoginType": "havana",
    "Cookie": f"{cookie}",
    "X-Platform": "tongyi"
}
data = {}

response = requests.post(url, headers=headers, json=data)

print(response.status_code)
print(response.json())

if response.status_code == 200:
    response_data = response.json()
    chapters = []
    for sub_task in response_data["data"]["task"]["subUserTasks"]:
        if "taskGuidance" in sub_task and "appJumpUrl" in sub_task["taskGuidance"]:
            print("Decoded prompt data:", sub_task["taskGuidance"]["appJumpUrl"])

            prompt_index = sub_task["taskGuidance"]["appJumpUrl"].find("&prompt=")
            if prompt_index != -1:
                prompt_data = sub_task["taskGuidance"]["appJumpUrl"][prompt_index + len("&prompt="):]
                decoded_prompt_data = unquote(prompt_data)
                print("Decoded prompt data:", decoded_prompt_data)
                chapters.append(decoded_prompt_data)
                print("Extracted appJumpUrls:", decoded_prompt_data)
            else:
                print("prompt= not found in the URL")
            
else:
    print("Failed to retrieve data, status code:", response.status_code)
current_date = datetime.date.today()
days_passed = (current_date - start_date).days
print(f"Request for {days_passed} {current_date} {start_date} ")
if days_passed < 0:
    send_message('通义签到结果', "起始时间大于当前时间不执行签到")
    #没有青龙面板可以删掉下面代码
    QLAPI.notify('通义签到结果', "起始时间大于当前时间不执行签到")

    print("起始时间大于当前时间不执行签到")
elif (days_passed >= len(chapters) and days_passed<300):
    send_message('通义签到结果', "签到完成啦")
    #没有青龙面板可以删掉下面代码
    QLAPI.notify('通义签到结果', "签到完成啦")

    print("签到完成啦")
else:
    if days_passed>300:
        days_passed=days_passed-366
    chapter_to_request = chapters[days_passed]
    print(f"Request for {chapter_to_request} ")
    
    # 构建原始请求数据
    original_request = {
        "model": "",
        "action": "next",
        "mode": "chat",
        "userAction": "probe",
        "requestId": f"{requestId}",
        "sessionId": f"{sessionId}",
        "parentMsgId": f"{parentMsgId}",
        "sessionType": "text_chat",
        "contents": [
            {
                "role": "user",
                "contentType": "text",
                "content": f"请帮我精读{chapter_to_request}",
                "__inputMode": "keyboard"
            }
        ],
        "params": {
            "agentId": f"{agentId}"
        }
    }

    # 发送初始POST请求
    url = "http://qianwen.biz.aliyun.com/dialog/conversation"
    headers = {
        "Host": "qianwen.biz.aliyun.com",
        "X-NetworkQuality": "wifi, rtt: 482ms",
        "Connection": "keep-alive",
        "Accept": "text/event-stream",
        "X-UmidToken": f"{UmidToken}",
        "Accept-Encoding": "br;q=1.0, gzip;q=0.9, deflate;q=0.8",
        "Accept-Language": "zh-Hans-JP;q=1.0",
        "Cache-Control": "no-cache",
        "Content-Type": "application/json",
        "Content-Length": str(len(json.dumps(original_request))),
        "X-DeviceId":f"{DeviceId}",
        "User-Agent": "TONGYI/3.22.0 (com.aliyun.ios.tongyi; build:42271522; iOS 16.6.0) Alamofire/5.8.0 DeviceModel/iPhone14,5 AppType/Release",
        "X-App-Version": "3.22.0",
        "Referer": "https://qianwen-mobile.aliyun.com/",
        "X-LoginType": "havana",
        "Cookie": f"{cookie}",
        "X-Platform": "tongyi"
    }

    response = requests.post(url, headers=headers, data=json.dumps(original_request))
    print(f"Request for {chapter_to_request} sent with status code: {response.status_code}")
    send_message('通义签到结果', f"{response.status_code}")
     #没有青龙面板可以删掉下面代码
    QLAPI.notify('通义签到结果', f"{response.status_code}")


```
## 配置定时签到（没青龙面板的忽略此步骤）
![](https://cdn.java.pet/img/2024/12/1734493292-b409a6f37d2660a.jpg)
## 示例图片
![](https://cdn.java.pet/img/2024/12/1734493065-0ee8036e8019eee.png)
## 本文作者
**Soujer**
