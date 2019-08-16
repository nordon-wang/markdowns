## Selenium 简介

百度百科介绍：

> Selenium [1]  是一个用于Web应用程序测试的工具。Selenium测试直接运行在浏览器中，就像真正的用户在操作一样。支持的浏览器包括IE（7, 8, 9, 10, 11），[Mozilla Firefox](https://baike.baidu.com/item/Mozilla Firefox/3504923)，Safari，Google Chrome，Opera等。这个工具的主要功能包括：测试与浏览器的兼容性——测试你的应用程序看是否能够很好得工作在不同浏览器和操作系统之上。测试系统功能——创建回归测试检验软件功能和用户需求。支持自动录制动作和自动生成 .Net、Java、Perl等不同语言的测试脚本。

## 使用流程

1. 根据平台下载需要的webdrive

| Browser           | Component                                                    |
| :---------------- | :----------------------------------------------------------- |
| Chrome            | [chromedriver(.exe)](http://chromedriver.storage.googleapis.com/index.html) |
| Internet Explorer | [IEDriverServer.exe](http://selenium-release.storage.googleapis.com/index.html) |
| Edge              | [MicrosoftWebDriver.msi](http://go.microsoft.com/fwlink/?LinkId=619687) |
| Firefox           | [geckodriver(.exe)](https://github.com/mozilla/geckodriver/releases/) |
| Safari            | [safaridriver](https://developer.apple.com/library/prerelease/content/releasenotes/General/WhatsNewInSafari/Articles/Safari_10_0.html#//apple_ref/doc/uid/TP40014305-CH11-DontLinkElementID_28) |

> 根据自己的环境进行下载，将下载好的**压缩包解压**到项目的根目录**不需要安装**,各个浏览器的版本和dirver的版本的选择需要相近，不能盲目选择最新的版本，否则会出现意想不到的bug，建议最好将浏览器升级至最新的稳定版本并选择对应的包。

2. 下载依赖包

```
npm install selenium-webdriver
```

3. 玩一下官方demo, 做适当的代码修改并根据代码进行注释

```
// 1. 引入selenium-webdriver包，解构需要的对象和方法
const {Builder, By, Key, until} = require('selenium-webdriver');
 
// 2. 将需要的代码包在一个自执行函数中
(async function example() {
	// 实例化 driver 对象， chrome 代表使用的浏览器
  let driver = await new Builder().forBrowser('chrome').build();
  
  try {
  	// 需要打开的网站地址
    await driver.get('https://www.baidu.com/');
    
    // Key.RETURN enter回车
    // By.id('id') 百度查询滴输入内容
    // 找到元素 向里面发送一个关键字并按回撤
		await driver.findElement(By.id('kw')).sendKeys('酒店', Key.RETURN);
		
		// 等1秒之后，验证是否搜索成功
    // await driver.wait(until.titleIs('酒店_百度搜索'), 1000);
  } finally {
  	// 关闭浏览器
    // await driver.quit();
  }
})();

```

## 爬取掘金小册数据

- 实现的功能
  - 自动打开掘金页面的首页
  - 自动点击**小册**、**前端**进行路由切换
  - 将前端的全部小册数据爬取
- 注意点
  - 由于selenium内部都是基于promise进行的封装，所有的方法调用其实返回的都是一个promise对象，因此会大量的使用async语法

### 自动打开掘金页面的首页

> 一句话搞定自动打开掘金首页

```
// 自动打开掘金
await driver.get('https://juejin.im/timeline');
```

### 自动点击进行路由跳转

1. 在浏览器中，查看页面的布局结构，找到**小册**的位置，若是使用jq进行dom选择，则是 **$('.main-header-box .nav-item:nth-of-type(4)')**

![image-20190813152416769](/Users/nordon.wang/Desktop/self/markdowns/node/爬虫/image-20190813152416769.png)

2. selenium中的**By**拥有很多的选择，其使用规则和jq非常相似，**By.css('.main-header-box .nav-item:nth-of-type(4)')**便可以找到对应的元素，调用click事件就可以模拟自动点击

```
// 点击小册子的navBar 切换路由到小册
await driver
.findElement(By.css('.main-header-box .nav-item:nth-of-type(4)'))
.click();
await driver.sleep(1000)
```

3. 当点击小册之后，会调用 **await driver.sleep(1000)**，因为当页面点击之后，页面会进行重新渲染，此时防止接下来的操作，获取不到数据或者获取的数据不是期望值，增加一个延迟确保数据的准确性

### 将前端的全部小册数据爬取

1. 根据**driver.findElements(By.css('.list-wrap .books-list .item'))**获取前端小册列表，需要注意的是，当页面点击navBar之后，页面会进行重新渲染，但是此时若是直接去获取小测列表将会存在风险，因为在页面没有渲染完成之前获取不到期望值，而代码也会异常终止程序的运行
2. 进行迭代取出希望获取的数据，根据**itemInfo.findElement(By.css('.info .title')).getText()**

```
while (true) {
  let listViewError = true;
  
  try {
    // 获取小册列表
    let _li = await driver.findElements(By.css('.list-wrap .books-list .item'));
    console.log(_li.length);
    
    for (let i = 0, _len = _li.length; i < _len; i++) {
      const itemInfo = _li[i];
      const title = await itemInfo.findElement(By.css('.info .title')).getText()
      const desc = await itemInfo.findElement(By.css('.info .desc')).getText()
      let price = await itemInfo.findElement(By.css('.info .price-text')).getText()

      _result.push({
        title,
        desc,
        price
      })
    }

    console.log('_result',_result);
    
  } catch (error) {
    if (error) listViewError = false;
  } finally {
    if (listViewError) break;
  }
}
```

> 在获取列表的时候，为什么会在最外层增加一个while呢？在 try catch 中的处理又是起到什么作用？
>
> 1. while能确保会不断的获取数据，直到页面渲染完成获取到期望的数据
> 2. try catch 可以保证程序在遇到异常时不会直接终止程序，可以继续运行
> 3. listViewError表示程序是否存在异常情况，若是存在则会进入 catch 中，这个时候 listViewError 为 false，finally 则不会走break，会继续执行while程序，直到能获取到数据finally才为true，这个时候 finally中则会break 整个while的循环，跳出循环继续执行



### 总结

> 几十行的代码便可以将掘金的小册数据全部爬到，还是简单和好用的

全部代码

```
/*
 * @Author: nordon-wang
 * @Date: 2019-08-13 11:05:36
 * @Description: 爬取掘金数据
 * @Email: nordon-wang@oyohotels.cn
 */

const { Builder, By, Key, until } = require('selenium-webdriver');
let _result = []; // 用来收集获取的数据

(async function start() {
  let driver = await new Builder().forBrowser('chrome').build();

  try {
    // 自动打开掘金
    await driver.get('https://juejin.im/timeline');

    // 点击小册子的navBar 切换路由到小册
    await driver
      .findElement(By.css('.main-header-box .nav-item:nth-of-type(4)'))
      .click();
    await driver.sleep(1000)

    // 点击二级菜单
    await clickViewNav(driver);
    await driver.sleep(1000)

    // 获取数据
    await getList(driver);

  } catch (error) {
    console.log(error);
  } finally {
    let timer = setTimeout(async () => {
      clearTimeout(timer);
      await driver.quit();
    }, 600000);
  }
})();

// 获取渲染完成的按钮
async function clickViewNav(driver) {
  while (true) {
    let viewNavError = true;
    
    try {
      await driver
        .findElement(By.css('.main-container .view-nav .nav-item:nth-of-type(2)'))
        .click();
    } catch (error) {
      if (error) viewNavError = false;
    } finally {
      if (viewNavError) break;
    }
  }
}

// 获取列表数据
// 页面在渲染完成之前无法获取到页面的元素
async function getList(driver) {
  while (true) {
    let listViewError = true;
    
    try {
      // 获取小册列表
      let _li = await driver.findElements(By.css('.list-wrap .books-list .item'));
      console.log(_li.length);
      
      for (let i = 0, _len = _li.length; i < _len; i++) {
        const itemInfo = _li[i];
        const title = await itemInfo.findElement(By.css('.info .title')).getText()
        const desc = await itemInfo.findElement(By.css('.info .desc')).getText()
        let price = await itemInfo.findElement(By.css('.info .price-text')).getText()

        _result.push({
          title,
          desc,
          price
        })
      }

      console.log('_result',_result);
      
    } catch (error) {
      if (error) listViewError = false;
    } finally {
      if (listViewError) break;
    }
  }
}

```
