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
   opencode attach <session-id>
   
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

   

