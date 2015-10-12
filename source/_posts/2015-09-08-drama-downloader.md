title: 美剧自动保存到百度盘
date: 2015-09-08 11:45:07
tags: [programming, python, 美剧, 百度盘]
categories: python
---

现在习惯每周的美剧都用百度盘的离线下载，然后回家在Pad或者电脑看。

这样每周看美剧是否更新，把链接复制到离线下载就成了机械工作，所以写了个脚本来自动化搞定。

Github: <https://github.com/goorockey/drama-downloader>

<!--more-->

## 流程

- 每天定时到指定网址检查美剧的更新
- 解析出更新链接
- 添加百度盘的离线任务

## 依赖

- 百度盘的操作，用了[ly0/baidupcsapi](https://github.com/ly0/baidupcsapi)
- 自动识别验证码，用了[Cloudsight](http://cloudsightapi.com)
- 解析链接，用了[XPATH](https://en.wikipedia.org/wiki/XPath)
- python生成exe，用了[pyinstaller](https://github.com/pyinstaller/pyinstaller/wiki)

## 遇到的问题

用pyinstaller生成exe后，运行会报`SSLError: no such file or directory`，原因是requests库的SSL证书没找到。

参考<https://github.com/kennethreitz/requests/issues/557>，把requests的`cacert.pem`放到代码目录，并在代码开头加上以下代码解决问题:

    os.environ['REQUESTS_CA_BUNDLE'] = os.path.join(
      os.path.dirname(os.path.abspath(__file__)),
      'cacert.pem'
    )
