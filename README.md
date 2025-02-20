# Scrapy with selenium
[![PyPI](https://img.shields.io/pypi/v/scrapy-selenium.svg)](https://pypi.python.org/pypi/scrapy-selenium) [![Build Status](https://travis-ci.org/clemfromspace/scrapy-selenium.svg?branch=master)](https://travis-ci.org/clemfromspace/scrapy-selenium) [![Test Coverage](https://api.codeclimate.com/v1/badges/5c737098dc38a835ff96/test_coverage)](https://codeclimate.com/github/clemfromspace/scrapy-selenium/test_coverage) [![Maintainability](https://api.codeclimate.com/v1/badges/5c737098dc38a835ff96/maintainability)](https://codeclimate.com/github/clemfromspace/scrapy-selenium/maintainability)

Scrapy middleware to handle javascript pages using selenium.

## Installation
```
$ pip install scrapy-selenium-customdriverinstance
```
You should use **python>=3.6**. 
You will also need one of the Selenium [compatible browsers](http://www.seleniumhq.org/about/platforms.jsp).

## Configuration
1. Add the browser to use, the path to the driver executable, and the arguments to pass to the executable to the scrapy settings:
```python
from shutil import which

SELENIUM_DRIVER_NAME = 'firefox'
SELENIUM_DRIVER_EXECUTABLE_PATH = which('geckodriver')
SELENIUM_DRIVER_ARGUMENTS=['-headless']  # '--headless' if using chrome instead of firefox
```

Optionally, set the path to the browser executable:
```python
SELENIUM_BROWSER_EXECUTABLE_PATH = which('firefox')
```

In order to use a remote Selenium driver, specify `SELENIUM_COMMAND_EXECUTOR` instead of `SELENIUM_DRIVER_EXECUTABLE_PATH`:
```python
SELENIUM_COMMAND_EXECUTOR = 'http://localhost:4444/wd/hub'
```
Alternatively, you can omit the `SELENIUM_DRIVER_NAME`, `SELENIUM_DRIVER_EXECUTABLE_PATH` and `SELENIUM_DRIVER_ARGUMENTS` settings and pass a Selenium Webdriver instance to each SeleniumRequest instance at the meta parameter 'driver' key.

2. Add the `SeleniumMiddleware` to the downloader middlewares:
```python
DOWNLOADER_MIDDLEWARES = {
    'scrapy_selenium_customdriverinstance.SeleniumMiddleware': 800
}
```
## Usage
Use the `scrapy_selenium_customdriverinstance.SeleniumRequest` instead of the scrapy built-in `Request` like below:
```python
from scrapy_selenium_customdriverinstance import SeleniumRequest

yield SeleniumRequest(url=url, callback=self.parse_result)
```
Optionally, you can provide a webdriver instance to the request, in the `meta` parameter with the key `driver`:
```python
from scrapy_selenium_customdriverinstance import SeleniumRequest

yield SeleniumRequest(url=url, callback=self.parse_result, meta={'driver': driver})
```
Either you have to provide a driver instance or the settings `SELENIUM_DRIVER_NAME`, `SELENIUM_DRIVER_EXECUTABLE_PATH` and `SELENIUM_DRIVER_ARGUMENTS` should be set, so the Selenium middleware instance contains a webdriver instance.
The request will be handled by Selenium, and the callback metod will be called, with a response object with a `meta` key named `driver`, containing the selenium driver with the request processed.
```python
def parse_result(self, response):
    print(response.request.meta['driver'].title)
```
For more information about the available driver methods and attributes, refer to the [selenium python documentation](http://selenium-python.readthedocs.io/api.html#module-selenium.webdriver.remote.webdriver)

The `selector` response attribute work as usual (but contains the html processed by the selenium driver).
```python
def parse_result(self, response):
    print(response.selector.xpath('//title/@text'))
```

### Additional arguments
The `scrapy_selenium_customdriverinstance.SeleniumRequest` accept 4 additional arguments:

#### `wait_time` / `wait_until`

When used, selenium will perform an [Explicit wait](http://selenium-python.readthedocs.io/waits.html#explicit-waits) before returning the response to the spider.
```python
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC

yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    wait_time=10,
    wait_until=EC.element_to_be_clickable((By.ID, 'someid'))
)
```

#### `screenshot`
When used, Selenium will take a screenshot of the page and the binary data of the .png captured will be added to the response `meta`:
```python
yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    screenshot=True
)
```
```python
def parse_result(self, response):
    with open('image.png', 'wb') as image_file:
        image_file.write(response.meta['screenshot'])
```

#### `script`
When used, Selenium will execute custom JavaScript code.
```python
yield SeleniumRequest(
    url=url,
    callback=self.parse_result,
    script='window.scrollTo(0, document.body.scrollHeight);',
)
```
