---
title: 基于docker实现分布式UI自动化
date: 2022-02-14 11:38:21
tags: [Docker, Selenium-Grid]
top: true
---
# 基于docker实现分布式UI自动化

## 一、将对应的浏览器版本的[Chromedriver](https://registry.npmmirror.com/binary.html?path=chromedriver/)下载解压到/usr/local/bin/（此处为Mac地址）下，并在此目录下执行以下命令授信

```bash
xattr -d com.apple.quarantine chromedriver
```

## 二、docker配置

1.原理基于selenium-grid实现，以下为docker建立主从节点，从节点可以多个

主节点:

```bash
docker run --name=hub -p 5001:4444 -e GRID_TIMEOUT=0 -e GRID_THROW_ON_CAPABILITY_NOT_PRESENT=true -e GRID_NEW_SESSION_WAIT_TIMEOUT=-1 -e GRID_BROWSER_TIMEOUT=15000 -e GRID_TIMEOUT=30000 -e GRID_CLEAN_UP_CYCLE=30000 -d selenium/hub:3.7.1-beryllium
```
<!--more-->
从节点:

Mac下:

```bash
docker run --name=chrome -p 5902:5900  -e NODE_MAX_INSTANCES=6 -e NODE_MAX_SESSION=6 -e NODE_REGISTER_CYCLE=5000 -e DBUS_SESSION_BUS_ADDRESS=/dev/null -v */Users/xinximo/app/docker/shm:/dev/shm* --link hub -d selenium/node-chrome-debug:3.7.1-beryllium
```

Linux下:

```bash
docker run --name=chrome -p 5902:5900 -e NODE_MAX_INSTANCES=6 -e NODE_MAX_SESSION=6 -e NODE_REGISTER_CYCLE=5000 -e DBUS_SESSION_BUS_ADDRESS=/dev/null -v /dev/shm:/dev/shm --link hub -d selenium/node-chrome-debug:3.7.1-beryllium
```

## 三、Python代码配置

（1）初始基础类

```bash
import configparser
import os
import pytest
import yaml
from selenium import webdriver
from selenium.webdriver import DesiredCapabilities
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.remote.webdriver import WebDriver
from selenium.webdriver.support import expected_conditions
from selenium.webdriver.support.wait import WebDriverWait

class SaasBasePage:
    web_url = "https://xxx.cn"

    def __init__(self, base_driver=None):
        # 注解，不是赋值操作。用作ide的类型提示
        base_driver: WebDriver
        config = self.get_config()
        # 控制是否采用无界面形式运行自动化测试
        if base_driver is None:
            try:
                browser = os.environ["browser"]
            except KeyError:
                browser = None
                print('没有配置环境变量 browser, 按照默认有界面方式运行自动化测试')

            chrome_options = Options()
            if browser is not None and browser.lower() == 'no_gui':
                print('使用无界面方式运行')
                chrome_options.add_argument("--headless")
                self.driver = webdriver.Chrome(executable_path=config.get('driver', 'chrome_driver'),
                                               options=chrome_options)

            elif browser is not None and browser.lower() == 'chrome_remote':
                docker_remote = config.get('driver', 'chrome_remote')
                print(f'使用远程方式运行, remote url:{docker_remote}')
                self.driver = webdriver.Remote(command_executor=docker_remote,
                                               desired_capabilities=DesiredCapabilities.CHROME)

            elif browser is not None and browser.lower() == 'firefox_remote':
                docker_firefox_remote = config.get('driver', 'firefox_remote')
                print(f'使用远程方式运行, remote url:{docker_firefox_remote}')
                self.driver = webdriver.Remote(command_executor=docker_firefox_remote,
                                               desired_capabilities=DesiredCapabilities.FIREFOX)

            else:
                print('使用有界面Chrome浏览器运行')
                self.driver = webdriver.Chrome(executable_path=config.get('driver', 'chrome_driver'),
                                               options=chrome_options)
            self.driver.maximize_window()
            self.driver.get(self.web_url)
        else:
            self.driver = base_driver
        self.driver.implicitly_wait(10)

    def get_config(self):
        config = configparser.ConfigParser()
        config.read(os.path.join(os.environ['HOME'], 'iselenium.ini'))
        return config
```

（2）iselenium.ini配置

```bash
[driver]
chrome_driver=/usr/local/bin/chromedriver
chrome_remote="http://localhost:5001/wd/hub"
firefox_remote=<your selenium remote driver url>
```

## 四、观测执行结果

建议下载一个VNC Viewer，可以用该软件查看服务器上UI页面执行结果

默认密码:secret

并发执行:pytest -n 3 test_selenium.py
