# Docker を使った環境構築の簡易版
[Docker を使った環境構築の簡易版](https://github.com/akama-hub/setup_redash/Original_setup)


# Local developement setup
[公式Doc](https://github.com/getredash/redash/wiki/Local-development-setup)

## Requirements
python
WSL2

### Upgrade 
[公式Doc](https://redash.io/help/open-source/admin-guide/how-to-upgrade)

---

<!-- ## Auto Setup
Redash の Docker image を使った環境構築手順を示す -->


## 1. Manual Set up the prerequisites
Redash のローカル環境の作成手順(Manual)を以下に記す

### Install needed packages

docker-compose の version 2 が入っていない場合は下記をインストール

```
$ sudo apt -y install docker.io docker-buildx docker-compose-v2
$ sudo apt -y install build-essential curl docker-compose pwgen python3-venv xvfb
```

### Add your user to the "docker" group

```
$ sudo usermod -aG docker $USER
```

### Install Node Version Manager

Node 管理ツール(nvm)が入っていない場合は下記をインストール

```
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```

Node が入っていない場合は下記をインストール
nvm の version は 14.16 <= 17.0.0

```
$ nvm install --lts 16
$ nvm alias default 16
$ nvm use 16
```

### Install Yarn 1.x

```
$ npm install --global yarn@1.22.19
```

### Clone the Redash source code and install the NodeJS dependencies

```
$ git clone https://github.com/getredash/redash
$ cd redash
$ yarn
```

### Generate your local environment variables file

```.env```ファイルを作成する.
+ Cookie Secret: The cookie secret is taken from the REDASH_COOKIE_SECRET environment variable. It is used for various cryptographic features of the web server, such as authenticating users, signing cookies, and storing user session information. 

```python -c 'import secrets; print(secrets.token_hex())'```

上記で作成した秘密鍵を```.env```ファイル内に記す


+ Application Secret: The application secret is taken from the REDASH_SECRET_KEY environment variable. It is used to encrypt all settings on the Settings > Data Sources screen and Settings > Alert Destinations screen.

If you attempt to start Redash without it, the webserver will not initialise and you will find the following message in your logs:

```
$ make env
```

## 2. Compile and build
Redash uses GNU Make to run things, so if you're not sure about something it's often a good idea to take a look over the Makefile which can help. 😄

### Build the Redash front end

```
$ make build
```

### Build local Redash Docker image

```
$ make compose_build
```

確認
```$ docker image list
REPOSITORY         TAG       IMAGE ID       CREATED         SIZE
redash_scheduler   latest    85bc2dc57801   2 minutes ago   1.38GB
redash_server      latest    85bc2dc57801   2 minutes ago   1.38GB
redash_worker      latest    85bc2dc57801   2 minutes ago   1.38GB```
```

### Start Redash locally, using the docker images you just built

```
$ make create_database
$ make up

$ docker compose ps
       Name                     Command                  State                                  Ports                            
---------------------------------------------------------------------------------------------------------------------------------
redash_email_1       bin/maildev                      Up (healthy)   1025/tcp, 1080/tcp, 0.0.0.0:1080->80/tcp,:::1080->80/tcp    
redash_postgres_1    docker-entrypoint.sh postg ...   Up             0.0.0.0:15432->5432/tcp,:::15432->5432/tcp                  
redash_redis_1       docker-entrypoint.sh redis ...   Up             6379/tcp                                                    
redash_scheduler_1   /app/bin/docker-entrypoint ...   Up             5000/tcp                                                    
redash_server_1      /app/bin/docker-entrypoint ...   Up             0.0.0.0:5001->5000/tcp,:::5001->5000/tcp,                   
                                                                     0.0.0.0:5678->5678/tcp,:::5678->5678/tcp                    
redash_worker_1      /app/bin/docker-entrypoint ...   Up             5000/tcp
```



Once you've finished confirming everything works the way you want, then shut down the containers with:
```
$ make down
```

## 3. Set up Python for local backend development
```
$ sudo apt install -y --no-install-recommends default-libmysqlclient-dev freetds-dev libffi-dev libpq-dev \
    python3-dev libsasl2-dev libsasl2-modules-gssapi-mit libssl-dev unixodbc-dev xmlsec1
```

### 仮想環境にはいる
+ venv を使用する場合
```
$ python3 -m venv ~/redashvenv1
$ source ~/redashvenv1/bin/activate
```
+ pipenv を使用する場合
```
$ pipenv shell
```

### With that done, install the rest of the Python dependencies
```
(redash) $ pip3 install wheel  # "wheel" needs to be installed by itself first
(redash) $ pip3 install --upgrade ruff launchpadlib pip setuptools
(redash) $ pip3 install poetry
(redash) $ poetry install --only main,all_ds,dev
```

```
sudo apt install pre-commit -y

(redashvenv1) $ make format
pre-commit run --all-files
isort....................................................................Passed
black....................................................................Passed
flake8...................................................................Passed
```