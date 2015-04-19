title: python发邮件脚本
date: 2012-09-30
tags: python
categories: python
---

[blat]: http://www.blat.net

项目组有每天值日搞卫生和发工作日报的规定，由于不提醒容易忘记，则想到通过内部邮件定时提醒，练练手。

整个“任务”可以分为发邮件+定时两部分。


##发邮件##

由于服务器是windows系统，google得知，windows下有[blat]这发邮件的大杀器，所以刚开始是想用blat+批处理做的。

执行`blat -h`或者看官网上的帮助，使用blat发邮件还是很简单的（所以官网特别提醒不要用blat来发SPAM。。)

    blat <邮件正文文件> -from <发送地址> -to <接受地址> -subject <邮件标题> 
            -server <smtp服务器地址> -username <登录服务器用户名> -password <密码>

    blat - -body <邮件正文> -from <发送地址> -to <接受地址> -subject <邮件标题> 
            -server <smtp服务器地址> -username <登录服务器用户名> -password <密码>

blat还可以通过-install把参数保存到注册表。blat确实是自动发邮件的大杀器！

P.S 在linux实现自动发邮件，可以用msmtp,sendmail等～

这本来是很简单的，但因为任务要根据星期几发送邮件给指定的人，而且我们是12个人分成两周，所以想到用一个二维数组存放成员来实现，这也没什么问题。

问题在于在计算要提醒的人时，要根据今天离开始值日的天数，来从数组获取成员，而这求天数在windows用批处理可不好搞，因为要考虑每月不同天数和闰年啊。

当然也不是不能完成，网上也有方法:

- <http://bbs.bathome.net/thread-11128-1-1.html>
- <http://bbs.bathome.net/viewthread.php?tid=5659&highlight=%2Bbatman>
- <http://bbs.bathome.net/viewthread.php?tid=5682&highlight=%2Bbatman>

嫌麻烦，我还是决定找有现成库的方法来做，所以就想到用有各种库的python实现了。

<!-- more -->

python有[smtplib](http://docs.python.org/library/smtplib.html "smtplib")库实现smtp发邮件，核心代码也很简单：

    import smtplib
    # some code ...
    smtp = smtplib.SMTP()
    smtp.connect(server)
    smtp.login(username, password)
    smtp.sendmail(sender, receiver, msg)
    smtp.quit()

而用python计算相差的天数更是简单不过：

    import datetime
    # some code ...
    days = (datetime.datetime.now() - datetime.datetime(2012,9,30)).days


##定时##

定时在windows可以用计划任务实现

P.S 在linux可以用cron实现


##整个代码##

    #coding=utf-8
    import smtplib
    import datetime
    import sys
    
    from email.mime.text import MIMEText
    from email.header import Header
    
    
    member = (('member1', 'member2', 'member3', 'member4', 'member5', 'member6'),
            ('member7', 'member8', 'member9', 'member10', 'member11', 'member12'))
    
    suffix = '@abc.com'
    
    def send_mail(receiver, content_filename,
                sender = 'admin@abc.com',
                server = '192.168.1.1',
                username = 'admin@abc.com',
                password = 'admin'):
    
        subject = ''
        content = ''
    
        # 从文件读取邮件正文
        try:
            content_file = open(content_filename, 'r')
            try:
                subject = content_file.readline()
                content = content_file.read()
    
                # 转为utf-8
                subject = subject.decode('gbk', 'ignore').encode('utf-8')
                content = content.decode('gbk', 'ignore').encode('utf-8')
            finally:
                content_file.close()
        except IOError, e:
            sys.stderr.write("cannot open file " + content_filename)
            return
    
        content = content + "\n\n系统生成，请勿回复 :)"
        #print content
    
        # 构造邮件
        msg = MIMEText(content, 'plain', 'utf-8')
        msg['Subject'] = Header(subject, 'utf-8')
        msg['From'] = sender
        msg['To'] = receiver
    
    
        if (len(msg) > 0):
            try:
                # stmp模块发送邮件
                smtp = smtplib.SMTP()
                smtp.connect(server)
                smtp.login(username, password)
                smtp.sendmail(sender, receiver, msg.as_string())
                smtp.quit()
    
                print "Success"
                return True
    
            except Exception, e:
                print str(e)
                return False
    
    
    def get_on_duty():
        receiver = ''
        days = (datetime.datetime.now() - datetime.datetime(2012,10,8)).days
    
        if days > 0:
            days = days + 1 # 提前一天提醒
            week = (days / 7) % 2
            date = (days % 7)
    
            # 周六发周一的值日
            if date == 6:
                date = 0
    
            if week < len(member) and date < len(member[week]) and len(member[week][date]) > 1:
                receiver = member[week][date] + suffix
    
        return receiver
    
    
    if __name__ == '__main__':
    
        if len(sys.argv) > 1:
            job = sys.argv[1]
    
            # 值日
            if job == 'duty':
                receiver = get_on_duty()
                print receiver
    
                content_filename = 'duty.txt'
                if datetime.datetime.now().weekday() == 5:
                    content_filename = 'duty_Sat.txt'
    
                if len(receiver) > len(suffix):
                    send_mail(receiver = receiver, content_filename = content_filename)
    
            # 每天日报提醒
            elif job == 'daily':
                send_mail(receiver = 'partner', content_filename = 'daily_alert.txt')
