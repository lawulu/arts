## Algorithm
```
https://leetcode.com/problems/add-two-numbers/description/

You are given two non-empty linked lists representing two non-negative integers. The digits are stored in reverse order and each of their nodes contain a single digit. Add the two numbers and return it as a linked list.
You may assume the two numbers do not contain any leading zero, except the number 0 itself.

Example:

Input: (2 -> 4 -> 3) + (5 -> 6 -> 4)
Output: 7 -> 0 -> 8
Explanation: 342 + 465 = 807.
```
好久没有做算法题了，这道简单的题提交了三次才过。


```
class Solution {
    public ListNode addTwoNumbers(ListNode l1, ListNode l2) {
         if (l1 == null) {
            return l2;
        }
        if (l2 == null) {
            return l1;
        }

        int toBeAdded = 0;

        int temp = l1.val + l2.val;
        int val = (temp % 10);
        toBeAdded = (temp / 10);


        ListNode result = new ListNode(val);


        ListNode pointNode = result;

        while (l1.next != null && l2.next != null) {
            l1 = l1.next;
            l2 = l2.next;


            temp = l1.val + l2.val + toBeAdded;
            val = (temp % 10);
            toBeAdded = (temp / 10);
            ListNode tempNode = new ListNode(val);
            pointNode.next = tempNode;
            pointNode = tempNode;


        }

        while (l1.next != null) {
            l1 = l1.next;

            temp = l1.val + toBeAdded;
            val = (temp % 10);
            toBeAdded = (temp / 10);
            ListNode tempNode = new ListNode(val);
            pointNode.next = tempNode;
            pointNode = tempNode;
        }
        while (l2.next != null) {
            l2 = l2.next;

            temp = l2.val + toBeAdded;
            val = (temp % 10);
            toBeAdded = (temp / 10);
            ListNode tempNode = new ListNode(val);
            pointNode.next = tempNode;
            pointNode = tempNode;
        }

        if(toBeAdded>0){
            ListNode last =new ListNode(toBeAdded);
            pointNode.next=last;
        }

        return result;

    }
}
```

## Review
https://doc.akka.io/docs/akka/current/stream/stream-introduction.html

本周技术调研，看了下Akka Stream的文档。有几个有意思的地方：
- 把数据当做Stream来处理，是和现实中一致的(例如TCP),也是必要的（因为数据经常会变得太大）
- Akka Stream实现了Reactive Streams规范，但是对用户是不可见的，因为后者是不同操作边界之间的规范


## Tip

### Selenium+Chrome Headless
刚工作时候曾经试图研究过Web自动化测试，留下了Selenium上手成本太高的顽固印象。最近要爬某个反爬虫做的很好的网站。之前对PhantomJS印象也不好，尝试了HtmlUnit失败之后，发现Github上的爬虫基本都是基于Selenium，决定开搞。
#### WebDriver
Selenium抽象出一个WebDriver，来和浏览器打交道。简单阅读了[官网](https://www.seleniumhq.org/docs/03_webdriver.jsp)之后，意外的发现其实很简单。对于WebDriver来说，可以做这些事：
- 选中Element：使用findElement API，支持CSS和XPath选择器
- 执行JS：可以直接执行（JavascriptExecutor），也可以选中Element，调用对应的事件
- 等待页面执行：使用wait方法
- 好像不能输出某个DOM结构的html?

#### Chrome Headless

Chrome官方推出[Headless模式](https://developers.google.com/web/updates/2017/04/headless-chrome)之后，简直是反爬虫的噩梦。对于强度不大的爬取，反爬虫几乎毫无办法。
- 支持Repl模式和Debug模式（服务器上实验没成功）
- 支持各平台的[WebDriver](https://github.com/SeleniumHQ/selenium/wiki/ChromeDriver)


#### 上代码

```
System.setProperty("webdriver.chrome.driver", "/service/app/webdriver/chromedriver");
ChromeOptions chromeOptions = new ChromeOptions();
String userAgent ="Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/60.0.3112.50 Safari/537.36";
chromeOptions.addArguments("--headless");
chromeOptions.addArguments("--disable-gpu");
chromeOptions.addArguments("--user-agent="+userAgent);
WebDriver driver = new ChromeDriver(chromeOptions);

driver.get("http://m.ctrip.com/webapp/hotel/hoteldetail/dianping/4709722.html?&fr=detail&atime=20180720&days=1");

String oldList = driver.findElement(By.cssSelector("#content > article > div.js_comment_wrap > div.myhotel-w.js_commentlist")).getText();
int oldLength = oldList.length();

List<WebElement> elements = driver.findElements(By.xpath("//li[@class='js_filter_type']"));
for (WebElement element : elements) {
    String dataKey = element.getAttribute("data-ubt-key");
    if("c_hotel_comment_filter_sortType".equals(dataKey)){
        element.click();
    }
}


(new WebDriverWait(driver, 10)).until(new ExpectedCondition<Boolean>() {
    public Boolean apply(WebDriver d) {

        WebElement clickButtons = d.findElement(By.xpath("//*[@id=\"ui-view-3\"]/ul[3]"));
        return clickButtons.isDisplayed();
    }
});
driver.quit();
```

### 一个思维方式的误区
爬取过程中，发现在不使用Headless模式时候，爬取正常，使用之后，只要有ajax请求，页面就显示错误。

有两种可能，一种是驱动/chrome/OS版本匹配问题，一种是反爬虫机制起了作用。

因为先入为主的认为反爬虫无法识别Headless的Chrome，然后一直折腾第一个。其实但凡成熟的软件，非特殊需求，遇到问题，还是要先怀疑使用的对不对。
无意中看到这个link：[Detect Chrome running in headless mode from JavaScript](https://stackoverflow.com/questions/44397492/detect-chrome-running-in-headless-mode-from-javascript),才意识到可以从JS可以识别Headless模式。找到了[这篇文章](https://intoli.com/blog/making-chrome-headless-undetectable/)，才解决了这个问题。一共花了差不多一个人天才搞定。

其实花一些成本很低的方式验证一下第二个可能是非常有效的，比如先看下Header：

```
google-chrome --headless --disable-gpu --no-sandbox --dump-dom https://httpbin.org/get
```
就会发现
```
"User-Agent":"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) HeadlessChrome/67.0.3396.99 Safari/537.36"
```



## Share
### 贸易战

对贸易战的的未来持悲观态度。欧美民众优越的生活是凭借科技和文化优势，占据产业链顶端，对亚非拉广大同胞进行吸血的基础之上，现在天朝要搞产业升级，这么大的体量，假如产业升级成功，对欧美来说就是一个灾难。观海同志在位时候，就说过地球支撑不起中国人和美帝过上同样的生活的话。（我一直觉得民主党的TPP比川普的贸易战狠多了）。这么大的利益冲突之下，基本是无解，大概率是天朝认怂。

认怂就是又一次倒逼改革,这次是真·消费升级还是换一种方式割韭菜？我们拭目以待。贸易战是一个黑天鹅，如果早知道今年贸易战这么激烈，肯定不会换房。某种程度上，我也被刘总理给骗了。