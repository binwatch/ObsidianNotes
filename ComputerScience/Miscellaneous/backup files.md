
[阿里云盘](https://www.aliyundrive.com/drive)

获取 refresh token：开发者模式 -> Application -> Local Storage -> 域名 ->
 查看 token 的值 

[docker hub: aliyunpan-sync](https://hub.docker.com/r/tickstep/aliyunpan-sync)

```SHELL
docker run -d --name=aliyunpan-sync --restart=always -v "/home/xyu/readings:/home/app/data" -e TZ="Asia/Shanghai" -e ALIYUNPAN_REFRESH_TOKEN="" -e ALIYUNPAN_PAN_DIR="readings" -e ALIYUNPAN_SYNC_MODE="sync" -e ALIYUNPAN_TASK_STEP="sync" tickstep/aliyunpan-sync:v0.2.3
```