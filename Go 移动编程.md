[原文链接](https://www.imooc.com/article/31419)

虽然 Go 并不是一门新语言，不过最近两年来 Go 还是增加了很多有趣的特性，而且使用这门语言的知名项目的数量也在快速的增长。我写过一篇文章，介绍了 SitePoint 用到的编程语言，其中提到了移动端的支持，所以我觉得需要研究一下可能性。

我很高兴 Android 是支持 Go 语言的，这一方面应该是二者都是 Google 的技术，另一方面恐怕也与开发者希望用 Go 替换 Java 的愿望有关。



开始
你需要安装 Golang 1.5+。

接下来需要安装 GoMobile 工具，用于编译和运行 Android 和 iOS 的应用：
```
go get golang.org/x/mobile/cmd/gomobile
gomobile init
```
我们会参考 gomobile 包里的例子，位于 GoLangInstalldir/src/golang.org/x/mobile/example/。如果你没有安装这些例子，参考下面的命令来安装：
```
go get -d golang.org/x/mobile/example/basic
```
构建和安装 Go 的 Native 应用
对于很多应用，编译 Go 的 Native 应用时，忽略那些平台相关的库和接口是可以接受的。如果是这样的情况，编译已有的 Go 代码是很轻松的，我们可以选择使用一个功能子集，这些功能包括：

App 控制和配置
OpenGL ES 2
资源管理
事件管理
一些实验性的包，包括 OpenAL、audio、font、sprite 和运动传感器
我们将从已有的 gomobile 项目里的一些例子开始，你可以用自己项目里的文件替换它们。

Android
构建一个 Android 的 APK 包
```
gomobile build -target=android golang.org/x/mobile/example/basic
```
部署到设备上
```
gomobile install golang.org/x/mobile/example/basic
```
iOS
构建一个 iOS 的 IPA 包
```
gomobile build -target=ios golang.org/x/mobile/example/basic
```
部署到设备
跟 Android 不一样，对于 iOS 来说没有一个统一的部署命令，你需要用你熟知的方式把包拷贝到设备或者模拟器上，例如使用 ios-deploy 工具。

可以用上面的步骤，试试 golang.org/x/mobile/example/audio 这个例子。

让我们深入了解一下 audio 这个例子（详细的代码就不列出了了），你并不需要对 Go 语言非常精通（我就是不太精通），我们先了解一下都能干些啥。

首先你可以看到一些 import 语句：
```
import (...
    "golang.org/x/mobile/app"
    "golang.org/x/mobile/asset"...
)
```
如果你查看一下 import 的这些包所在的目录 GoLangInstalldir/src/golang.org/x/mobile/* 下的文件，你可以发现那些编译到你的代码里的那些 Java 和 Objective-C 文件。

再进一步了解一下，你可以在代码里找到对这些 import 的包（例如 app 和 glctx）的引用。

Going Native
我们可以用 Go 写代码，然后构建一个紧凑的优化过的 native 应用，但是目前这个应用还不是完全的 native 的风格，因为所有依赖的库还都是 Java 或者 Objective-C / Swift 的。我们怎样来改善这个体验呢？

Go Mobile 团队给我们了另一个选择，可以在一个 native 应用里使用 go 的包（也即你的程序）。特别是共享一些公共的 Go 代码，把它们绑定到 native 的代码上是非常好用的。这种方式上手很快，不过长期来说维护会比较麻烦一些。

Android
如果使用 Android Studio，可以导入项目 GoLangInstalldir/src/golang.org/x/mobile/example/bind/android，打开 build.grade (hello 模块)文件，更新一下 GOPATH 和 GO 的路径，下面是我的文件内容(我是用 Homebrew 安装的 GoLang):


![使用 Go 进行 iOS 和 Android 编程](http://static.codeceo.com/images/2015/12/09113753_zscg.png)

同步 Gradle 后，应用就可以部署到仿真器或者真实设备上了。

注意: 当前这种方式只支持基于 ARM 的设备和仿真器。

让我们看一下 Java 和 Go 的代码：
```
MainActivity.java

package org.golang.example.bind;
import android.app.Activity;
import android.os.Bundle;
import android.widget.TextView;
import go.hello.Hello;

public class MainActivity extends Activity {

    private TextView mTextView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mTextView = (TextView) findViewById(R.id.mytextview);

        // Call Go function.
        String greetings = Hello.Greetings("Android and Gopher");
        mTextView.setText(greetings);
    }}
src/golang.org/x/mobile/example/bind/hello/hello.go
```
```
package hello
import "fmt"

func Greetings(name string) string {
    return fmt.Sprintf("Hello, %s!", name)
}
```
通过 import go.hello.Hello 来 import 对应的 go 文件，文件里的 Greetings 函数在 Java 代码里可以通过 Hello.Greetings 来调用。并不需要太复杂的步骤，在go 函数和 native 的 UI 元素之间就可以建立上绑定关系。

iOS
把一个 iOS 应用和 Go 程序直接进行绑定需要不同的步骤。首先需要运行下面的命令：
```
cd GoLang_Install_dir/src/golang.org/x/mobile/example/bind
gomobile bind -target=ios golang.org/x/mobile/example/bind/hello
```
这样会在当前目录下创建一个叫 Hello.framework 的 bundle，在项目里可以使用它。

在 Xcode 打开例子中的 iOS 项目，位于 GoLangInstalldir/src/golang.org/x/mobile/example/bind/ios/bind.xcodeproj ，把 Hello.framework 拖到项目里，如果需要，选择”Copy items”。目录结构现在看上去是下面这样：

![使用 Go 进行 iOS 和 Android 编程](http://static.codeceo.com/images/2015/12/09113753_dugh.png)

构建和运行这个应用（更像 Android 应用），我们可以看到在 Objective-C 代码里进行 Go 函数的调用。

看一下现在的代码：
```
#import "ViewController.h"
#import "hello/Hello.h"  // Gomobile bind generated header file in hello.framework

@interface ViewController ()
@end

@implementation ViewController

@synthesize textLabel;

- (void)loadView {
    [super loadView];
    textLabel.text = GoHelloGreetings(@"iOS and Gopher");
}

@end
#import “hello/Hello.h”导入了之前生成的 framework，textLabel.text = GoHelloGreetings(@”iOS and Gopher”);调用了它暴露出的一个函数来设置一个 label 的值。
```
也可以使用同样是自动生成的基于 Swift 的项目里的 Objective-C 的 framework，像下面这样：
```
let msg = Hello.GoHelloGreetings("gopher")
```
是否值得？
嗯，简单的说可能是不值得。如果你已经在使用 Go 来写应用了，并且不在乎应用是否 native 的，那么你可以放开手继续做，因为你已经知道了构建和部署用 Go 写的 native 应用是很简单的。如果你打算花更多的精力尝试一下绑定，你可以走的更远一些，不过还是需要稍微控制一下。

如果你没在用 Go，那么就不太值的现在就在开发 native 的移动应用时考虑 Go。不过我有很强烈的预感，在不久的将来，Go 会成为这方面很有潜力的选择的。最后欢迎你的建议和意见。

