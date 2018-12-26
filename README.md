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
yum mariadb mariadb-server -y
```

2. 安裝完成後，啟動 MariaDB 服務，並且設定開機同時啟動：

```
systemctl start mariadb
systemctl enable mariadb
```

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
MariaDB [(none)]> create user 'mmAdmin'@'localhost' identified by 'P@ssw0rd';
MariaDB [(none)]> grant all on mattermost.* to 'mmAdmin'@'localhost' identified by 'P@ssw0rd' with grant option;
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

4. 創建儲存檔案的目錄

```
mkdir /opt/mattermost/data
```

> 這個目錄會包含所有上傳的檔案、圖片、使用者的貼文等紀錄。需準備足夠的硬碟空間去儲存這些資料。

5. 設定系統使用者與群組 mattermost，並將 Mattermost 目錄的存取權限賦予 mattermost

```
useradd --system --user-group mattermost
chown -R mattermost:mattermost /opt/mattermost
chmod -R g+w /opt/mattermost
```

6. 

## <a name=config>設定 Mattermost 伺服器</a>



## <a name=ref>引用文獻</a>


