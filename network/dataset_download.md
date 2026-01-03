# hf download
```bash
hf download ShapeNet/ShapeNetCore --repo-type=dataset \
    --local-dir=./ --cache-dir=./ --max-workers 1

conda deactivate
hf download ShapeNet/ShapeNetCore-archive --repo-type=dataset \
    --local-dir=./ --cache-dir=./
```

🪫: env.python >=3.9 is required by hf cli

## 卡住
* 之前的进程未kill
```bash
ps aux | grep hf
```


```python
远端
 ↓
cache-dir        ← 断点续传、校验
 ↓
local-dir        ← 你实际使用的数据
```

* 卡住过一会，就recover normality了

# snapshot_download
```
from huggingface_hub import snapshot_download

snapshot_download(
    repo_id="ShapeNet/ShapeNetCore",
    repo_type="dataset",
    local_dir="./ShapeNetCore",
    max_workers=4,
    allow_patterns="*",  # 可以也只下部分文件
```

# aria2c
* replace curl+wget
```bash
aria2c  -i  input_file
aria2c  http://download.cs.stanford.edu/orion/nocs/camera_train.zip
```
