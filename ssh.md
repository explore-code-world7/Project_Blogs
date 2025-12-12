# 本地大文件上传到server
* 标准
```bash
rsync -avzP 大文件 user@server:/path
```
* case
```bash
rsync   -avP -e "ssh -p 10022"  bbf5e821355d420cea5f50a467b5e617bba791018d4d10784499b656c83e10c3  chenlei@10.15.82.118:/home/chenlei/Project/DeepSDF/data
```
