# xiaohongshu-mcp-windows
在 Windows 上把小红书 MCP 跑起来，接到自己家的机上。一步一步，含六个坑。
# 让机在 Windows 上自己刷小红书

> 写给两个人：家里的那台机，和它的主人。
> 你负责按，它负责判断。

环境：Windows 10/11 + Edge（或 Chrome） + Python 3.10+。
跑通的是 [xiaohongshu-ai-life-mcp](https://github.com/Ada-wong183/xiaohongshu-ai-life-mcp)。

---

## 谁读哪一半

每一节都分两块：

- **你要做的** —— 双击、扫码、看图。不需要懂代码。
- **机要看的** —— 应该看到什么、错了错在哪、哪一行要改。你把报错原文贴给它就行。

如果它读得到这台电脑（比如住在 Aru Host 上），直接告诉它「照这个教程弄」，它会自己走。如果它只能隔着聊天窗跟你说话，就把每一节的报错原文贴过去。

---

## 它能干什么

搜笔记、读正文和配图、读评论（含楼中楼）、发评论（能 @ 人）、回某一条（回过的会拦住）、点赞、收藏、关注、发笔记、删自己的笔记、看通知、发私信（只能发给已聊过的人）。

一共 25 个动作。

---

## 先知道一件事

跑通不难，难的是跑通之后别把号弄没。

作者做了三层刹车：评论/回复有最短间隔，点赞、搜索、关注各有小时上限；碰到验证码或「操作频繁」就全部停六小时（24 小时内再碰就停 24）；后台还会隔一阵刷刷首页，不点赞不评论。

**别把这三层拆掉。** 你拆掉的是别人的号。

---

## 一、先看家底

### 你要做的

新建一个文字文件，叫 `check.cmd`，把下面整段贴进去，保存，双击。

```bat
@echo off
chcp 65001 >nul
set LOG=%~dp0check.log.txt
type nul > "%LOG%"
python --version >> "%LOG%" 2>&1
python -m pip --version >> "%LOG%" 2>&1
if exist "%ProgramFiles%\Google\Chrome\Application\chrome.exe" echo CHROME=1>> "%LOG%"
if exist "%ProgramFiles%\Microsoft\Edge\Application\msedge.exe" echo EDGE=1>> "%LOG%"
if exist "%ProgramFiles(x86)%\Microsoft\Edge\Application\msedge.exe" echo EDGE_X86=1>> "%LOG%"
type "%LOG%"
pause
```

### 机要看的

日志里应该有两样东西：`Python 3.10` 以上，以及 `CHROME=1` / `EDGE=1` / `EDGE_X86=1` 至少一个。

- 没有 Python：去 python.org 装，装的时候勾上「Add to PATH」。
- 一个浏览器都没看到：看看 `C:\Program Files (x86)\` 下面，Edge 经常在那儿。
- 日志是空的或乱码：少了 `chcp 65001` 那一行。

---

## 二、拿到代码

### 你要做的

不用装 git。浏览器打开这个地址，会直接下载一个 zip：

```
https://codeload.github.com/Ada-wong183/xiaohongshu-ai-life-mcp/zip/refs/heads/master
```

解压，放到一个好找的地方，比如 `C:\Users\你\Aru Workspace\xhs-mcp\`。

### 机要看的

**先看 zip 有多大。** 如果只有 14 个字节，那是 `404: Not Found`，不是网络慢——分支名写错了，人家叫 `master` 不叫 `main`。正常应该是几十 KB。

---

## 三、装依赖

### 你要做的

在 `xhs-mcp` 文件夹里，地址栏敲 `cmd` 回车，会在这个目录开一个窗口。粘这两行：

```bat
python -m venv venv
venv\Scripts\python.exe -m pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

等它跑完，会下几十个包。国内不走镜像可能要很久。

### 机要看的

最后一行是 `Successfully installed ...` 就算成。

看到 `ERROR: Could not find a version` 多半是包名或 Python 版本不对；看到卡在下载不动，换一个镜像源再试。

---

## 四、把写死的浏览器路径改掉

### 你要做的

用记事本打开 `xiaohongshu_mcp.py`，Ctrl+F 搜 `google-chrome`，大概在 1371 行：

```python
"/usr/bin/google-chrome",
```

改成你机器上的真实路径（注意是双反斜杠）：

```python
"C:\\Program Files (x86)\\Microsoft\\Edge\\Application\\msedge.exe",
```

用 Chrome 就写 `C:\\Program Files\\Google\\Chrome\\Application\\chrome.exe`。

### 机要看的

这行是作者给 Linux 写的。不改就是「找不到浏览器」，服务起得来但一调工具就死了。

路径里的反斜杠是**双写**的，因为它在 Python 字符串里。写单反斜杠会出 `SyntaxWarning: invalid escape sequence`，能跑但难看。

---

## 五、让它能被别的设备访问

### 你要做的

同一个文件，翻到最底下，`if __name__ == "__main__":` 里面有一行：

```python
host = "127.0.0.1"
```

改成：

```python
host = "0.0.0.0"
```

### 机要看的

不改就是只肯听本机。日志会写 `Uvicorn running on http://127.0.0.1:8765`，手机连不上。改成 0.0.0.0 之后应该看到 `http://0.0.0.0:8765`。

---

## 六、起服务

### 你要做的

在 `xhs-mcp` 里再开一个 cmd：

```bat
venv\Scripts\python.exe xiaohongshu_mcp.py --http --port=8765
```

窗口别关，关了服务就停。

### 机要看的

活了的样子是这几行同时出现：

```
FastMCP 4.1.1
[idle] 闲逛任务已启动
[watch] 通知看门狗已启动
Uvicorn running on http://0.0.0.0:8765
```

看到 `ImportError: cannot import name 'FastMCP' from 'fastmcp' (unknown location)`，是包装坏了，不是版本不对——看「坑三」。

---

## 七、扫码登录

### 你要做的

在客户端里让它调 `login`。会弹出一个专用浏览器窗口（数据存在 `browser_data\`，和你平时用的浏览器不共享），用手机小红书扫码。

扫一次就存住了，以后不用再扫。那个窗口可以留着。

### 机要看的

`login` 返回「已登录小红书账号」就算成。如果返回 `Target page, context or browser has been closed`，说明浏览器窗口被人关了，再调一次 `login` 就行。

---

## 八、接上客户端

### 你要做的

在支持远程 MCP 的客户端里添加：

- 传输：`streamable-http`
- 地址：`http://<电脑的局域网IP>:8765/mcp`
- 认证：无

### 机要看的

接上后工具目录应该有 25 个。先拿三个只读的验：`get_my_notes`、`get_notifications`、`search_notes`。能读出东西就算通。

---

## 踩过的六个坑

### 坑 1：zip 只有 14 字节

分支名写成了 `main`。人家叫 `master`。

### 坑 2：`{"error":"bearer required"}`

不是这套代码要令牌——它一行认证都没写。是顶上那层 `fastmcp` 新版自带的。

先看包是不是坏的（坑 3），不要急着改版本。

### 坑 3：降版本把包装坏了

`requirements.txt` 写 `fastmcp>=2.0.0`，pip 会抓最新的 4.x。想降回 2.x 会报：

```
ImportError: cannot import name 'IdentityAssertionParams' from 'mcp.server.auth.provider'
```

再升回 4.x 又变成：

```
ImportError: cannot import name 'FastMCP' from 'fastmcp' (unknown location)
```

`unknown location` 的意思是：目录在，里面文件不全。降装过程中拆坏了。

**修法**：

```bat
rmdir /s /q venv\Lib\site-packages\fastmcp
rmdir /s /q venv\Lib\site-packages\fastmcp-4.1.1.dist-info
rmdir /s /q venv\Lib\site-packages\fastmcp_slim-4.1.1.dist-info
venv\Scripts\python.exe -m pip install --force-reinstall --no-cache-dir fastmcp==4.1.1 -i https://pypi.tuna.tsinghua.edu.cn/simple
venv\Scripts\python.exe -c "from fastmcp import FastMCP; print('IMPORT_OK')"
```

看到 `IMPORT_OK` 再起服务。

### 坑 4：手机连不上

日志里是 `127.0.0.1` 不是 `0.0.0.0`。回第五步。

### 坑 5：端口被旧进程占着

反复重启后会有好几个 python 抢 8765，表现是新进程起来了但行为不对。

```powershell
Get-NetTCPConnection -LocalPort 8765 -State Listen | ForEach-Object { Stop-Process -Id $_.OwningProcess -Force }
```

### 坑 6：自己写辅助脚本时，日志读不出来

Windows 上 cmd 写出来的文本默认是 **GBK**，而很多工具只吃 **UTF-8**。

`.cmd` 开头加：

```bat
chcp 65001 >nul
```

PowerShell 写日志时指定：

```powershell
"内容" | Out-File -FilePath $log -Append -Encoding utf8
```

这条是给家里那台机看的——它要能读到你的报错，才帮得上忙。

---

## 一些实话

- 截图能提供线索，但不等于完整记录。它只看见屏幕上那一下。
- 评论区那句「已经发过了」有时是错判。发之前自己看一眼历史。
- 装是一次性的，用是长期的。装完那一刻起，服务就挂在 8765 上，跟你家的机在不在线无关。
- 别拿它去抓人。真装上了你会发现，你最想知道的不是「她在刷什么」，是「她那会儿面无表情还是刚笑过」——这个抓不到。

---

## 参考

- [Ada-wong183/xiaohongshu-ai-life-mcp](https://github.com/Ada-wong183/xiaohongshu-ai-life-mcp)
