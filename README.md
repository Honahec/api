# 我的 API 仓库

## 一言 API

一个随机返回精选句子的 API 服务

### 使用方法

1. 安装依赖

```bash
pip install -r requirements.txt
```

2. 创建`logs`目录

```bash
mkdir logs
```

3. 迁移数据库

```bash
python3 manage.py makemigrations
python3 manage.py migrate
```

4. 收集静态文件

```bash
python3 manage.py collectstatic
```

5. 创建超级用户

```bash
python3 manage.py createsuperuser
```

5. 运行

```bash
python3 manage.py runserver
```

### API 接口说明

- 获取随机句子：`GET /yiyan/get/`
- 管理界面：`/yiyan/admin/`
- RESTful API：
  - 获取所有句子：`GET /yiyan/sentences/`
  - 添加新句子：`POST /yiyan/sentences/`
  - 更新句子：`PUT /yiyan/sentences/{id}/`
  - 删除句子：`DELETE /yiyan/sentences/{id}/`

### nginx 配置

```nginx
server {
    listen 80;
    server_name api.honahec.cc;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name api.honahec.cc;

    ssl_certificate /etc/letsencrypt/live/honahec.cc/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/honahec.cc/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /static/ {
        alias /home/ecs-user/api/yiyan/staticfiles/;
    }
}
```

### systemd 配置

```bash
[Unit]
Description=Django development server for yiyan
After=network.target

[Service]
WorkingDirectory=/home/ecs-user/api/yiyan
Environment="PATH=/home/ecs-user/api/yiyan/venv/bin"
ExecStart=/home/ecs-user/api/yiyan/venv/bin/python manage.py runserver 0.0.0.0:8000
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```
