#### launch.json 文件

<br/>

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "C/C++ 远程调试",
      "type": "cppdbg",
      "request": "launch",
      "program": "${fileDirname}/${fileBasenameNoExtension}", // 远程路径
      "args": [],
      "stopAtEntry": false,
      "cwd": "${fileDirname}",
      "environment": [],
      "externalConsole": false,
      "MIMode": "gdb",
      "miDebuggerPath": "/usr/bin/gdb",  // 远程 gdb 路径
      "setupCommands": [
        {
          "description": "启用整齐打印",
          "text": "-enable-pretty-printing",
          "ignoreFailures": true
        }
      ],
      "preLaunchTask": "C/C++: gcc 生成活动文件"  // 关联 tasks.json
    }
  ]
}
```