由于puppeteer这玩意玩得还不太熟，但是基本命令还是懂得的，例如这是我从某游官网去自行点击的一小部分操作，原本目的就不是爬虫而是对官网的中队系统奖励自动点击分配物资使用的，但却死在了iframe和验证码手上（虽然想过验证码自己输入然后监听这个事件的发生），也就随便研究玩玩把，总的来说先记录一下探索过程的问题。

1.简单使用
<pre>

const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({headless: false});
  const page = await browser.newPage();
  await page.goto('https://baidu.com');
  await page.type('#kw', 'puppeteer', {delay: 100});
  page.click('#su')
  await page.waitFor(1000);
  const targetLink = await page.evaluate(() => {
    return [...document.querySelectorAll('.result a')].filter(item => {
      return item.innerText && item.innerText.includes('Puppeteer的入门和实践')
    }).toString()
  });
  await page.goto(targetLink);
  await page.waitFor(1000);
  browser.close();
})()



</pre>
