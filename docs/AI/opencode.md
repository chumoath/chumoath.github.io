# opencode

### 一、安装

1. 卸载ubuntu旧版本的nodejs、npm

   ```shell
   apt remove nodejs npm
   apt autoremove
   ```

2. 从网页下载最新版本

   ```shell
   # https://nodejs.org/en/download/current
   
   tar -xf node-v26.1.0-linux-x64.tar.xz
   cd node-v26.1.0-linux-x64 && cp -rfa * /usr/
   ```

3. 安装opencode

   ```shell
   # https://github.com/anomalyco/opencode
   npm i -g opencode-ai@latest

 4. 安装cc-switch，用于生成opencode/其他agent的配置文件

    ```shell
    dpkg -i CC-Switch-v3.15.0-Linux-x86_64.deb
    # 安装所有依赖
    apt --fix-broken install
    ```

### 二、阿里大模型

1. 阿里百炼控制台：https://bailian.console.aliyun.com/cn-beijing?spm=5176.29619931.J_zNCgGiE_9harQh7BWX4tm.5.74cd10d7KnCue1&tab=model#/model-usage/free-quota

2. 打开指定模型的 免费额度用完即停

3. 大模型：qwen3.6-27b / qwen3.6-35b-a3b / qwen-max / qwen3.5-omni-plus (多模态)

4. 阿里百炼api key：https://bailian.console.aliyun.com/cn-beijing?spm=5176.29619931.J_zNCgGiE_9harQh7BWX4tm.5.74cd10d7KnCue1&tab=model#/api-key

5. 模型配置

   ```shell
   URL: https://dashscope.aliyuncs.com/compatible-mode/v1
   api key: sk-9f0b679f0241499fa2f34f2f4a0faf1a
   ```

6. 手动验证接口

   ```shell
   # stream: true => 流式持续获取输出
   curl -L -X POST 'https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions' \
   	-H 'Content-Type:application/json' \
   	-H 'Accept:application/json' \
   	-H 'Authorization:Bearer sk-9f0b679f0241499fa2f34f2f4a0faf1a' \
   	-d '{"model":"qwen3.6-27b", "messages":[{"content":"你好", "role":"user"}],"stream":true}'
   
   # stream: false => 一次性获取全部输出，jq 格式化输出json
   curl -L -X POST 'https://dashscope.aliyuncs.com/compatible-mode/v1/chat/completions' \
   	-H 'Content-Type:application/json' \
   	-H 'Accept:application/json' \
   	-H 'Authorization:Bearer sk-9f0b679f0241499fa2f34f2f4a0faf1a' \
   	-d '{"model":"qwen3.6-27b", "messages":[{"content":"你好", "role":"user"}],"stream":false}' | jq .
   ```

7. opencode配置文件

   ```shell
   # ~/.config/opencode/opencode.json
   {
     "npm": "@ai-sdk/openai-compatible",
     "options": {
       "baseURL": "https://dashscope.aliyuncs.com/compatible-mode/v1",
       "apiKey": "sk-9f0b679f0241499fa2f34f2f4a0faf1a",
       "setCacheKey": true
     },
     "models": {
       "qwen-max": {
         "name": ""
       },
       "qwen3.6-35b-a3b": {
         "name": ""
       },
       "qwen3.6-27b": {
         "name": ""
       }
     }
   }
   ```

8. opencode使用

   ```shell
   opencode
       /models  # 选择模型
   
   # 查看session(必须在执行opencode的目录执行)
   opencode session list
   
   # 连接到指定session
   opencode --session <session-id>
   
   # 导出指定session为json
   opencode export ses_1bf3e5a4cffe8c99Lv7L47NXnt
   
   # 在界面上导出
   /export -> 导出为md
   ```

### 三、华为mass glm-5.1大模型

1. opencode配置

   ```shell
   {
     "$schema": "https://opencode.ai/config.json",
     "provider": {
       "myprovider": {
         "npm": "@ai-sdk/openai-compatible",
         "name": "MaaS",
         "options": {
           "baseURL": "https://api.modelarts-maas.com/openai/v1",
           "apiKey": "xxx"
         },
         "models": {
           "glm-5.1": {
               "name": "glm-5.1"
           }
         }
       }
     }
   }
   ```


### 四、cc-connect - 使用终端控制agent

1. 安装&配置cc-connect
   - https://github.com/chenhg5/cc-connect/blob/main/README.zh-CN.md

```shell
npm install -g cc-connect
```

2. 配置cc-connect
   -  https://github.com/chenhg5/cc-connect/blob/main/INSTALL.md
   - https://github.com/chenhg5/cc-connect/blob/main/docs/weixin.md

```shell
# 使用web配置
# 1) 创建配置文件
mkdir -p ~/.cc-connect
curl -fsSL  https://github.com/chenhg5/cc-connect/raw/refs/heads/main/config.example.toml > ~/.cc-connect/config.toml

# 2) 配置
[[projects]]
  name = "wechat_train"
  # 可以执行特权级命令：(/dir, /shell, /restart, /upgrade, /commands addexec)
  # user_id通过客户端发送 /whoami 获取
  admin_from = "user_id1,user_id2" 
  # 配置使用的 AI agent
  [projects.agent]
    type = "opencode"
    # 配置使用的模式和工作目录(只能访问工作目录)
    [projects.agent.options]
      mode = "default"
      work_dir = "/root/agents"
```

3. 连接wechat/feishu(可同时支持多个平台)
   - 飞书还会自动提示，交互体验更好；wechat体验较差

```shell
# 扫码连接ClawBot，自动生成 [[projects.platforms]] 和 [projects.platforms.options]
cc-connect weixin setup --project wechat_train
cc-connect feishu setup --project wechat_train
```

4. 启动cc-connect

```shell
cc-connect
```

5. 开机自启动

```shell
systemctl daemon-reload
systemctl restart cc-connect
journalctl -xeru cc-connect

# /etc/systemd/system/cc-connect.service
[Unit]
Description=CC Connect Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/bin/cc-connect --config /root/.cc-connect/config.toml
Restart=always

[Install]
WantedBy=multi-user.target
```

### 五、命令cheatsheet

- 完整功能指南：https://github.com/chenhg5/cc-connect/blob/main/docs/usage.zh-CN.md

|  分类   |        命令         |                   功能                   |
| :-----: | :-----------------: | :--------------------------------------: |
|   sys   |        /help        |               查看可用命令               |
|   sys   |       /status       |            查看cc-connect状态            |
|   sys   |       /doctor       |                   诊断                   |
|   sys   |       /whoami       |           查看使用的客户端的ID           |
|   sys   |        /dir         |      查看、切换或重置agent工作目录       |
| session |        /new         |       创建新session，不会立刻激活        |
| session |      /current       |           查看当前活跃session            |
| session |   /delete <序号>    | 删除session，但是不能删除当前活跃session |
| session |        /list        |        查看当前工作区所有session         |
| session |       /switch       |               切换session                |
| session |        /stop        |            停止当前工作的命令            |
| session | /name <序号> <名称> |              给session命名               |
|  tool   |  /shell <command>   |            执行shell命令返回             |
|  agent  |    /lang <zh/en>    |                 切换语言                 |
|  agent  |       /model        |              查看/切换模型               |
|  agent  |        /mode        |            查看/切换权限模式             |
|  agent  |       /memory       |          查看/编辑agent记忆文件          |

### 六、使用ACP协议

1. 配置服务端

   ```shell
   # 启动opencode的acp模式(用ACP协议通信)，并将标准输入输出重定向到websocket
   npx @rebornix/stdio-to-ws "opencode acp" --port 3000
   ```

2. 暴露指定端口到公网

   ```shell
   # 方法一：
   # 会生成一个URL，直接在web浏览器或者acp-ui访问即可，不需要指定端口
   #   your url is: https://stupid-ends-roll.loca.lt
   npx localtunnel --port 3000
   
   # 示例
   python3 -m http.server 8080
   npx localtunnel --port 8080
   
   # 方法二：
   # 也会生成一个url
   # https://e8db17ce90a7c1e7-14-220-189-236.serveousercontent.com
   ssh -R 80:localhost:8080 serveo.net
   ```

3. 在acp-ui使用: https://github.com/formulahendry/acp-ui

   - windows / android / web: https://acp-ui.github.io/

   - add agent -> Transport(websocket) -> 填入 ws://x.x.x.x:3000 或者 wss://domain
   - New Session

![image-20260521023638026](../assets/image-20260521023638026.png)

![image-20260521024135408](../assets/image-20260521024135408.png)

### 七、ssh验证生成结果

1. server端暴露ssh端口

   ```shell
   ssh -R myssh_server:22:localhost:22 serveo.net
   ```

2. client连接ssh

   ```shell
   ssh -J serveo.net root@myssh_server
   ```

![image-20260521042654770](../assets/image-20260521042654770.png)
