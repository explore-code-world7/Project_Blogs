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

## decompress
* zip

```bash
# 1. 安装（如果尚未安装）
sudo apt install p7zip-full

# 2. 使用多线程解压（-mmt参数启用多线程，x为解压）
7z x ShapeNetCore.v1.zip -mmt=on

# 3. 如果想静默解压且不覆盖已有文件，可加-o和-y参数
7z x ShapeNetCore.v1.zip -mmt=on -o./ -y
```


* zip
```bash
# 1. 安装
sudo apt install libarchive-tools

# 2. 解压（它会自动尝试最优方式）
bsdtar -xvf ShapeNetCore.v1.zip
```

* zip内的tar
```bash
# 1. 安装pigz
sudo apt install pigz

# 2. 流式解压（如果zip内是tar.gz/tar）
unzip -p ShapeNetCore.v1.zip | pigz -dc | tar xf -
```


* tar


# windows
```bash
curl -o ".\garden.bag"  "https://drive.usercontent.google.com/download?id=1GgkBF0Nql8yMR-edQ7rWhU1MNXM6iKwc&export=download&confirm=t&uuid=bf3761f8-5cd7-46ba-a9eb-3516b5c09def"
Invoke-WebRequest -Uri "https://example.com/file.zip" -OutFile "C:\Path\To\Save\file.zip"
```
