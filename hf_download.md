# hf download
```bash
hf download ShapeNet/ShapeNetCore --repo-type=dataset \
    --local-dir ./ --cache-dir= ./ --max-workers 1

hf download ShapeNet/ShapeNetCore --repo-type=dataset \
    --local-dir ./ --cache-dir= ./ --max-workers 1
```

🪫: env.python >=3.9 is required by hf cli

```python
远端
 ↓
cache-dir        ← 断点续传、校验
 ↓
local-dir        ← 你实际使用的数据
```


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
