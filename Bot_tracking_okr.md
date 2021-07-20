# Tạo Bot Report Tracking OKR Cho Thành Viên Trong Team Mỗi Tuần

![image](https://user-images.githubusercontent.com/68221273/126114466-5902b940-d3df-4ac7-8784-6fc1772f16c3.png)


Demo ở chế độ trực quan ( có thể setting chế độ console (headless) khi lên server) 


https://user-images.githubusercontent.com/68221273/126180723-002e2d6c-2f97-44eb-be9d-5d555273d830.mp4




Mỗi tuần trôi qua rất nhanh với nhiều đầu mục công việc cần phải hoàn thành, nhiều khi dặn lòng mình đừng quên UPDATE OKR nhé nhé (Ăn X đấy) nhưng mà "Đời không như mơ" tới khi bị ăn X rồi mới nhận ra là đã quá muộn màng rồi (yaoming).

Vậy nên Tool này viết ra nhằm mục đích tracking việc update OKR của bản thân cũng như ae trong TEAM luôn:
- Tool sẽ tư động crawl dữ liệu OKR của member trên https://goal.sun-asterisk.vn/ để lấy thông tin OKR của mọi người.
- Dùng chatwork API để gửi report tình trạng update OKR của mọi người trong TEAM.
- Sử dụng cron để send report định kỳ vào 15h thứ năm và thứ sáu hàng tuần.

## Kịch bản :
- Tự động login WSM sau đó Login Sgoal
- Vào URL tracking member group để crawl data trên SGOAL
- Làm sạch dữ liệu
- Gửi dữ liệu report tracking OKR vào box chatwork
- Quá trình này sẽ được call vào lúc 15h thứ năm và thứ sáu hàng tuần.

## Tech sử dụng :

- [Selenium] - Đây là 1 framework kiểm thử tự động ( Automation Tesing ) cho phép chúng ta điều khiển trình duyệt một cách tự động theo kịch bản có sẵn.  
- [Chatwork API] - Sử dụng Python và thư viện pychatwork để thao tác với API mà chatwork hỗ trợ.
- [Python] - Sử dụng ngôn ngữ lập trình Python

## Cài đặt Selenium trên Linux:
- Precondition :Máy đã cài python3 và pip3 nhé
- Step 1 - Install Selenium
```
pip3 install selenium
```
- Step 2 – Install Google Chrome
```
sudo curl -sS -o - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add
sudo echo "deb [arch=amd64]  http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google-chrome.list
sudo apt-get -y update
sudo apt-get -y install google-chrome-stable
```
- Step 3 – Install ChromeDriver

```
wget https://chromedriver.storage.googleapis.com/2.41/chromedriver_linux64.zip
unzip chromedriver_linux64.zip
You can find the latest ChromeDriver on its official download page. Now execute below commands to configure ChromeDriver on your system.

sudo mv chromedriver /usr/bin/chromedriver
sudo chown root:root /usr/bin/chromedriver
sudo chmod +x /usr/bin/chromedriver
```

## Viết Script:

- Chúng ta sẽ viết một script khởi tạo một profile ( hay cá nhân ) trên trình duyệt Google Chrome để lưu lại  phiên làm việc sau đỡ login lại thông tin nhé.
```
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
import requests
import os
import pychatwork as ch

def initDriverProfile(profile):
    CHROMEDRIVER_PATH = '/usr/bin/chromedriver'
    WINDOW_SIZE = "400,600"
    chrome_options = Options()
    #chrome_options.add_argument("--headless")
    chrome_options.add_argument("--window-size=%s" % WINDOW_SIZE)
    chrome_options.add_argument("user-data-dir=./.config/google-chrome/" + str(profile))
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument('disable-infobars')
    chrome_options.add_argument('--disable-gpu') if os.name == 'nt' else None
    chrome_options.add_argument("--verbose")
    chrome_options.add_argument("--no-default-browser-check")
    chrome_options.add_argument("--ignore-ssl-errors")
    chrome_options.add_argument("--allow-running-insecure-content")
    chrome_options.add_argument("--disable-web-security")
    chrome_options.add_argument("--disable-feature=IsolateOrigins,site-per-process")
    chrome_options.add_argument("--no-first-run")
    chrome_options.add_argument("--disable-notifications")
    chrome_options.add_argument("--disable-translate")
    chrome_options.add_argument("--ignore-certificate-error-spki-list")
    chrome_options.add_argument("--ignore-certificate-errors")
    chrome_options.add_argument("--disable-blink-features=AutomationControllered")
    chrome_options.add_experimental_option('useAutomationExtension', False)
    prefs = {"profile.default_content_setting_values.notifications": 2}
    chrome_options.add_experimental_option("prefs", prefs)
    chrome_options.add_argument("--start-maximized")  # open Browser in maximized mode
    chrome_options.add_argument("--disable-dev-shm-usage")  # overcome limited resource problems
    chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
    chrome_options.add_experimental_option("prefs", {"profile.managed_default_content_settings.images": 2})
    chrome_options.add_argument('disable-infobars')

    driver = webdriver.Chrome(executable_path=CHROMEDRIVER_PATH,
                              options=chrome_options
                              )
    return driver
```

Những chrome_options params trên mục đích là bypass một số trang web có thể chặn trình duyệt ở chế độ testing và một số setting cần thiết như disable ảnh  trên trang web load cho nhanh = ))

- Tiếp theo là Script Login Vào WSM Và Sgoal để lưu lại phiên làm việc:

```
def checkIsLogin(driver):
    try:
        isLogin = driver.find_elements_by_xpath('//a[contains(@href, "dashboard/user_timesheets")]')
        if len(isLogin):
            print("Logined!")
            return True

        return False
    except:
        print("check login err")
        return False


def loginWSM(driver, username, password):
    try:
        driver.get("https://wsm.sun-asterisk.vn/")
        time.sleep(2)
        if (checkIsLogin(driver)):
            return True
        driver.find_elements_by_class_name('btn-login')[0].click()
        time.sleep(1)
        driver.find_elements_by_id('user_email')[0].send_keys(username)
        time.sleep(1)
        driver.find_elements_by_id('user_password')[0].send_keys(password)
        time.sleep(1)
        driver.find_elements_by_class_name('span-remember')[0].click()
        time.sleep(2)
        driver.find_elements_by_id('wsm-login-button')[0].click()
        time.sleep(5)
        print("Login WSM Success!")

        return True
    except:
        print("Login Err!")
        return False


def loginOKR(driver):
    try:
        driver.get("https://goal.sun-asterisk.vn/login/framgia")
        time.sleep(3)
        print("Login SGoal Success")
        return True
    except:
        print("Login OKR Err")
        return False
```

- Viết một đoạn script để crawl data tình hình OKR member của team mình nào :
```
def trackingData(driver, groupId = "3079"):
    try:
        dataMyGroup = mappingChatWorkIdWithNameTraSuaGroup()
        driver.get("https://goal.sun-asterisk.vn/groups/" + str(groupId) + "#member-tab")
        elements = driver.find_elements_by_xpath('//*[@id="member-list-table"]/tbody/tr')
        time.sleep(2)
        print("This Group Have: " + str(len(elements)) + " Member")
        totalData = []
        for member in elements:
            time.sleep(1)
            dataRow = processData(member.text)
            if dataRow['name'] in dataMyGroup:
                dataRow['to'] = dataMyGroup[dataRow['name']]
                totalData.append(dataRow)

        return totalData
    except:
        print("Extrack data fail")

def processData(row):
    data = row.split("\n")
    return {
        "name" : data[1],
        "okr_format": data[-5],
        "okr_update": data[-4],
        "current_process": data[-3],
        "status": data[-2],
        "time": data[-1]
    }
```

- Tiếp theo Formart dữ liệu OKR và gửi lên Box Chatwork nhé:
```
def mappingChatWorkIdWithNameTraSuaGroup():
    return {
        'Phạm Xuân Nam': '[To:4945012]Pham Xuan Nam (97)',
        'Lê Thị Bé': '[To:3731269]Le Thi Be',
        'Nguyễn Phước Thắng': '[To:4808136]Nguyen Phuoc Thang (92)',
        'Nguyễn Đình Long': '[To:5044471]Nguyen Dinh Long',
        'Nguyễn Văn Ngọc B': '[To:2347431]Nguyen Van Ngoc B',
        'Nguyễn Văn Thành E': '[To:6044039]Nguyen Van Thanh E'
    }

def getCurrentTime():
    try:
        currentTime = requests.get('http://worldtimeapi.org/api/timezone/Asia/Bangkok')

        return currentTime.json()
    except:
        print("get current time err")

def sendDataToChatWorkRoom(dataAlert, room_id, tokenChatwork):
    client = ch.ChatworkClient(tokenChatwork)
    res = client.get_messages(room_id= room_id, force=True)
    if (res):
        dayTime = getCurrentTime()['datetime'].split("T")
        client.post_messages(room_id=room_id, message='[code]' + ' TRACKING OKR TOOL ' + dayTime[0] + " At: " + dayTime[1].split(".")[0] +  '[/code]')
        for data in dataAlert:
            time.sleep(1)
            client.post_messages(room_id= room_id, message=formatMessageSend(data))

def formatMessageSend(row, okrUpdateThresold = 100):
    messageAlert = ''
    if (int(str(row['okr_update']).replace("%", "")) < okrUpdateThresold):
        messageAlert = ' - Chưa Update OKR nè (tat)'

    return '[info]' + '[title]' + row['to'] + ' (*)' + row['status'] + '  ' + messageAlert +  '[/title]' + ' (*) OKR FORMAT: ' + row['okr_format'] + ' (h) OKR UPDATE: ' + row['okr_update'] + ' (F) CURENT PROCESS: ' + row['current_process']  + '[/info]'
    
roomId = ''
token = ''
username = ""
password = ""

def task():
    try:
        driver = initDriverProfile("tracking")
        if (loginWSM(driver, username, password)):
            loginOKR(driver)
            sendDataToChatWorkRoom(trackingData(driver), roomId, token)

        driver.close()
    except:
        driver.close()
        client = ch.ChatworkClient(token)
        client.post_messages(room_id=roomId, message='[To:5044471]Nguyen Dinh Long - Tool có vấn đề rồi a ey (F)')
        print("Tracking Fail")

task()
```


![image](https://user-images.githubusercontent.com/68221273/126142494-b45f638f-1695-4578-9ff6-c1037f8d8a30.png)



## Deploy lên server: ( nhớ bảo mật thông tin nhó, có thể Encrypt code để run cho an toàn ):
- Lên server thì cũng cài môi trường tương tự.
- Nhớ check Timezone cho đúng với giờ Việt Nam (setting bằng cách) :
```
timedatectl
#Set time zone (ex Bangkok)
sudo timedatectl list-timezones | grep Bangkok
sudo timedatectl set-timezone Asia/Bangkok
#Check xem đúng Timezone chưa
timedatectl
```

- Cài đặt NTP Server để đồng bộ thời gian:
```
sapt install chrony
vim /etc/chrony.conf	
#client port (udp)
acquisitionport 1123

sudo systemctl start chronyd
sudo systemctl enable chronyd
sudo systemctl status chronyd
chronyc sources
```
- Viết file sh (/root/cron.sh) để run script: 
```
#!/bin/bash

python3 /tool/okr/index.py
```

- Setting cron chạy 15h thứ năm và thứ sáu:
```
sudo crontab -e

// Add 2 line này
00 15 * * 4,5 sh /root/cron.sh >/dev/null 2>&1
```
- Nhớ bỏ comment dòng code này để chạy ở chế độ headless ( Không có giao diện nhé ) :
```
chrome_options.add_argument("--headless")
```

## The End:

Hi vọng không bị ăn (X) thêm lần nào nữa (yaoming) 



## Full Source: 

```
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import time
import requests
import os
import pychatwork as ch

def initDriverProfile(profile):
    CHROMEDRIVER_PATH = '/usr/bin/chromedriver'
    WINDOW_SIZE = "400,600"
    chrome_options = Options()
    chrome_options.add_argument("--headless")
    chrome_options.add_argument("--window-size=%s" % WINDOW_SIZE)
    chrome_options.add_argument("user-data-dir=/home/dinhlongit/.config/google-chrome/" + str(profile))
    chrome_options.add_argument('--no-sandbox')
    chrome_options.add_argument('disable-infobars')
    chrome_options.add_argument('--disable-gpu') if os.name == 'nt' else None
    chrome_options.add_argument("--verbose")
    chrome_options.add_argument("--no-default-browser-check")
    chrome_options.add_argument("--ignore-ssl-errors")
    chrome_options.add_argument("--allow-running-insecure-content")
    chrome_options.add_argument("--disable-web-security")
    chrome_options.add_argument("--disable-feature=IsolateOrigins,site-per-process")
    chrome_options.add_argument("--no-first-run")
    chrome_options.add_argument("--disable-notifications")
    chrome_options.add_argument("--disable-translate")
    chrome_options.add_argument("--ignore-certificate-error-spki-list")
    chrome_options.add_argument("--ignore-certificate-errors")
    chrome_options.add_argument("--disable-blink-features=AutomationControllered")
    chrome_options.add_experimental_option('useAutomationExtension', False)
    prefs = {"profile.default_content_setting_values.notifications": 2}
    chrome_options.add_experimental_option("prefs", prefs)
    chrome_options.add_argument("--start-maximized")  # open Browser in maximized mode
    chrome_options.add_argument("--disable-dev-shm-usage")  # overcome limited resource problems
    chrome_options.add_experimental_option("excludeSwitches", ["enable-automation"])
    chrome_options.add_experimental_option("prefs", {"profile.managed_default_content_settings.images": 2})
    chrome_options.add_argument('disable-infobars')

    driver = webdriver.Chrome(executable_path=CHROMEDRIVER_PATH,
                              options=chrome_options
                              )
    return driver

def checkIsLogin(driver):
    try:
        isLogin = driver.find_elements_by_xpath('//a[contains(@href, "dashboard/user_timesheets")]')
        if len(isLogin):
            print("Logined!")
            return True

        return False
    except:
        print("check login err")
        return False

def loginWSM(driver, username, password):
    try:
        driver.get("https://wsm.sun-asterisk.vn/")
        time.sleep(2)
        if (checkIsLogin(driver)):
            return True
        driver.find_elements_by_class_name('btn-login')[0].click()
        time.sleep(1)
        driver.find_elements_by_id('user_email')[0].send_keys(username)
        time.sleep(1)
        driver.find_elements_by_id('user_password')[0].send_keys(password)
        time.sleep(1)
        driver.find_elements_by_class_name('span-remember')[0].click()
        time.sleep(2)
        driver.find_elements_by_id('wsm-login-button')[0].click()
        time.sleep(5)
        print("Login WSM Success!")

        return True
    except:
        print("Login Err!")
        return False


def loginOKR(driver):
    try:
        driver.get("https://goal.sun-asterisk.vn/login/framgia")
        time.sleep(3)
        print("Login SGoal Success")
        return True
    except:
        print("Login OKR Err")
        return False

def trackingData(driver, groupId = "3079"):
    try:
        dataMyGroup = mappingChatWorkIdWithNameTraSuaGroup()
        driver.get("https://goal.sun-asterisk.vn/groups/" + str(groupId) + "#member-tab")
        elements = driver.find_elements_by_xpath('//*[@id="member-list-table"]/tbody/tr')
        time.sleep(2)
        print("This Group Have: " + str(len(elements)) + " Member")
        totalData = []
        for member in elements:
            time.sleep(1)
            dataRow = processData(member.text)
            if dataRow['name'] in dataMyGroup:
                dataRow['to'] = dataMyGroup[dataRow['name']]
                totalData.append(dataRow)

        return totalData
    except:
        print("Extrack data fail")

def processData(row):
    data = row.split("\n")
    return {
        "name" : data[1],
        "okr_format": data[-5],
        "okr_update": data[-4],
        "current_process": data[-3],
        "status": data[-2],
        "time": data[-1]
    }

def mappingChatWorkIdWithNameTraSuaGroup():
    return {
        'Phạm Xuân Nam': '[To:4945012]Pham Xuan Nam (97)',
        'Lê Thị Bé': '[To:3731269]Le Thi Be',
        'Nguyễn Phước Thắng': '[To:4808136]Nguyen Phuoc Thang (92)',
        'Nguyễn Đình Long': '[To:5044471]Nguyen Dinh Long',
        'Nguyễn Văn Ngọc B': '[To:2347431]Nguyen Van Ngoc B',
        'Nguyễn Văn Thành E': '[To:6044039]Nguyen Van Thanh E'
    }

def getCurrentTime():
    try:
        currentTime = requests.get('http://worldtimeapi.org/api/timezone/Asia/Bangkok')

        return currentTime.json()
    except:
        print("get current time err")

def sendDataToChatWorkRoom(dataAlert, room_id, tokenChatwork):
    client = ch.ChatworkClient(tokenChatwork)
    res = client.get_messages(room_id= room_id, force=True)
    if (res):
        dayTime = getCurrentTime()['datetime'].split("T")
        client.post_messages(room_id=room_id, message='[code]' + ' TRACKING OKR TOOL ' + dayTime[0] + " At: " + dayTime[1].split(".")[0] +  '[/code]')
        for data in dataAlert:
            time.sleep(1)
            client.post_messages(room_id= room_id, message=formatMessageSend(data))

def formatMessageSend(row, okrUpdateThresold = 100):
    messageAlert = ''
    if (int(str(row['okr_update']).replace("%", "")) < okrUpdateThresold):
        messageAlert = ' - Chưa Update OKR nè (tat)'

    return '[info]' + '[title]' + row['to'] + ' (*)' + row['status'] + '  ' + messageAlert +  '[/title]' + ' (*) OKR FORMAT: ' + row['okr_format'] + ' (h) OKR UPDATE: ' + row['okr_update'] + ' (F) CURENT PROCESS: ' + row['current_process']  + '[/info]'


roomId = '237172731'
token = 'xxxxxxxxx'
username = "xxxxxxxx@sun-asterisk.com"
password = "xxxxxxxxx"

def task():
    try:
        driver = initDriverProfile("tracking")
        if (loginWSM(driver, username, password)):
            loginOKR(driver)
            sendDataToChatWorkRoom(trackingData(driver), roomId, token)

        driver.close()
    except:
        driver.close()
        client = ch.ChatworkClient(token)
        client.post_messages(room_id=roomId, message='[To:5044471]Nguyen Dinh Long - Tool có vấn đề rồi a ey (F)')
        print("Tracking Fail")

task()
```
