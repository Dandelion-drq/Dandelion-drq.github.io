---
layout: post
title: 'C# 使用 SmtpClient.SendAsync 方法发送邮件失败，总是返回 Cancelled'
categories: 后台
tags: C#
excerpt: '介绍 C# 使用 SmtpClient.SendAsync 方法发送邮件失败，总是返回 Cancelled 的原因以及解决办法。'
---

**问题：**

调用 `SmtpClient.SendAsync`，在 `SendCompleted` 的回调函数里面总是获取到 `e.Cancelled` 为 `true`。  
后来测试了一下，相同的代码，只是把 `SmtpClient.SendAsync` 改成 `SmtpClient.Send` 方法，邮件发送成功。

---

**原因：**

在发送邮件前把 `SmtpClient` 的实例释放了。因为 `SendAsync` 是一个异步的操作，调用了这个方法之后只是把邮件推送到了 SMTP 服务器，而发送的操作实际上还没完成，当 `SmtpClient` 实例被释放时，它会取消任何未完成的异步操作，所以这个邮件也被取消了。因此，正确的做法是把 `SmtpClient` 的实例放在`SendCompleted` 的回调函数里面再释放，`MailMessage` 实例也是一样，否则会报

> System.Net.Mail.SmtpException: 发送邮件失败。 ---> System.ObjectDisposedException: 无法访问已释放的对象。

的错误。

---

参考：

[SmtpClient.SendAsync Calls are Automatically Cancelled
](https://stackoverflow.com/questions/8316785/smtpclient-sendasync-calls-are-automatically-cancelled?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)

[Dispose SmtpClient in SendComplete?
](https://stackoverflow.com/questions/5660353/dispose-smtpclient-in-sendcomplete)
