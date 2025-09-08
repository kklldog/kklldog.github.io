前几天在群里看到有大神分享 Copoilot AI 写代码，看了几个截图有点不敢相信自己的眼睛。今天赶紧自己也来体验一下 Copoilot AI 写代码到底有多神奇。
## 申请
现在 Copoilot 还处在预览阶段，想要体验需要先申请。等待大概一晚会收到邮件提示申请试用成功，然后就可以安装 copilot 的插件进行体验了。   
申请的地址为 https://copilot.github.com/ 。
## 安装插件
打开 VSCode 的扩展面板，搜 copilot 找到插件点击安装。   
![](https://static.xbaby.xyz/20211104002027.png)
安装完成后会提示你需要登录 github 进行授权。这一步在国内可能会校验失败，多尝试几次就会成功。   
![](https://static.xbaby.xyz/20211104002134.png)
## 开始体验
安装完插件后就可以开始体验了。使用其实来也非常简单，你只需要写下注释比如 // 比较两个数的大小 ，然后回车，稍等一会就 copilot 就会把建议代码给列出来。或者直接写下方法名，比如 function compareDate () { 写下第一个 { 的时候按下回车，稍等一会 copilot 就会给出建议代码，按一次 tab 就会生成代码。   
copilot 可能会给出多种建议的代码，使用 alt + [ 或者 alt + ] 可以进行切换。   
下面让我们来测试一下 copilot 生成的代码到底怎么样。
### 测试JavaScript
新建一个名为 copilottest.js 的文件。   
先来一个简单的，实现一个快速排序，这种标准的算法生成起来应该没啥难度。直接 // 快排 回车就生成出来了，看了下代码，还不错哈。
```
//快排
function quickSort(arr, left, right) {
    if (left < right) {
        var pivotIndex = partition(arr, left, right);
        quickSort(arr, left, pivotIndex - 1);
        quickSort(arr, pivotIndex + 1, right);
    }
}

```
再来一个比较常用的字符串转时间，并且顺便生成一个测试方法。
```
//把格式为"yyyy-MM-dd"的字符串转换为Date类型
function stringToDate(str) {
    var date = eval('new Date(' + str.replace(/\d+(?=-[^-]+$)/,
        function (a) { return parseInt(a, 10) - 1; }).match(/\d+/g) + ')');
    return date;
}
//测试 stringToDate 方法是否正确
function testStringToDate() {
    var str = "2017-01-01";
    var date = stringToDate(str);
    alert(date);
}
```
再来带判断逻辑的：比较两个数字，如果第一个数字大于第二个数字，则返回“微信”否则返回"qq"。   
这个考验一下 copilot 对中文的理解。
```
//比较两个数字，如果第一个数字大于第二个数字，则返回“微信”否则返回“qq
function compare(num1, num2) {
    if (num1 > num2) {
        return "微信";
    } else {
        return "qq";
    }
}
```
再来一个判断时间是星期几的函数
```
//根据时间判断星期几
function getWeek(date) {
    var week;
    switch (date.getDay()) {
        case 0:
            week = "星期日";
            break;
        case 1:
            week = "星期一";
            break;
        case 2:
            week = "星期二";
            break;
        case 3:
            week = "星期三";
            break;
        case 4:
            week = "星期四";
            break;
        case 5:
            week = "星期五";
            break;
        case 6:
            week = "星期六";
            break;
    }
    return week;
}
```
再来试试 http 请求写的怎么样，访问一下百度把html返回。
```
//返回baidu.com的html
function getBaiduHtml() {
    var url = "http://www.baidu.com";
    var xhr = new XMLHttpRequest();
    xhr.open("GET", url, false);
    xhr.send();
    var html = xhr.responseText;
    return html;
}

```
JavaScript 操作 html 也很常见，那么试试查找元素怎么样
```
//查找页面上所有的div
function findAllDiv() {
    var divs = document.getElementsByTagName("div");
    return divs;
}

//查找所有页面上class=user的input元素
function findAllUserInput() {
    var inputs = document.getElementsByClassName("user");
    return inputs;
}
```
再来试试 http 提交对象怎么样
```
//请求一个url，post一个json对象
function postJson(url, json) {
    var xhr = new XMLHttpRequest();
    xhr.open("POST", url, false);
    xhr.setRequestHeader("Content-Type", "application/json");
    xhr.send(JSON.stringify(json));
    var response = xhr.responseText;
    return response;
}
```
来个有点难度的，爬取博客园首页的文章的标题
```
//爬取cnblogs.com所有文章的标题
function getAllArticleTitle() {
    var url = "http://www.cnblogs.com/";
    var xhr = new XMLHttpRequest();
    xhr.open("GET", url, false);
    xhr.send();
    var html = xhr.responseText;
    var reg = /<a href="(.*?)" target="_blank">(.*?)<\/a>/g;
    var result = html.match(reg);
    return result;
}
```
测试一下使用某个类库来完成一些操作
```
// 使用hls.js播放视频
function playVideo(video) {
    var hls = new Hls();
    hls.loadSource("http://www.w3school.com.cn/i/movie.mp4");
    hls.attachMedia(video);
    hls.on(Hls.Events.MANIFEST_PARSED, function () {
        video.play();
    });
}
```
看到这里有没有被震惊到？以上代码全部是 copilot 自动生成的，我仅仅是写下了几行注释。可以看到注释其实写的也很简单，copilot 生成的代码基本上都符合注释的内容，可以说 copilot 对中文注释的理解非常到位。如果你的注释写的更加详细，那么代码就会更加符合你的要求。
### 测试.NET C#
让我们在来试试 C# 代码的提示功能怎么样。新建一个文件名为 copilottest.cs 的文件。   
先来一个简单的，比较两个时间的大小。
```
 //比较两个时间的大小
    public static bool CompareTime(string time1, string time2) {
        DateTime dt1 = DateTime.Parse(time1);
        DateTime dt2 = DateTime.Parse(time2);
        if (dt1 > dt2) {
            return true;
        } else {
            return false;
        }
    }
```
做为 CRUD boy 自然要测试下数据库操作。   
先来试试使用EF来查找用户
```
//使用EF查询用户
    public static List<User> GetUserList() {
        using (var db = new CopiloteContext()) {
            return db.User.ToList();
        }
    }

```
再来试试使用 dapper 的写法
```
  //使用dapper查询用户    
    public static List<User> GetUserListDapper() {
        using (IDbConnection db = new SqlConnection("server=.;database=copilote;uid=sa;pwd=123456")) {
            return db.Query<User>("select * from user").ToList();
        }
    }
```
让我们加大难度，来个连接跟分组统计人数
```
   //查询2020年每个班级学生的人数
    public static void GetClassStudentCount() {
        using (IDbConnection db = new SqlConnection("server=.;database=copilote;uid=sa;pwd=123456")) {
            db.Query<ClassStudentCount>("select class.name as classname,count(student.id) as studentcount from student left join class on student.classid=class.id where class.year=2020 group by class.name").ToList();
        }
    }
```
让它自动生成改删查操作
```
    class student {
        public int id { get; set; }
        public string name { get; set; }
        public int classid { get; set; }
    }

    //student的新增操作
    public static void AddStudent(student s) {
        using (IDbConnection db = new SqlConnection("server=.;database=copilote;uid=sa;pwd=123456")) {
            db.Execute("insert into student(name,classid) values(@name,@classid)", s);
        }
    }
    //student的更新操作
    public static void UpdateStudent(student s) {
        using (IDbConnection db = new SqlConnection("server=.;database=copilote;uid=sa;pwd=123456")) {
            db.Execute("update student set name=@name,classid=@classid where id=@id", s);
        }
    }
    //student的删除操作
    public static void DeleteStudent(int id) {
        using (IDbConnection db = new SqlConnection("server=.;database=copilote;uid=sa;pwd=123456")) {
            db.Execute("delete from student where id=@id", new { id = id });
        }
    }
```
再来试试生成 ASP.NET MVC 的 action 方法。
```
class UserController : Controller {
        //从 request 获取 name 参数查询用户，如果查到就返回否则返回状态404
        public ActionResult GetUser(string name) {
            User user = UserService.GetUser(name);
            if (user != null) {
                return Json(user);
            } else {
                return HttpNotFound();
            }
        }

        //使用[FromBody]映射成user对象，并保存到数据库
        public ActionResult AddUser([FromBody]User user) {
            UserService.AddUser(user);
            return Json(user);
        }
}
```
其实我还试验了一下 JAVA 的代码，也是毫无压力，这里就不贴出来了。
## 总结
到这里我已经有点无话可说了。 copilot 深深的震撼了我，感觉 copilot 对注释的理解根据人类无差别，生成的代码基本是符合要求的，即使有一点问题那也是因为没有上下文的原因， copilot 只能生成最常用的语句。copilot 虽然只是生成一个个短小的函数，但是再复杂的系统不都是由无数个简单的函数组成的吗？况且 copilot 还只是预览版，如果再迭代几个版本，AI 再训练几年那么是不是可以有无限可能。到这里心里略有一点忧伤，以后一些低级代码工作很可能被 AI 代替，程序员的入门门槛进一步降低，这到底是好事还是坏事呢？

## 关注我的公众号一起玩转技术   

![](https://static.xbaby.xyz/qrcode.jpg)