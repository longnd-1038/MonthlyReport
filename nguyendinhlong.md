# Tạo Bot Report Tracking OKR Cho Thành Viên Trong Team Mỗi Tuần

![image](https://user-images.githubusercontent.com/68221273/126114466-5902b940-d3df-4ac7-8784-6fc1772f16c3.png)

Mỗi tuần trôi qua rất nhanh với nhiều đầu mục công việc cần phải hoàn thành, nhiều khi dặn lòng mình đừng quên UPDATE OKR nhé nhé (Ăn X đấy) nhưng mà "Đời không như mơ" tới khi bị ăn X rồi mới nhận ra là đã quá muộn màng rồi (yaoming).

Vậy nên mục đích của Tool này viết ra nhằm mục đích tracking việc update OKR của bản thân cũng như ae trong TEAM luôn:
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

- [Selenium] - Đây là 1 framework kiểm thử tự độn( Automation Tesing ) cho phép chúng ta điều khiển trình duyệt một cách tự động theo kịch bản có sẵn.  
- [Chatwork API] - Sử dụng Python và thư viện pychatwork để thao tác với API mà chatwork hỗ trợ.
- [Python] - Sử dụng ngôn ngữ lập trình Python

## Cài đặt Selenium trên Linux:
- Precondition :Máy đã cài python và pip nhé
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


```
sh
cd dillinger
npm i
node app
```

For production environments...

```sh
npm install --production
NODE_ENV=production node app
```

## Plugins

Dillinger is currently extended with the following plugins.
Instructions on how to use them in your own application are linked below.

| Plugin | README |
| ------ | ------ |
| Dropbox | [plugins/dropbox/README.md][PlDb] |
| GitHub | [plugins/github/README.md][PlGh] |
| Google Drive | [plugins/googledrive/README.md][PlGd] |
| OneDrive | [plugins/onedrive/README.md][PlOd] |
| Medium | [plugins/medium/README.md][PlMe] |
| Google Analytics | [plugins/googleanalytics/README.md][PlGa] |

## Development

Want to contribute? Great!

Dillinger uses Gulp + Webpack for fast developing.
Make a change in your file and instantaneously see your updates!

Open your favorite Terminal and run these commands.

First Tab:

```sh
node app
```

Second Tab:

```sh
gulp watch
```

(optional) Third:

```sh
karma test
```

#### Building for source

For production release:

```sh
gulp build --prod
```

Generating pre-built zip archives for distribution:

```sh
gulp build dist --prod
```

## Docker

Dillinger is very easy to install and deploy in a Docker container.

By default, the Docker will expose port 8080, so change this within the
Dockerfile if necessary. When ready, simply use the Dockerfile to
build the image.

```sh
cd dillinger
docker build -t <youruser>/dillinger:${package.json.version} .
```

This will create the dillinger image and pull in the necessary dependencies.
Be sure to swap out `${package.json.version}` with the actual
version of Dillinger.

Once done, run the Docker image and map the port to whatever you wish on
your host. In this example, we simply map port 8000 of the host to
port 8080 of the Docker (or whatever port was exposed in the Dockerfile):

```sh
docker run -d -p 8000:8080 --restart=always --cap-add=SYS_ADMIN --name=dillinger <youruser>/dillinger:${package.json.version}
```

> Note: `--capt-add=SYS-ADMIN` is required for PDF rendering.

Verify the deployment by navigating to your server address in
your preferred browser.

```sh
127.0.0.1:8000
```
