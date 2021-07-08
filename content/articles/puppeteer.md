---
title: "Puppeteer"
date: 2021-06-28T09:52:48+08:00
draft: false
---

初识Puppeteer
2017年，Chrome开发团队发布了一个node.js的包，用来模拟Chrome浏览器的运行，这个包就是Puppeteer。和同类产品Phantomjs、Selenium相比，Puppeteer作为“自家的”孩子，具有天然的优势。Puppeteer默认以无头（无界面）的方式运行，但是也可以配置运行为有界面的Chrome。使用puppeteer可以自动化在浏览器中完成手动执行的大多数事情，比如：
生成页面的屏幕截图和PDF。
抓取SPA并生成服务端渲染SSR。
自动化提交表单、UI测试、键盘输入等。
捕捉网站的时间线跟踪，以帮助诊断性能问题。
（一）爬虫
以前尝试用过python写爬虫，但是因为对语言不熟悉，对它的库也不熟悉，所以写起来非常耗时，需要学习很多新的知识。然后我尝试用最熟悉的js语言来写，果然发现了新的世界，一切变得轻松起来。
下面是我爬取SonarQube的项目数据的代码，算是较为复杂的场景。
1. 使用launch创建一个无头浏览器
2. 使用newPage创建一个新窗口
3. page.goto进入登录页面
4. 网站使用的是react渲染页面，所有进入页面时，页面元素还没有渲染，如果直接获取DOM会导致获取不到，这里使用waitForSelector来等待DOM渲染完成
5. 使用page.type给输入框输入用户名和密码
6. 模拟用户点击登录操作
7. page.goto进入目标页面。目前很多网站使用前端渲染，导致直接输入地址无法获取到最终数据，需要模拟用户输入，等待接口响应，重新渲染之后才能开始爬取数据
使用waitForSelector等待搜索框加载完成
使用page.evaluate()进入浏览器的上下文环境，在这里可以直接使用DOM操作语句等，这里使用document.querySelector获取到搜索框并清空搜索框文字
page.type给搜索框输入关键字
waitForResponse等待接口响应，因为响应的内容不是纯粹可读取数据，无法直接解析，所以等到页面渲染后来解析页面元素。响应内容如下图。

8. 页面是分页加载的，所以如果有下一页，还需要点击下一页
9. 继续等待接口响应
10. 解析页面元素，page.$$eval()类似于document.querySelectorAll，page.$eval()类似于document.querySelector()
11. 操作结束后browser.close()关闭浏览器
12. 使用xlsx将爬取的数据导出为excel
13. 将excel文件保存到本地
```
const puppeteer = require('puppeteer-core');
const findChrome = require('carlo/lib/find_chrome');
const XLSX = require('xlsx');

// 搜索关键词
const keywords = ['k12', 'ippp'];

const data = [];

(async () => {
  let findChromePath = await findChrome({})
  let executablePath = findChromePath.executablePath

  const browser = await puppeteer.launch({
    executablePath,
    headless: true,
  })

  const page = await browser.newPage();
  page.on('console', (msg) => console.log(msg.text()))
  // 登录
  await page.goto('http://22.4.15.55:9000/sessions/new')
  await page.waitForSelector('#login')
  await page.type('input[name="login"]', 'chenyu')
  await page.type('input[name="password"]', 'chenyu')
  await page.click('#login_form .text-right button')

  try {
    await page.goto('http://22.4.15.55:9000/projects/favorite')
  } catch(err) {
    console.log(err)
  }

  for (let keyword of keywords) {
    await (async (keyword) => {
      await page.waitForSelector('.boxed-group')
      await page.waitForSelector('.projects-topbar-item-search input[type="search"]')
      await page.evaluate(() => {
        console.dir(document.querySelector('.projects-topbar-item-search input'))
        document.querySelector('.projects-topbar-item-search input').value = ''
      })
      await page.type('.projects-topbar-item-search input[type="search"]', keyword)
      await page.waitForResponse('http://22.4.15.55:9000/api/organizations/search?organizations=default-organization')
      await page.waitFor(1000)
      await page.waitForSelector('.boxed-group')
      try {
        await page.waitForSelector('.spacer-top.note.text-center .spacer-left')
        await page.click('.spacer-top.note.text-center .spacer-left')
        await page.waitForResponse('http://22.4.15.55:9000/api/organizations/search?organizations=default-organization')
        await page.waitFor(1000)
      } catch(err) {
        console.log(err)
      }
      const arr = await page.$$eval('.projects-list .boxed-group', (els) => {
        const result = els.map((el) => {
          const obj = {}
          obj.name = el.querySelector('.project-card-name a').innerText;
          el.querySelectorAll('.project-card-measure-number .spacer-right').forEach((item, key) => {
            switch (key) {
              case 0:
                obj.bug = item.innerText;
                break;
              case 1:
                obj['漏洞'] = item.innerText;
                break;
              case 2:
                obj['坏味道'] = item.innerText;
                break;
              case 3:
                obj['覆盖率'] = item.nextElementSibling.innerText;
                break;
              case 4:
                obj['重复率'] = item.nextElementSibling.innerText;
                break;
              default:
                break;
            }
          })
          el.querySelectorAll('.project-card-measure-number .rating').forEach((item, key) => {
            switch (key) {
              case 0:
                obj['bug级别'] = item.innerText;
                break;
              case 1:
                obj['漏洞级别'] = item.innerText;
                break;
              case 2:
                obj['坏味道级别'] = item.innerText;
                break;
              default:
                break;
            }
          })
          return obj
        })
        return result
      })
      data.push(arr)
    })(keyword)
  }
  await browser.close()
})().then(() => {
  data.forEach((val, index) => {
    exportFun(val, keywords[index])
  })
})

const exportFun = function (data, name) {
  const ws = XLSX.utils.json_to_sheet(data)
  const wb = XLSX.utils.book_new()
  wb.Sheets['Sheet1'] = XLSX.utils.book_append_sheet(wb, ws, '数据详情')
  console.log(name)
  XLSX.writeFile(wb, `sonar_${name}.xlsx`)
}
```

（二）自动化测试
自动化测试的重要性不言而喻，每次开发一个新功能或者对现有功能进行优化，总是担心测试无法覆盖全局，要是全部做回归测试，又非常耗时。在实践 DevOps 流程的过程中，我们意识到，每次提交一小段代码、频繁提交通常可以极大地减少 bug 的数量，使我们的产品能够更好地响应用户和市场需求，同时为我们自己创造一个良好的工作环境。在实现频繁发布的过程中，手动测试常常是一个很大的难点。因为不管出于什么原因，组织需要为每个版本运行一遍完整的手工测试。这使得新功能的测试会有滞后，以便 QA 可以一次处理尽可能多的之前版本出现的已知 Bug。如果我们忽略了这一点，一旦如果出现 Bug，那么很难跟踪几十次提交中哪一次引入了 Bug。这样通常会阻塞开发几天，只为了在一个版本上重现这个 bug。我们发明了一些复杂的过程，比如一个星期的代码冻结，以创建一个 “稳定的测试环境”，但实际上，它往往并不稳定。
借用Puppeteer，我们可以代码模拟用户行为，将测试用例的操作步骤“录制”下来，之后每次点击运行就可以自动测试完成，并生成测试报告。目前比较成熟的方案就包括cucumber+puppeteer的自动化测试方案。下面是一个简单的demo。
编写测试用例（剧本），按照一定的格式（每句话以Given、When、Then、And、But开头），也支持中文（假如、并且、那么等）
```
# language: zh-CN
功能: 网站测试
主要功能点：
1.计数器功能
2.动态显示

  场景: 计数器功能测试
    假如打开网页，页面应该有标题显示
    并且初始数量应该为0
    那么点击Increment按钮时数量应该增加1

  场景: 动态显示功能
    假如点击Display Message按钮下方动态显示一条数据
```

（2）编写代码实现剧本操作逻辑和预期结果
```
const { BeforeAll, AfterAll } = require("cucumber");

const { Given, When, Then } = require("cucumber");
const puppeteer = require("puppeteer-core");
const findChrome = require("carlo/lib/find_chrome");
const assert = require("assert");
const fs = require("fs");

let browser = null;
let page = null;

BeforeAll(async function () {
  let findChromePath = await findChrome({});
  let executablePath = findChromePath.executablePath;

  browser = await puppeteer.launch({
    executablePath,
    headless: false,
    slowMo: 200
  });
  page = await browser.newPage();
});

AfterAll(async function () {
  await page.close();
  await browser.close();
});

Then(/^打开网页，页面应该有标题显示$/, async function () {
  await page.goto("file:///D:/cassie_project/pick-data/cucu_test.html");
  const pngdata = await page.screenshot({ encoding: "base64" });
  this.attach(pngdata, "image/png");
  const headlines = await page.$$("h1");

  assert.equal(headlines.length, 1);
});

Given(/^初始数量应该为(\d+)$/, async function (n) {
  const count = await page.$eval('[data-test="count-output"]', (e) =>
    parseInt(e.innerHTML)
  );
  const pngdata = await page.screenshot({ encoding: "base64" });
  this.attach(pngdata, "image/png");

  assert.equal(count, n);
});

Then(/^点击Increment按钮时数量应该增加(\d+)$/, async function (arg1) {
  const incrementBtn = await page.$('[data-test="button-increment"]');
  const initialCount = await page.$eval('[data-test="count-output"]', (e) =>
    parseInt(e.innerHTML)
  );
  const expectedCount = initialCount + 1;
  await incrementBtn.click();
  const newCount = await page.$eval('[data-test="count-output"]', (e) =>
    parseInt(e.innerHTML)
  );
  const pngdata = await page.screenshot({ encoding: "base64" });
  this.attach(pngdata, "image/png");

  assert.equal(newCount, expectedCount);
});

Given(/^点击Display Message按钮下方动态显示一条数据$/, async function () {
  await page.goto("file:///D:/cassie_project/pick-data/cucu_test.html");
  await page.$eval('[data-test="button-display"]', (e) => e.click());
  const pngdata = await page.screenshot({ encoding: "base64" });
  this.attach(pngdata, "image/png");

  const displays = await page.$$('[data-test="display"]');
  assert.equal(displays.length, 1);
});
```

（3）执行结果，分别为用例执行正确和用例执行错误的情况。（更详细的内容，更方便的操作，可以使用cukeTest配合）

