# Nexus 설치 및 설정





## 설치

### 1. Nexus 설치
```
-- Creating necessory folder structure
# mkdir -p /data/nexus-data /opt/nexus

-- Download latest Nexus artifact
# mkdir ~/install/nexus && cd ~/install/nexus
# wget -O nexus.tar.gz http://download.sonatype.com/nexus/3/latest-unix.tar.gz

-- Extract it to /opt/nexus
# tar xvfz nexus.tar.gz -C /opt/nexus --strip-components 1

-- Adding a service account for nexus
# useradd --system --no-create-home nexus

-- Provide necessory folder permissions
# chown -R nexus:nexus /opt/nexus
# chown -R nexus:nexus /data/nexus-data
```
https://notebook.yasithab.com/centos/install-nexus-repository-oss-on-centos-7/

## 환경설정

### 2-1. Nexus 환경설정 (NEXUS HOME 설정)
```
# vim /etc/profile
```
```
    -- Setting up JAVA_HOME (설정되어 있다면 Pass)
    # export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which javac)))))

    -- Setting up NEXUS_HOME
    # export NEXUS_HOME=/opt/nexus

    PATH=$PATH:$NEXUS_HOME/bin
    export PATH
```
```
# source /etc/profile
# echo $ NEXUS_HOME
```

## 환경설정

### 2-2. Nexus 환경설정 (기본 설정 변경)
```
-- Default 항목을 아래와 같이 변경
# vi $NEXUS_HOME/bin/nexus.vmoptions
```
```
    -Xms1200M
    -Xmx1200M
    -XX:MaxDirectMemorySize=4G
    -XX:+UnlockDiagnosticVMOptions
    -XX:+UnsyncloadClass
    -XX:+LogVMOutput
    -XX:LogFile=/data/nexus-data/nexus3/log/jvm.log
    -Djava.net.preferIPv4Stack=true
    -Dkaraf.home=.
    -Dkaraf.base=.
    -Dkaraf.etc=etc/karaf
    -Djava.util.logging.config.file=etc/karaf/java.util.logging.properties
    -Dkaraf.data=/data/nexus-data/nexus3
    -Djava.io.tmpdir=/data/nexus-data/nexus3/tmp
    -Dkaraf.startLocalConsole=false
```

### 2-3. Nexus 환경설정 (nexus user set)
```
# vi $NEXUS_HOME/bin/nexus.rc
```
```
    run_as_user="nexus"
```

## 환경설정

### 2-4. Nexus 환경설정 (nexus service 생성)
```
# vi /etc/systemd/system/nexus.service
```
```
    [Unit]
    Description=Nexus Server
    After=syslog.target network.target
    [Service]
    Type=forking
    LimitNOFILE=65536
    ExecStart=/opt/nexus/bin/nexus start
    ExecStop=/opt/nexus/bin/nexus stop
    User=nexus
    Group=nexus
    Restart=on-failure
    [Install]
    WantedBy=multi-user.target
```

### 2-5. Nexus 환경설정 (open file limit increase)
```
# vi /etc/security/limits.conf
```
```
    nexus    -    nofile    65536
```

### 2-5. Nexus 환경설정 (port 변경)
```
# vi /opt/nexus/etc/nexus-default.properties
```
```
    application-port=40080
```

## 실행

### 1. 실행
```
# systemctl daemon-reload
# systemctl start nexus.service
# systemctl enable nexus.service
```

### 2. 확인
```
# netstat -tulpn | grep 40080
```
### 3. log
```
# tail –f /data/nexus-data/nexus3/log/nexus.log
```

### 4. 초기password
```
/data/nexus-data/nexus3/admin.password
```

