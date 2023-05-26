![](https://github.com/kiwiwow/ChatGPTCursorFollow/blob/main/RPReplay_Final1685067589.gif?raw=true)

简单写了下 iOS GPT 应用流式输出 + 可追随光标
也没什么代码，就直接传了。

对于我想要实现的方案来说，整体就是一个textView，关于光标或者头像位置，则采用计算方式放置。

关于分行头像展示问题，我的方案是：
1. 使用textview同等字体、间距文本属性，计算文本高度，放置图像在左侧即可。插入到TextView持有的滚动视图指定位置。

关于光标追随，我的方案是：
1. 如下代码

```

- (void)viewDidLoad {
    [super viewDidLoad];

    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"news" ofType:@"txt"];
    NSError *error = nil;
    NSString *fileContents = [NSString stringWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:&error];
    if (error) {
        NSLog(@"Error reading file: %@", error.localizedDescription);
    } else {
        self.curNews = fileContents;
    }

    self.textView = [[UITextView alloc] init];
    self.textView.font = [UIFont systemFontOfSize:16];
    self.textView.backgroundColor = [UIColor whiteColor];
    self.textView.editable = NO;
    self.textView.textContainerInset = UIEdgeInsetsMake(64, 20, 0, 20);
    self.textView.translatesAutoresizingMaskIntoConstraints = NO;
    [self.view addSubview:self.textView];
    [NSLayoutConstraint activateConstraints:@[
        [self.textView.topAnchor constraintEqualToAnchor:self.view.topAnchor constant:0],
        [self.textView.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor constant:0],
        [self.textView.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor constant:-33],
        [self.textView.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor constant:0]
    ]];

    self.cursorView = [[UIImageView alloc] initWithFrame:CGRectZero];
    self.cursorView.backgroundColor = [UIColor blueColor];
    self.cursorView.layer.masksToBounds = YES;
    self.cursorView.layer.cornerRadius = 15/2.0;
    [self.view addSubview:self.cursorView];

    // 每隔0.5秒钟，将内容输出到textView
    [NSTimer scheduledTimerWithTimeInterval:0.5 target:self selector:@selector(addText) userInfo:nil repeats:YES];
}

- (void)addText {
    if (cur < [self.curNews length]) {

        float t = (arc4random() % 3 + 1) / 10.0;
        [NSThread sleepForTimeInterval:t];
        int read = arc4random() % 15;
        if ((cur + read) > self.curNews.length) {
            read = (int)self.curNews.length - cur;
        }
        [self.textView insertText:[self.curNews substringWithRange:NSMakeRange(cur, read)]];
        
        UITextRange *selectedRange = self.textView.selectedTextRange;
        CGRect cursorRect = [self.textView caretRectForPosition:selectedRange.start];
        cursorRect = CGRectInset(cursorRect, 0, 3);
        cursorRect.size = CGSizeMake(15, 15);
        if (cursorRect.origin.y > CGRectGetMaxY(self.textView.frame)) {
            [self.textView setContentOffset:CGPointMake(0, self.textView.contentSize.height - self.textView.frame.size.height) animated:NO];
            cursorRect.origin.y = CGRectGetMaxY(self.textView.frame) - 15 - 2;
        }
        
        [UIView animateWithDuration:0.1 animations:^{
            self.cursorView.alpha = 0.3;
            self.cursorView.frame = CGRectOffset(cursorRect, self.textView.frame.origin.x, self.textView.frame.origin.y);
        } completion:^(BOOL finished) {
            [UIView animateWithDuration:0.05 animations:^{
                self.cursorView.alpha = 0.8;
            }];
        }];

        cur += read;
    } else {
        cur = 0;
        self.textView.text = @""; // 清空
    }

}

@end

```
