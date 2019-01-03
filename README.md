## Mattermost for CentOS 7

* [更新作業系統](#linux)
* [安裝 MariaDB 伺服器](#mariadb)
* [安裝 Mattermost 伺服器](#mattermost)
* [設定 Mattermost 伺服器](#config)
* [引用文獻](#ref)

## <a name=linux>更新作業系統</a>

1. 安裝 CentOS 7 之後，執行下列指令：

```
yum update -y
yum upgrade -y
```

## <a name=mariadb>安裝 MariaDB 伺服器</a>

1. 安裝 MariaDB：

```
yum install mariadb mariadb-server -y
```

2. 安裝完成後，啟動 MariaDB 服務，並且設定開機同時啟動：

```
systemctl start mariadb
systemctl enable mariadb
```

> 安裝 MySQL 5.7+ 或 MariaDB 10.+
>
> mysql 5.6 以前的伺服器版本，啟動 Mattermost 時會出現 error code:
>
> "Failed to create index Error 1214: The used table type doesn't support FULLTEXT indexes"

3. 設定 root 密碼：

```
mysql_secure_installation
```

4. 回答 terminal 上，逐條出現的問題：

```
Enter current password for root (enter for none): Enter
Set root password? [Y/n]: Y
Enter root password: ********
Re-enter root password: ********
Remove anonymous users? [Y/n]: Y
Disallow root login remotely? [Y/n]: Y
Remove test database and access to it? [Y/n]: Y
Reload privilege tables now? [Y/n]: Y
```

5. 登入 MariaDB，隨後輸入上一步驟所設定的密碼：

```
mysql -u root -p
```

6. 登入 MariaDB 後，創建一個 Mattermost 使用的資料庫 mattermost：

```
MariaDB [(none)]> create database mattermost;
```

7. 創建 Mattermost 登入資料庫的使用者帳號，並設定權限：

```
MariaDB [(none)]> create user '<username>'@'localhost' identified by '<password>';
MariaDB [(none)]> grant all on mattermost.* to '<username>'@'localhost' identified by '<password>' with grant option;
MariaDB [(none)]> \q
```

## <a name=mattermost>安裝 Mattermost 伺服器</a>

1. 下載<a href="https://mattermost.com/download/">最新版 Mattermost</a>

```
wget https://releases.mattermost.com/5.6.2/mattermost-5.6.2-linux-amd64.tar.gz
```

2. 解壓縮 Mattermost Server 檔案：

```
tar -xvzf *.gz
```

3. 移動解壓縮後的 Mattermost Server 檔案到 /opt 目錄：

```
mv mattermost /opt
```

<a name="file">4. 創建儲存檔案的目錄：</a>

```
mkdir /opt/mattermost/data
```

> 這個目錄會包含所有上傳的檔案、圖片、使用者的貼文等紀錄。需準備足夠的硬碟空間去儲存這些資料。

5. 設定系統使用者與群組 mattermost，並將 Mattermost 目錄的存取權限賦予 mattermost：

```
useradd --system --user-group mattermost
chown -R mattermost:mattermost /opt/mattermost
chmod -R g+w /opt/mattermost
```

6. 在設定檔 /opt/mattermost/config/config.json 加入資料庫設定： 

```
...
121 "SqlSettings": {
122     "DriverName": "mysql",
123     "DataSource": "<username>:<password>@tcp(<ip-address>:3306)/mattermost?charset=utf8mb4,utf8&readTimeout=30s&writeTimeout=30s",
...
```

7. 測試 Mattermost 伺服器運行：

```
sudo -u mattermost /opt/mattermost/bin/mattermost
```

> 伺服器啟動後，出現 ```Server is listening on :8065```，按 ctrl + c 退出。

8. 把 Mattermost 服務程序加入系統管理 (systemd init daemon)：

建立 Mattermost 執行設定檔：

```
touch /etc/systemd/system/mattermost.service
```

編輯執行設定檔，將下列設定指令加入檔案中：

```
[Unit]
Description=Mattermost
After=syslog.target network.target mysqld.service

[Service]
Type=notify
WorkingDirectory=/opt/mattermost
User=mattermost
ExecStart=/opt/mattermost/bin/mattermost
PIDFile=/var/spool/mattermost/pid/master.pid
TimeoutStartSec=3600
LimitNOFILE=49152

[Install]
WantedBy=multi-user.target
```

將執行設定檔，轉換成可執行模式：

```
chmod 664 /etc/systemd/system/mattermost.service
```

重新載入系統管理：

```
systemctl daemon-reload
```

設定 Mattermost 伺服器開機時啟動：

```
systemctl enable mattermost
```

9. 啟動 Mattermost 伺服器：

```
systemctl start mattermost
```

## <a name=config>設定 Mattermost 伺服器</a>

建立系統管理者帳號，並設定 Mattermost 伺服器。

1. 打開瀏覽器，連線到 Mattermost 的執行個體 (伺服器)。例如，執行個體的 IP 位置為 ```10.0.22.73```，Mattermost 的伺服器為 <http://10.0.22.73:8065>。

2. 創建第一個團隊與使用者。第一位在伺服器上註冊的使用者，具有系統管理者 (```system_admin```) 的權限，可以操作系統控制台 (System Console)

3. 點擊使用者名稱上方的控制選項，點選 **System Console**，打開系統控制台。

4. 設定 Mattermost 網址
> a. 在 **GENERAL** 的段落, 點擊 **Configuration**。
>
> b. 在 **Site URL** 輸入伺服器的 URL。

5. 設定 email 通知
> a. 在 **NOTIFICATIONS** 的段落，點擊, **Email** 輸入下列必填欄位：
>
> * 將允許 email 通知 (Enable Email Notifications) 設為 true
> * 設定寄送通知的郵件信箱 (Notification From Address)，例如：berchem.lin@gomail.com
> * 設定寄件伺服器 (SMTP Server)，例如：smtp.gomail.com
> * 設定寄件伺服器埠 (SMTP Server Port)，例如：587
> * 將允許 SMTP 驗證 (Enable SMTP Authentication) 設為 true
> * 設定寄件帳號 (SMTP Server Username)，例如：berchem.lin@gomail.com
> * 設定寄件帳號的密碼 (SMTP Server Password)
> * 依照寄件伺服器所允許的安全性連線，設定 TLS 或 STARTTLS。
>
> b. 點擊 **Test Connection**。
>
> c. 測試成功後，點擊 **Save**。

6. 設定檔案與圖片的儲存位置。

> a. 在 **FILES** 的段落，點擊 **Storage**。
>
> b. 如果你將檔案存在 local 端，在 **File Storage System** 的選單選擇 **Local File System**，在 **Local Storage Directory** 的設置中，可使用預設目錄或自行輸入目錄。Mattermost 伺服器必須對這個目錄具有讀寫的權限[[recall: 安裝 Mattermost 伺服器 step 4]](#file)。此外，這個目錄可以是絕對路徑或相對路徑，相對路徑的參考目錄為 ```/opt/mattermost/```。
>
> c. 點擊 **Save**。

7. 重新啟動 Mattermost 伺服器：

```
sudo systemctl restart mattermost
```

## <a name=ref>引用文獻</a>

* <a href="https://docs.mattermost.com/install/install-rhel-71.html#configuring-mattermost-server">Installing Mattermost on RHEL 7.1</a>
