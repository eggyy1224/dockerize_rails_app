### 什麼是docker
[參考文章](https://philipzheng.gitbooks.io/docker_practice/content/introduction/what.html)

### mac如何安裝docker CLI
[按我](https://docs.docker.com/docker-for-mac/install/#install-and-run-docker-for-mac)
安裝後請在終端機輸入`$docker version` 確認安裝成功。

### dockerize rails 應用示範
假設app會用到別的服務(redis, postgres)的情況下。簡單說明如何在本地端 dockerize Rails 應用，撰寫 Dockerfile 以及 docker-compose.yml。
1. 在應用根目錄新增Dockerfile -> `$touch Dockerfile`  

``` 
# Base image: 選擇base image，你的應用的image將會以這個image為基底
FROM ruby:2.4.2

# Run commands: 在新的container中安裝必要的套件
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs
RUN mkdir /myapp

# Create working directory: 新增工作目錄
WORKDIR /myapp

# Copy files to working directory: 將應用程式的程式碼複製到工作目錄並且安裝rails gem
COPY Gemfile /myapp/Gemfile
COPY Gemfile.lock /myapp/Gemfile.lock
RUN bundle install
COPY . /myapp
```

2. 在應用根目錄新增docker-compose.yml -> `$touch docker-compose.yml`

``` yml
version: '3'
services: # 列出所有需要的服務
  db: # 服務的名稱
    image: postgres # 指定鏡像檔案
    volumes: # 暫存檔位置
    - ./tmp/db:/var/lib/postgresql/data
  redis:
    image: 'redis:4.0-alpine'
    command: redis-server --requirepass yourpassword # 運行後要下的指令
    volumes:
    - 'redis:/data'
  web:
    build: . # 依照根目錄的Dockerfile去build鏡像檔案
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    volumes:
    - .:/myapp
    ports: # 要開哪個port。格式為:本地端port：container中的port
    - "3000:3000"
    depends_on: # 這個鏡像依賴哪一些鏡像檔案
    - db
    - redis
  cable:
    depends_on:
    - 'redis'
    build: .
    command: puma -p 28080 cable/config.ru
    ports:
    - '28080:28080'
    volumes:
    - '.:/app'
volumes:
  redis:
  postgres:
```

3. 在終端機中輸入`$ docker-compose up`即可將應用在container中跑起來。

### Notes
在正在運行的container輸入指令：`$docker-compose exec <service_name> <commands>`。
用上面的例子來說，如果要遷移資料庫的話的話需要下這樣的指令：
`$docker-compose exec web rake db:migrate`

### 參考資料
1. https://docs.docker.com/compose/rails/#connect-the-database
2. https://nickjanetakis.com/blog/dockerize-a-rails-5-postgres-redis-sidekiq-action-cable-app-with-docker-compose