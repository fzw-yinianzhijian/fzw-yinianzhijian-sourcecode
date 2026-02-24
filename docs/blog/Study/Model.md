---
title: 模型入门
tags:
  - Study
createTime: 2026/02/05 09:05:00
permalink: /blog/Study/Model/
article: true
---

### 下载模型

这里就说怎么从 `huggingface` 上下载了，正常情况下你直接调用 `transformer` 库或者相关内容的时候他都会自己下载，会存在缓存里，具体位置在 `~/.cache/huggingface/hub/` 下，你发现里面每个模型有个 `blobs` 文件夹，确实是模型数据，但是显示的是 hash 值。

因为他反正是使用而不是给你看的会有部分重复共用的优化。

这里是下载全模型的一个例子，`snapshot_download` 下载全部模型， `hf_hub_download` 下载单个文件，需要指定 filename 名字
```python
from huggingface_hub import snapshot_download
import os

download_id='google/gemma-7b-it'
download_path='./model_weights'

snapshot_download(repo_id=download_id, local_dir=download_path, token=os.getenv('huggingface_api'))
```

另一种下载方式。
```bash
#在终端中导入环境变量的方式
set -a
source .env
set +a # 这个 set 是可以自动放入环境变量的

env | grep huggingface_api # 查看当前环境变量有没有

export HF_ENDPOINT=https://hf-mirror.com #可选，在下载之前
huggingface-cli download google/gemma-7b-it --local-dir ./model_weights --token $huggingface_api
```

一个加速小方法。
```bash
pip install hf_transfer
export HF_HUB_ENABLE_HF_TRANSFER=1
# 据说会快很多，下次有空试试。
```



### vllm 部署模型

```
python -m vllm.entrypoints.openai.api_server \
    --model ./model_weights/gemma-7b-it \
    --trust-remote-code \
    --host 0.0.0.0 \
    --port 8000 \
    --gpu-memory-utilization 0.9 \
    --max-model-len 4096 \
    --tensor-parallel-size 4 \
    --served-model-name gemma_test
```

--model 模型地址，如果是名字自动上 hugging face 上下载，默认放到 ~/.cache/huggingface/hub/ 下。

--trust-remote-code 信任远程代码，允许使用远程代码，除了权重文件以外的内容。

--host 监听地址，0.0.0.0 表示监听所有地址。

--port 监听端口，8000 表示监听 8000 端口。

--gpu-memory-utilization 显卡内存利用率，0.9 表示利用 90% 的显卡内存。

--max-model-len 模型最大长度，4096 表示最大长度为 4096。

--tensor-parallel-size `<N>` 显卡并行数量，`<N>` 表示使用 `<N>` 个显卡。

--served-model-name `<NAME>` 服务模型名称，`<NAME>` 表示服务模型名称，不然是默认名字，可能是路径啥的。

- `http://localhost:8000/v1/models` 查看当前模型列表。
提问：一个端口可以跑多个模型吗？AI 这边给我的答复是同时只能跑一个。
- `http://localhost:8000/v1/completions` 生成文本。
- `http://localhost:8000/v1/chat/completions` 生成对话。

最好的 GET 方法是直接在浏览器里输入。

```
curl http://localhost:8000/v1/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "gemma_test",
        "prompt": "San Francisco is a",
        "max_tokens": 7,
        "temperature": 0
    }'

curl http://loaclhost:8000/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "Qwen/Qwen2.5-1.5B-Instruct",
        "messages": [
            {"role": "system", "content": "You are a helpful assistant."},
            {"role": "user", "content": "Who won the world series in 2020?"}
        ]
    }'
```