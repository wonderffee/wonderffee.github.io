---
layout: post
title: "iOS6下UITextField退格变清空问题的解决方法"
date: 2013-08-07 19:39
comments: true
categories: 
---

自己的项目中一直有一个必现的UITextField退格变清空问题，测试发现只有iOS6下有，其它iOS版本都没有问题，基本上可以确定与iOS6有关。由于问题不是很大又忙就搁置了很久，今天实在不能忍受便花了点时间解决此问题，记录一下吧。


这个问题是在某个有默认值的UITextField上是必现的，一旦用退格键删除最后一个字符或中间某个字符，整个UITextfield的内容被清空了，一直百思不得其解，google了一下发现有人遇到类似的问题，并给出了复现步骤([链接](http://blog.csdn.net/kindazrael/article/details/8075245))，如下：
<!--more-->


有一个secureTextEntry为Yes的UITextField和 一个普通的UITextField，重现步骤：     
1. 点击普通的UITextField输入类容，        
2. 点击密码UITextField输入内容，    
3. 点击普通的UITextField重新获得焦点，    
4. 接着点击键盘上的退格键，    
结果：这时会发现普通的UITextField被清空了。     


测试了一下，随便找一个有用户名与密码登录页面的应用，在iOS6下就会复现出这个问题，比如iPhone自带的邮件app。但这个重现步骤说得并不准确，第1步和第2步的前提是对应的UITextField已经有内容，虽然跟自己的复现方法有点不一样，不过总算知道这个问题是怎么回事了。怎么说呢，这应该是iOS6的UITextField的一个新特性引入的问题，原文把这个问题称之为“iOS 6 Secure密码UITextField造成非密码UITextField退格清空Bug”，但是iOS6.0就有这个问题，iOS6.1.3都没有解决，估计apple没把它当bug，坑爹啊。


咱们拿apple没办法，那就只有想办法绕过去啊，初步的思路是截获退格键删除行为，每点一次退格键删除时只允许删除一个字符。这就要用到UITextFieldDelegate的一个textField:shouldChangeCharactersInRange:replacementString方法，这个方法是在UITextField的内容改变时调用，第二个参数表明内容改变的范围，第三个参数是替代的字符串。代码如下：

{% codeblock Here's Code lang:objc %}
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string
{
    if (range.location > 0 && range.length == 1 && string.length == 0)
    {
        textField.text = [textField.text substringToIndex:textField.text.length - 1];
        return NO;
    }
    return YES;
}
{% endcodeblock %}


设置一下UITextField的delegate就可以调用到此方法，range.length == 1 && string.length == 0就是删除一个字符时所满足的条件，如果条件满足，就只让textField的内容减少一个字符。初步测试，可以解决退格变清空的问题，但是又带来另一个问题：如果是在文字中间点退格键删除文字，就变成从文字的最后删除一个字符。这并不是我们想要的，怎么办呢，那就要找到删除的字符的位置，这个[链接](http://stackoverflow.com/questions/16765334/ios-6-uitextfield-secure-how-to-detect-backspace-clearing-all-characters)里给了一个终极解决办法，代码如下：

{% codeblock Here's Code lang:objc %}
- (BOOL)textField:(UITextField *)textField shouldChangeCharactersInRange:(NSRange)range replacementString:(NSString *)string
    {
        if (range.location > 0 && range.length == 1 && string.length == 0)
        {
            // Stores cursor position
            UITextPosition *beginning = textField.beginningOfDocument;
            UITextPosition *start = [textField positionFromPosition:beginning offset:range.location];
            NSInteger cursorOffset = [textField offsetFromPosition:beginning toPosition:start] + string.length;

            // Save the current text, in case iOS deletes the whole text
            NSString *text = textField.text;

            // Trigger deletion
            [textField deleteBackward];


            // iOS deleted the entire string
            if (textField.text.length != text.length - 1)
            {
                textField.text = [text stringByReplacingCharactersInRange:range withString:string];

                // Update cursor position
                UITextPosition *newCursorPosition = [textField positionFromPosition:textField.beginningOfDocument offset:cursorOffset];
                UITextRange *newSelectedRange = [textField textRangeFromPosition:newCursorPosition toPosition:newCursorPosition];
                [textField setSelectedTextRange:newSelectedRange];
            }
            return NO;
        }
        return YES;
    }
{% endcodeblock %}


思路是先取到光标位置，把文本内容暂存，清空原来UITextField的内容，然后将暂存的文本内容中光标左侧的文字replace掉重新赋给UITextField，并恢复光标位置，难点在于怎么获取光标位置与恢复光标位置。


一般用到用户名与密码UITextField的地方用这个方法就能解决问题了，只是还有点小问题，即如果不是从中间删除一个字符，而是一次删除若干个字符还是有问题的，这是由if中的range.length == 1条件限制的，稍加改造下应该就能解决。另外一个问题是如果有中文字符或其它UTF8字符，估计得考虑一下是不是应该要用“text.length - 1”，不过能有多少应用的用户名或密码带中文呢


#### 参考：    
* [iOS 6 Secure密码UITextField造成非密码UITextField退格清空Bug](http://blog.csdn.net/kindazrael/article/details/8075245)
* [Backspace functionality in iOS 6 & iOS 5 for UITextfield with 'secure' attribute](http://stackoverflow.com/questions/14400724/backspace-functionality-in-ios-6-ios-5-for-uitextfield-with-secure-attribute)
* [iOS 6 UITextField Secure - How to detect backspace clearing all characters?](http://stackoverflow.com/questions/16765334/ios-6-uitextfield-secure-how-to-detect-backspace-clearing-all-characters)

