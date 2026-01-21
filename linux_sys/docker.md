# 1. 初始配置
* 拉取image
```bash
docker pull tyoung96/lvi_sam:1.0
```
* 创建image的container
```bash
docker run -it --rm tyoung96/lvi_sam:1.0 /bin/bash
```

# 2. 进入创建好的container
```
docker exec -it <container_id> /bin/bash
```
