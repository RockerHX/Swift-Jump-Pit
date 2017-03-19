# Xcode8 Swift3 使用Reveal

很久没调试UI了，由于新的工程采用全程Swift编写，可预知的是Reveal的dlib加载方式已经扑街了（实际测试确实扑街）。

晚上搜了下，貌似这玩意儿就国内玩的人多，网上教程也比较旧，基本上都是dylib的断点加载方式，对于Reveal 2以后的版本都是采用的Framework加载。


老版本（dylib）-> `~/.lldbinit`：

```zsh
command alias reveal_load_sim expr dlopen("/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/libReveal.dylib", 2)
command alias reveal_load_dev expr dlopen(NSBundle.mainBundle().pathForResource("libReveal", ofType: "dylib")!, 2)
command alias reveal_start expr NSNotificationCenter.defaultCenter().postNotificationName("IBARevealRequestStart", object: nil)
command alias reveal_stop expr NSNotificationCenter.defaultCenter().postNotificationName("IBARevealRequestStop", object: nil)
```

Reveal 2以后的新版本（Framework）-> `~/.lldbinit`：

```zsh
command alias reveal_load_sim expr dlopen("/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework/RevealServer", 2)
command alias reveal_load_dev expr dlopen(NSBundle.mainBundle().pathForResource("RevealServer", ofType: "framework")!, 2)
command alias reveal_start expr NSNotificationCenter.defaultCenter().postNotificationName("IBARevealRequestStart", object: nil)
command alias reveal_stop expr NSNotificationCenter.defaultCenter().postNotificationName("IBARevealRequestStop", object: nil)
```

折腾了半天，方法看似可行，就是跑起来没效果，貌似断点里不再支持dlopen，无奈，那只能在代码里来一发。

```Swift
#if DEBUG
    dlopen("/Applications/Reveal.app/Contents/SharedSupport/iOS-Libraries/RevealServer.framework/RevealServer", 2)
#endif
```

到目前为止，我能找到最简单，也不会侵入工程的一种方式，还不需要`.lldbinit`。