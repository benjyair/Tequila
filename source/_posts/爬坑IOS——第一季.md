---
title: 爬坑IOS——第一季
layout: post
date: 2015/05/04 22:20:19
tags : Swift
---

辛苦了将近一个多月，公司应用的 IOS 第一个版本终于发布啦。从无到有，一步一个坑的走到了现在。自信心受到了前所未有的打击，不过还好坚持了下来。分享下爬坑的经验。

### 爬坑 —— String 类
Swift 语法对 String 支持的并不如 Java 全面，有好多方法调用都好麻烦，对于熟悉 Java 开发的我来说哪怕做个简单的字符串截取甚至都要查API，不过 Swift 有个特别的地方就是可以对类进行扩展（不是继承），下面是我对 Java String 类的常用方法的封装，用 Swift 来实现了几个 Java String 类的几个常用 API。

```swift

/**
*  @author KokerWang, 15-04-22 14:04:35
*
*  拓展String
*/
extension String {
    
    /// 返回长度
    var length: Int {
        return count(self)
    }
    
    func trim() -> String {
        return self.stringByTrimmingCharactersInSet(NSCharacterSet.whitespaceCharacterSet())
    }
    
    func split(regularExpression : String) -> [String] {
        return self.componentsSeparatedByString(regularExpression)
    }
    
    func substring(start : Int, end : Int) -> String {
        var si = getRange(start)!
        var ei = getRange(end)!
        return self.substringWithRange(Range<String.Index>(start: si, end: ei))
    }
    
    func replace(oldChary : String, newChart : String) -> String {
        return self.stringByReplacingOccurrencesOfString(oldChary, withString: newChart, options: nil, range: nil)
    }
    
    func getRange(index : Int) -> String.Index?{
        if(index > length){
            return nil
        }else{
            return advance(self.startIndex, index)
        }
    }

```

### 爬坑 —— 3DES，BASE64，SHA1 算法
出于安全性的考虑，用到的加密算法较多，组合模式也比较特殊，因为直接调用的和 Android 一样的接口，所以也就是说 IOS 端的加密方式要和 Android 端一致，但是我并没有找到 Swift 合适的类库来使用，后来只能使用了 OC 的算法来用。遇到的问题还是挺多的，这里我只说几个典型的问题。
> 1 、BASE64 加密每 76 个字符增加一个换行符

> 2 、BASE64 加密如果被加密字符串正好是 76 的整数倍最后一个不加换行符

> 3 、3DES 加密使用一种加密方式而不是使用三种加密

第一，二个问题因为我并没有找到如何切换 BASE64 加密选项的参数（ Java 内是有这个 Option 的）。所以最后我的解决方法是手动用 Swift 来实现了这个功能。

第三个问题我查 OC 的源码发现可以修改了 CCCrypt() 方法第三个参数的值来修改加密方式，源码里只有 2 个枚举，我先是直接传 nil 发现不行后来改传 0x0003 得以解决。代码如下

```swift

enum {
    /* options for block ciphers */
    kCCOptionPKCS7Padding   = 0x0001,
    kCCOptionECBMode        = 0x0002
    /* stream ciphers currently have no options */
};

```

```swift

var ccStatus = CCCrypt(encryptOrDecrypt,
                       kCCAlgorithm3DES,
                       0x0003,
                       vkey,
                       kCCKeySize3DES,
                       nil,
                       vplainText,
                       plainTextBufferSize,
                       (void *)bufferPtr,
                       bufferPtrSize,
                       &movedBytes);

```

以上代码并非完整代码，只是我拿出来的具体修改部分，有需要的可以在网上找对应的算法下载下来对比即可。

其他几个算法修改的幅度不大，出于安全考虑我也不多说了

### 爬坑——UI
1、上篇文章里我介绍了如何从 Storyboard 中加载出来一个 ViewController，但是如果要在 Storyboard 中给这些 VeiwController 添加上苹果推荐的 Navigation Controller 并顺利拿出来展示可就难办了，开始的时候我使用代码的方式来加上导航栏，这个没有难度。可是既然已经使用了 Storyboard 还是用代码编写实在说不过去，后来我想了个点子，那就是在 ViewController 的 Navigation Controller 上加一个 id 在通过上篇文章介绍的方式拿出来，如果需要设置 delegate 的话在从该 Controller 内拿出 childViewControllers[0] 来设置。 这样就既有导航栏，又没有写代码，可谓一举两得，具体代码看第三条。

2、网络请求处理
因为多数界面都有网络请求，对于多种请求结果的回调，还是需要合理的构思一下代码逻辑，我的做法和 Android 的处理类似，在所有的 ViewController 加一个基类，通过这个类来处理所有通用回调的处理在发送 HTTP 请求的时候把 self 作为 delegate 传过去。我在项目里并没用第三方网络请求类库，因为我感觉原生的已经足够满足我的需求了。看代码：

```swift
import UIKit

protocol NetCell{
    func noNet()
    func noCoonection()
    func noCell(str : String)
}

class BaseViewController: UIViewController, NetCell{
    
    func noNet(){
        self.presentViewController(buildAlert("网络不通，请检查后再试！"), animated: true, completion: nil)
    }
    
    func noCoonection(){
        self.presentViewController(buildAlert("网络异常，请稍后再试！"), animated: true, completion: nil)
    }
    
    func noCell(str : String){
        self.presentViewController(buildAlert(str), animated: true, completion: nil)
    }
    
}

```

代码中 buildAlert 方法是我写的一个工具类，效果是弹出一个提示框。具体我回在后面拿出来。
下面是 HTTP 请求的工具类

```swift
import UIKit
class NetUtil: NSObject{

}

func sendPost (url : String , data : String, delegate : NetCell, cell : (result: NSDictionary) -> Void) {
    if checkNetWork() {
        delegate.noNet()
        return
    }
    var url = NSURL(string: url)
    var request = NSMutableURLRequest(URL: url!, cachePolicy: NSURLRequestCachePolicy.ReloadIgnoringLocalAndRemoteCacheData, timeoutInterval: 10)
    request.HTTPMethod = "POST"
    request.addValue("application/json; charset=utf-8", forHTTPHeaderField: "Content-Type")
    request.HTTPBody = data.dataUsingEncoding(NSUTF8StringEncoding)
    NSURLConnection.sendAsynchronousRequest(request, queue: NSOperationQueue.mainQueue(), completionHandler: {
        (response:NSURLResponse!, data : NSData!,error : NSError!) -> Void in
        if data != nil && data.length > 0 {
            var jsonResult:NSDictionary = NSJSONSerialization.JSONObjectWithData(data, options: NSJSONReadingOptions.MutableContainers, error: nil) as! NSDictionary
            var code = jsonResult["code"] as! String
            if code.hasPrefix("1") {
                cell(result: jsonResult)
            }else{
              delegate.noCell(jsonResult["msg"] as! String)
            }

        }else{
            delegate.noCoonection()
        }
    })
}

```

有一小部分的逻辑代码可以忽略，这里只是为了说明调用的过程。

3、未登录自动跳转实现
这样的逻辑在常见 APP 中应用的很广泛，我的做法是重写了 UIViewController 的 viewWillAppear 方法

```swift

    override func viewWillAppear(animated: Bool) {
        login()
    }
    
    /**
    检查登陆
    */
    func login(){
        
        if /* 你的逻辑 */ {
            var controller = getController("reg_nav") as! UINavigationController
            var reg = controller.childViewControllers[0] as! RegisterController
            reg.delegate = self
            self.presentViewController(controller, animated: true,completion: nil)
        }
    }
    
```

4、工具类
在开发过程中还是抽象出来了好多的工具类，如下

```swift
import UIKit

/**
获取缓存数据

:param: key key

:returns: value
*/
func getData (key : String) -> String {
    var ud = NSUserDefaults.standardUserDefaults()
    if let value: AnyObject = ud.objectForKey(key) {
        return value as! String
    }else {
        return ""
    }
}

/**
保存缓存数据

:param: key   key
:param: value value
*/
func setDate(key : String, value : String){
    var ud = NSUserDefaults.standardUserDefaults()
    ud.setObject(value, forKey: key)
}

/**
获取Storyboard内的UI组件

:param: name 组件Storyboard ID

:returns: UIViewController
*/
func getController(name : String) -> UIViewController{
    var story =  UIStoryboard(name: "Main", bundle: nil)
    return story.instantiateViewControllerWithIdentifier(name) as! UIViewController
}

/**
Log 日志

:param: obj 日志输出对象
*/
func myLog (obj : AnyObject) {
    println("****************************************************************************")
    println("****")
    println("****log:  \(obj)")
    println("****")
    println("****************************************************************************")
}

/**
返回UIAlertController 对象 通过 self.presentViewController(buildAlert(msg as String), animated: true, completion: nil) 展示

:param: msg 提示信息

:returns:
*/
func buildAlert (msg : String) -> UIAlertController {
    let alert = UIAlertController(title: "提示", message: msg, preferredStyle: UIAlertControllerStyle.Alert)
    alert.addAction(UIAlertAction(title: "确定", style: UIAlertActionStyle.Default, handler: nil))
    return alert
}

/**
获取随机数

:param: length 随机数的长度

:returns: 指定长度的随机数
*/
func getVerificationCode(length : Int) -> String{
    if(length < 1 ){
        return ""
    }
    var str = ""
    for index in 1...length{
        str += ("\(arc4random_uniform(10))")
    }
    return str
}

private var hindenExtCellView : UIView?

/**
隐藏多余的table line

:param: tableview UITableView
*/
func hindenExtCellLine (tableview : UITableView){
    if hindenExtCellView == nil {
        hindenExtCellView = UIView()
        hindenExtCellView!.backgroundColor = UIColor.clearColor()
    }
    tableview.tableFooterView = hindenExtCellView
}

```

好啦，将近 2 个月的积累就说到这里啦，爬坑系类第一季到此结束。
