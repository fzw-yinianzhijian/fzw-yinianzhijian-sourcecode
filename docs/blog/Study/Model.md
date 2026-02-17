---
title: 模型入门
tags:
  - Study
createTime: 2026/02/05 09:05:00
permalink: /blog/Study/Model/
article: false
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
    --max-model-len 4096
```


??太神奇了 小伙立正了，怎么我在本机能访问的？
感觉发现问题之后就一下子清醒了。

原来这里的终端会自动把东西 ssh 过来吗 太神秘了吧。

确实很清晰，在 cursor 里面点击端口。
欸 这个好玩耶
