---
title: Amazon商品信息爬虫
date: 2019-01-29 18:00:40
tags:
recommended_posts: true

---

﻿### 1.环境配置

 - Ubuntu16.04 
 - python3 
 - Chrome以及与之对应的chromedriver
 - Selenium

版本参考：[chromedriver下载地址](http://chromedriver.chromium.org/downloads)
### 2.代码
1.在selenium中使用代理，在Ubuntu中设置sock5代理端口
```python
from selenium.webdriver.chrome.options import Options
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--proxy-server=socks5://127.0.0.1:1080')
```
2.爬虫类<!--more-->

```python
# coding: utf-8
"""
"""
from __future__ import print_function, division

import time
import sys
import pandas as pd
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
import random
chrome_options = webdriver.ChromeOptions()
chrome_options.add_argument('--proxy-server=socks5://127.0.0.1:1080')
#reload(sys)
#sys.setdefaultencoding("utf-8")


class Spider(object):
    def __init__(self):
        self.options = Options()
        self.options.add_argument('--headless')
        #q1(2)
        self.driver = webdriver.Chrome(chrome_options=chrome_options,executable_path="chromedriver")
        self.driver.implicitly_wait(10)
        self.verificationErrors = []
        self.accept_next_alert = True
        self.url = "https://www.amazon.com/s?ie=UTF8&page=1&rh=n%3A2407749011%2Ck%3AApple&page=1"
        self.driver.get(self.url)
        self.data = []
        #q1(1)
        self.detail_urls = []
    #qi(5)
    def fanye(self):
        next_page_button = self.driver.find_element_by_xpath("//a[@id='pagnNextLink']")
        next_page_button.send_keys("\n")
    #q1(3)
    def get_single_page(self):
        lists = self.driver.find_elements_by_css_selector("li[id^='result_']")
        
        #print(len(lists),lists)
        for li_element in lists[0:20]:

            manufacturer, screen_size, product_dimensions_text, rating_content,\
                reviews_content, more_offer_text, offer_Price, display_type_text, product_detail = [None] * 9
            try:
                manufacturer = li_element.find_elements_by_xpath(
                    ".//div[@class='a-fixed-left-grid-col a-col-right']/div/div")[1].text
                if "by Apple" not in manufacturer:
                    continue
            except Exception as e:
                manufacturer = None
                #print("ERROR: find manufacturer has something error: ", e)
            try:
                url_detail_in_div = li_element.find_element_by_xpath(
                    ".//div[@class='a-fixed-left-grid-col a-col-right']/div/div/a[contains(@title, 'Apple')]")
                detail_url_ = url_detail_in_div.get_attribute("href")
                product_detail = url_tile = url_detail_in_div.get_attribute("title")
                if "slredirect" in detail_url_:
                    continue
                # print(detail_url_)
                self.detail_urls.append(detail_url_)
                js = 'window.open("{}");'.format(detail_url_)
                self.driver.execute_script(js)
                handles = self.driver.window_handles
                for handle in handles:
                    if handle != self.driver.current_window_handle:
                        self.driver.switch_to_window(handle)
                        break
                try:
                    table1 = self.driver.find_elements_by_xpath("//table[@id='HLCXComparisonTable']/tbody/tr")
                    for tr in table1:
                        html1 = tr.get_attribute("innerHTML")
                        if "Screen Size" in html1:
                            td_inches = tr.find_elements_by_xpath(".//td")[0]
                            screen_size = td_inches.text
                            # print(screen_size)
                            break
                except Exception as e:
                    pass
                    #print("ERROR: find screen_size has something error: ", e)
                #q1(4)
                try:
                    table2 = self.driver.find_elements_by_xpath("//table[@id='productDetails_detailBullets_sections1']/tbody/tr")
                    for tr in table2:
                        html2 = tr.get_attribute("innerHTML")
                        if "Product Dimensions" in html2:
                            product_dimensions_html = tr.find_elements_by_xpath(".//td")[0]
                            product_dimensions_text = product_dimensions_html.text
                            # print(product_dimensions_text)
                            continue
                        if "Customer Reviews" in html2:
                            rating_html = tr.find_element_by_xpath(".//td")
                            rating_text = rating_html.text
                            reviews_text, rating_content = rating_text.split("\n")
                            reviews_content = reviews_text.split(" ")[0]
                            # print(rating_content)
                            # print(reviews_content)
                            continue
                except Exception as e:
                    pass
                    #print("ERROR: find product_dimensions_text rating_content reviews_content, some problems ",e)
                self.driver.close()
                self.driver.switch_to_window(handles[0])
                # print(product_detail)
                # print("\n")
            except Exception as e:
                pass
                #print("ERROR: get item detail url pages has problems", e)
            # break
        
            try:
                more_offer_html = li_element.find_element_by_css_selector("a[href*='offer-listing']")
                more_offer_text = more_offer_html.text
                # print("more_offer_text is: ", more_offer_text)
                # print(more_offer_html.get_attribute('innerHTML'))
            except Exception as e:
                pass
                #print("ERROR: more_offer has problem: ", e)
            try:
                offer_price_html = li_element.find_element_by_xpath(".//span[@class='sx-price sx-price-large']")
                offer_Price = offer_price_html.text
                # print(offer_Price)
            except Exception as e:
                pass
                #print("ERROR: find offer_Price has problem: ", e)
            try:
                display_type_html = li_element.find_elements_by_xpath(
                    ".//dl[@class='a-definition-list a-vertical s-padding-left-base']/li/span[@class='a-list-item a-size-small a-color-secondary']")
                for _span in display_type_html:
                    if "Display Type:" in _span.text:
                        display_type_html = _span.text
                        display_type_text = display_type_html.split(":")[1]
                        break
            except Exception as e:
                pass
                #print("ERROR: find display_type has problem: ", e)
                # print("\n")
                # break
    
            _tmp_dt = [manufacturer, screen_size, product_dimensions_text, rating_content, reviews_content, more_offer_text,
                  offer_Price, display_type_text, product_detail, None, None]
            print(_tmp_dt)
            self.data.append(_tmp_dt)
            time.sleep(random.randint(0,2))

    def main(self):
        i=0
        while True and i<=4:
            self.get_single_page()
            try:
                i=i+1
                self.fanye()
            except:
                break
        return self.data,self.detail_urls


a = Spider()
dt, url_lists = a.main()
print(url_lists)

col_name = ["Manufacturer", "Display_Size", "Display_Dimensions", "Rating", "Number_of_Review", "More_Offer", "Offer_Price", "Display_Type", "Product_detail", "Special_Feature", "Special_Offer"]
df = pd.DataFrame(dt,columns=col_name)
#df=df.rename(columns=col_name)
print(df)

```

