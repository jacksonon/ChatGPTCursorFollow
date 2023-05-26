仿写iOS GPT 应用流式输出 + 可追随光标
也没什么代码，就直接传了。

关于分行头像展示问题，我的方案是：
1. 使用textview同等字体、间距文本属性，就散文本高度，放置图像在左侧即可。

关于光标追随，我的方案是：
1. 如下代码

```
#import "ViewController.h"

@interface ViewController ()
@property (nonatomic, strong) UITextView *textView;
@property (nonatomic, strong) UIImageView *cursorView;
@property (nonatomic, copy) NSString *curNews;
@end

@implementation ViewController

static int cur = 0;

- (void)viewDidLoad {
    [super viewDidLoad];

    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"news" ofType:@"txt"];
    NSError *error = nil;
    NSString *fileContents = [NSString stringWithContentsOfFile:filePath encoding:NSUTF8StringEncoding error:&error];

    if (error) {
        NSLog(@"Error reading file: %@", error.localizedDescription);
    } else {
        NSLog(@"File contents: %@", fileContents);
        self.curNews = fileContents;
    }


    self.textView = [[UITextView alloc] init];
    self.textView.font = [UIFont systemFontOfSize:16];
    self.textView.backgroundColor = [UIColor whiteColor];
    self.textView.editable = NO;
    self.textView.textContainerInset = UIEdgeInsetsMake(64, 20, 0, 20);//UIEdgeInsetsMake(64, 20, 20, 20);
    self.textView.translatesAutoresizingMaskIntoConstraints = NO;
//    self.textView.textContainer.lineFragmentPadding = 0;
    [self.view addSubview:self.textView];
    [NSLayoutConstraint activateConstraints:@[
        [self.textView.topAnchor constraintEqualToAnchor:self.view.topAnchor constant:0],
        [self.textView.leadingAnchor constraintEqualToAnchor:self.view.leadingAnchor constant:0],
        [self.textView.bottomAnchor constraintEqualToAnchor:self.view.bottomAnchor constant:-33],
        [self.textView.trailingAnchor constraintEqualToAnchor:self.view.trailingAnchor constant:0]
    ]];

    self.cursorView = [[UIImageView alloc] initWithFrame:CGRectMake(0, 0, 0, 0)];
    self.cursorView.backgroundColor = [UIColor blueColor];
    self.cursorView.layer.masksToBounds = YES;
    self.cursorView.layer.cornerRadius = 15/2.0;
    [self.view addSubview:self.cursorView];

    // 每隔1秒钟，将kOUTEXT内容输出到textView
    [NSTimer scheduledTimerWithTimeInterval:0.5 target:self selector:@selector(addText) userInfo:nil repeats:YES];
}

- (void)addText {
    // 每0.5秒打印一个字
    if (cur < [self.curNews length]) {

        // 0.1 - 0.3 的随机数
        float t = (arc4random() % 3 + 1) / 10.0;
        [NSThread sleepForTimeInterval:t];

        int read = arc4random() % 15;
        
        if ((cur + read) > self.curNews.length) {
            read = (int)self.curNews.length - cur;
        }

        // 滚动到底部
        [self.textView insertText:[self.curNews substringWithRange:NSMakeRange(cur, read)]];
        
        
        

        // 在文本输入过程中将其添加到光标位置
        UITextRange *selectedRange = self.textView.selectedTextRange;
        CGRect cursorRect = [self.textView caretRectForPosition:selectedRange.start];
        cursorRect = CGRectInset(cursorRect, 0, 3); // 将光标位置略微调整，以使其与文本对齐
        cursorRect.size = CGSizeMake(15, 15);
        
        // // 当文本内容超出textView.rect.size.height
        if (cursorRect.origin.y > CGRectGetMaxY(self.textView.frame)) {
            
            // 文本仅设置便宜
            /*
             NSRange range = NSMakeRange(self.textView.text.length -1 , 1);
             [self.textView scrollRangeToVisible:range];
             */
            [self.textView setContentOffset:CGPointMake(0, self.textView.contentSize.height - self.textView.frame.size.height) animated:NO];

            
            cursorRect.origin.y = CGRectGetMaxY(self.textView.frame) - 15 - 2; // 底间距0 + 光标本身15 + 光标上移2
            
            NSLog(@"当前光标Y值 -> %f (%f)", cursorRect.origin.y, CGRectGetMaxY(self.textView.frame));
        }
        
        [UIView animateWithDuration:0.1 animations:^{
            self.cursorView.alpha = 0.3;
            self.cursorView.frame = CGRectOffset(cursorRect, self.textView.frame.origin.x, self.textView.frame.origin.y);
            
        } completion:^(BOOL finished) {
            [UIView animateWithDuration:0.05 animations:^{
                self.cursorView.alpha = 0.8;
            }];
        }];

        // cur 在 1 -3 中随机增加
        cur += read;
    } else {
        cur = 0;
        self.textView.text = @""; // 清空
    }

}

@end

```
