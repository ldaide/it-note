1. 下载redis源码

   ```
   https://github.com/redis/redis/archive/refs/tags/7.0.10.tar.gz
   ```

   

2. 对redis进行编译安装

   ```
   //解压redis源码文件
   tar -zxvf 7.0.10.tar.gz
   
   //进入redis目录
   cd redis-7.0.10
   
   //redis源码编译
   make 
   
   //redis安装
   make install
   ```

   

3. 利用vscode打开redis源码目录

4. 在redis源码目录下，创建.vscode文件

5. 创建.vscode/tasks.json文件，内容如下：

   ```
   {
       "tasks": [
           {
               "type": "cppbuild",
               "label": "build-redis",
               "command": "/usr/bin/make",
               "args": [
   
               ],
               "options": {
                   "cwd": "${workspaceFolder}"
               },
               "problemMatcher": [
                   "$gcc"
               ],
               "group": {
                   "kind": "build",
                   "isDefault": true
               },
               "detail": "调试器生成的任务。"
           }
       ],
       "version": "2.0.0"
   }
   ```

   

6. 创建.vscode/launch.json文件，内容如下：

   ```
   {
       "version": "0.2.0",
       "configurations": [
           {
               "name": "启动redis",
               "type": "cppdbg",
               "request": "launch",
               //redis启动服务redis-server的位置
               "program": "${workspaceFolder}/src/redis-server",
               //调试时传递的命令行参数，此处需要传入redis配置文件
               "args": ["redis.conf"],
               //设置为true时程序暂停在程序入口处
               "stopAtEntry": false,
               //项目路径
               "cwd": "${workspaceFolder}",
               "environment": [],
               "externalConsole": true,
               //指定连接的调试器
               "MIMode": "gdb",
               "setupCommands": [
                   {
                       "description": "为 gdb 启用整齐打印",
                       "text": "-enable-pretty-printing",
                       "ignoreFailures": true
                   },
                   {
                       "description": "将反汇编风格设置为Intel",
                       "text": "-gdb-set disassembly-flavor intel",
                       "ignoreFailures": true
                   }
               ],
               //与tasks.json文件中的tasks.label保持一致
               "preLaunchTask": "build-redis",
               //调试器的路径，此处是gdb的路径
               "miDebuggerPath": "/usr/local/bin/gdb"
           }
   
       ]
   }
   ```

   

   