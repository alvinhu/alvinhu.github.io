---
layout: post
title: "快速简单自定义UINavigationController中返回按钮的样式和文字"
date: 2012-11-28 17:40
comments: true
categories: iOS
keywords: iPhone, iOS, UINavigationController, custom back button, UIAppearance, 自定义返回按钮
description: iOS5下通过UIAppearance自定义返回按钮的样式，通过Category自定义返回按钮的文字
---
在开发iPhone应用程序的时候，很多时候都会用到UINavigationController，其中的返回按钮（Back Button）里的文字会自动调用前一个UIViewController的标题。而且在早期iOS版本的时候想自定义返回按钮比较麻烦。好在iOS5出来以后提供了UIAppearance以方便开发者自定义iOS的风格样式，这个时候自定义返回按钮的样式就比较容易了：

``` objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    // 自定义UIBarButtonItem返回按钮背景图片
    UIImage *backButton = [[UIImage imageNamed:@"BackButton.png"] resizableImageWithCapInsets:UIEdgeInsetsMake(0, 15.0, 0, 5.0)];
    UIImage *backButtonHighlighted = [[UIImage imageNamed:@"BackButtonHighlighted.png"] resizableImageWithCapInsets:UIEdgeInsetsMake(0, 15.0, 0, 5.0)];
    [[UIBarButtonItem appearance] setBackButtonBackgroundImage:backButton forState:UIControlStateNormal barMetrics:UIBarMetricsDefault];
    [[UIBarButtonItem appearance] setBackButtonBackgroundImage:backButtonHighlighted forState:UIControlStateHighlighted barMetrics:UIBarMetricsDefault];
    
    // 自定义UIBarButtonItem返回按钮标题文字位置
    [[UIBarButtonItem appearance] setBackButtonTitlePositionAdjustment:UIOffsetMake(-1.0, 0) forBarMetrics:UIBarMetricsDefault];
    
    return YES;
}
```

上面的代码自定义了返回按钮的样式。

如果我们想把返回按钮里面的文字改成`返回`，简单的办法就是：

``` objc
- (void)viewDidLoad
{
    [super viewDidLoad];
    
    UIBarButtonItem *backButton = [[UIBarButtonItem alloc] init];
    backButton.title = @"返回";
    self.navigationItem.backBarButtonItem = backButton;
}
```

这里我们假设打开了ARC，所以不需要`[backButton release]`。

上面的办法只是一次性定义返回按钮的文字，如果需要把所有返回按钮的文字都改成`返回`，那就需要用到`Category`：

``` objc UINavigationItem+MyBackButton.h
@interface UINavigationItem (MyBackButton)

@end
```

``` objc UINavigationItem+MyBackButton.m
#import "UINavigationItem+MyBackButton.h"

@implementation UINavigationItem (MyBackButton)

- (UIBarButtonItem *)backBarButtonItem
{
    return [[UIBarButtonItem alloc] initWithTitle:@"返回" style:UIBarButtonItemStylePlain target:nil action:nil];
}
@end
```

把上面两个文件拖到项目里面就可以了。是不是很简单！