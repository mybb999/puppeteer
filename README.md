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
  const targetLink = await page.evaluate(() => {//经常用到一步，写你所需要执行逻辑的方法，有点类似vue里面的method一样，并且一定要return返回结果
    return [...document.querySelectorAll('.result a')].filter(item => {//将a标签过滤传到过滤方法中
      return item.innerText && item.innerText.indexOf('"人形自走炮"是什么意思?_百度知道')!=-1   //判断自走炮地址的条件
    }).toString()
  });
  await page.goto(targetLink);
  await page.waitFor(1000);
  browser.close();
})()

</pre>


page.evaluate(方法，传入方法的参数)方法实际就是写你的逻辑方法地方，还有另一种page.$eval('选择目标',方法,传入方法的参数)代替，
实际上是多了个选择器,如果不选择元素的话用evaluate即可，据情况使用吧。

document.querySelectorAll返回的是一个NodeList特殊列表，所以用ES6扩展运算符的...把NodeList列表分割数列，再用[]把它们转换为数组。
...和[]经常用来代替ES5的function.apply(方法，[参数])和function.call(方法,参数A,参数B,参数一堆...........)，具体百度。

![image](https://github.com/mybb999/images/blob/master/puppeteer1.png)
![image](https://github.com/mybb999/images/blob/master/puppeteer2.png)


2.iframe获取方式

一种获取页面内容方式：
<pre>
const frame = await page.mainFrame()//返回页面的主frame
const bodyHandle = await page.$('body');//选择页面body
const html = await page.evaluate(body => body.innerHTML, bodyHandle);，输出body
await bodyHandle.dispose();//销毁句柄
console.log(html)
</pre>

还有一种获取iframe对象：
let iframe = await page.frames().find(f => f.name() === 'login_frame');
这个满大街都是，找到一个iframe的name的名字是叫"login_frame"的



另外要注意iframe有多个id和name相同，有点类似反爬虫的机制在里面，贴上我爬久游敢达的官网代码，本来打算自动登录中队页面，后来发现有验证码并且iframe元素选择不了，例如page.type()无法选取焦点，最后通过某个大神告诉我选择一个输入框里的另一个标签元素才成功输入。

<pre>

const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({headless: false});
  const page = await browser.newPage();
  page.setViewport({width:1920,height:1080});
  // page.on('console', msg => console.log('PAGE LOG:', msg.text()));
  await page.goto('http://ms.9you.com/web_v3/index.html');
  page.click('.fixMenu03 a');
  await page.waitFor(4000);
  // const frame2 = await page.mainFrame()
  // console.log(frame2)
  // const bodyHandle = await frame2.$('html');    
  // const html = await frame2.evaluate(body => body.innerHTML, bodyHandle);
  // await bodyHandle.dispose();  



  let iframe = await page.frames().find(f => f.name() === 'login_frame');  
  

  await iframe.waitFor(2000);
  await iframe.click('#uin_tips');  
  await iframe.type('#uin_tips','684654234542273');
  await iframe.click('#pwd_tips');
  await iframe.type('#pwd_tips','123456');  
  // iframe.click('#password');
  // iframe.type('#password','123456');
  // await page.waitFor(1000);  
  // await iframe.click('#password');  
  // await page.waitFor(1000); 
  // await iframe.type('#password','54686787',{delay:100});
  
  
  await page.waitFor(1000);  
  // browser.close();
})()

</pre>



里面有多个相同iframe的id和name，并且选取id元素输入不是在input标签里填
![image](https://github.com/mybb999/images/blob/master/puppeteer3.png)


反正蛋是碎了，总的来说如果不太深入只是想做普通简单的点击工具还是可以的，例如自己做的页面进行测试点击之类还是挺有帮助的。
