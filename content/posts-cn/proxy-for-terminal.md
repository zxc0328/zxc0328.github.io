---
title: "终端工具翻墙不完全指南"
date: 2017-03-26 22:10:22
tags: 
- Terminal
categories: 
- Terminal
---


写这篇文章的动机是之前Github曾经短暂的被墙过，这样的话，如果终端没有翻墙，那就没法推代码了。之后几天国内访问Github也一直很慢，于是尝试了给终端翻墙。网上的文章很多，但没有特别满意的，因此决定自己写一篇。

<!-- more -->

说是不完全指南，因为我这篇文章针对的是macOS下使用Shadowsocks翻墙的用户来说的。当然其他其他的系统，如果是使用Shadowsocks，道理应该差不多。

**预备工作**：安装Shadowsocks客户端，配置好服务器。搞清楚Shadowsocks在本地运行的端口，一般是1080或者1086。

### Homebrew翻墙

macOS下装软件要用Homebrew，但Homebrew的源在国外，国内用是很慢的。首先我们要让Homebrew翻墙，才能顺利的往下进行其他工作。

Homebrew下载用的是curl。因此我们只要配置curl使用代理就可以了。

curl的代理在`~/.curlrc`

我们在这里加一行：

`socks5 = “127.0.0.1:1080”` 

1080是之前说的Shadowsocks的本地端口，下文就不再说明了。

### 安装Polipo

`brew install polipo`

装好后记得把之前加上的curl的代理删掉。

然后配置polipo，修改`/usr/local/opt/polipo/homebrew.mxcl.polipo.plist`设置`parentProxy`:

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>homebrew.mxcl.polipo</string>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/local/opt/polipo/bin/polipo</string>
        <string>socksParentProxy=localhost:1080</string>
    </array>
    <!-- Set `ulimit -n 20480`. The default OS X limit is 256, that's
         not enough for Polipo (displays 'too many files open' errors).
         It seems like you have no reason to lower this limit
         (and unlikely will want to raise it). -->
    <key>SoftResourceLimits</key>
    <dict>
      <key>NumberOfFiles</key>
      <integer>20480</integer>
    </dict>
  </dict>
</plist>
```

增加了`<string>socksParentProxy=localhost:1080</string>`

然后配置polipo开机自启动：

```
launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.polipo.plist
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.polipo.plist
```

这样polipo就设置完成了，polipo的http代理默认在`localhost:8123`。

### NPM翻墙

```
npm config set proxy http://127.0.01:8123
npm config set https-proxy http://127.0.0.1:8123
```

### Git翻墙

```
git config --global http.proxy http://localhost:8123
git config --global https.proxy http://localhost:8123
```

可以用`git config --list`查看是否设置成功。

### 结语

上面讲了NPM和Git的代理。其他工具的话，道理是一样的，如果直接支持socks5代理，那是最好。一般都支持Http代理，比如Genymotion。遇到工具网络请求很慢的情况，设置具体工具的代理就可以了。我之前以为在终端设置`export http_proxy=http://localhost:8123`就可以。结果并不是这样，并不是所有的工具都走这个终端的代理。

终端也有了代理，从此我们就可以愉快的进行开发了。妈妈再也不用担心我的cnpm出什么问题了，推代码到Github的心情也更轻松了，连更新博客都勤快了很多呢！

### 参考链接

+ [为终端设置Shadowsocks代理](http://droidyue.com/blog/2016/04/04/set-shadowsocks-proxy-for-terminal/)

