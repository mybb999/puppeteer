Puppeteer是Node.js的一个库类，由于puppeteer这玩意玩得还不太熟，但是基本命令还是懂得的，例如这是我从某游官网去自行点击的一小部分操作，原本目的就不是爬虫而是对官网的中队系统奖励自动点击分配物资使用的，但却死在了iframe和验证码手上（虽然想过验证码自己输入然后监听这个事件的发生），也就随便研究玩玩把，总的来说先记录一下探索过程的问题。

一般是先安装个npm，然后装个cnpm(国内镜像)，然后接着cnpm i puppeteer就可以了，npm下载会出问题因为在下载puppeteer时会自带一个Chromium内核，但是国外封杀了下载不了Chromium。

1.简单使用
<pre>

const puppeteer = require('puppeteer');//引入库类

(async () => {
  const browser = await puppeteer.launch({headless: false});//headless代表有头还是无头，实际就是打不打开浏览器展示而已
  const page = await browser.newPage();//生成一个浏览器页面
  await page.setViewport({width:1800,height:1500})//设置打开的页面宽高
  await page.goto('https://baidu.com');//page.goto 前往页面
  await page.type('#kw', '人形自走炮', {delay: 100});//page.type('目标','输入文字',输入间隔时间)
  page.click('#su')//不用说了
  await page.waitFor(1000);//等待页面执行时间
  const targetLink = await page.evaluate(() => {//经常用到的一步，写你所需要执行逻辑的方法，有点类似vue里面的method一样，并且一定要return返回结果
    return [...document.querySelectorAll('.result a')].filter(item => {
      return item.innerText && item.innerText.indexOf('"人形自走炮"是什么意思?_百度知道')!==0
    }).toString()
  });
  await page.goto(targetLink);
  await page.waitFor(1000);
  browser.close();
})()

</pre>

page.evaluate()方法还有另一种代替

2.iframe获取方式
