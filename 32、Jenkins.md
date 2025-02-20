# 31、Jenkins



## 安装

```bash
[root@ubuntu2404 ~]#apt update && apt install -y openjdk-17-jdk
[root@ubuntu2404 ~]#wget https://mirror.tuna.tsinghua.edu.cn/jenkins/debian-stable/jenkins_2.492.1_all.deb
[root@ubuntu2404 ~]#dpkg -i ./jenkins_2.492.1_all.deb
[root@ubuntu2404 ~]#systemctl status jenkins.service 
● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/usr/lib/systemd/system/jenkins.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-02-19 14:39:08 CST; 37s ago
   Main PID: 16864 (java)
      Tasks: 50 (limit: 4557)
     Memory: 552.2M (peak: 554.1M)
        CPU: 13.779s
     CGroup: /system.slice/jenkins.service
             └─16864 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080

Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: 8ad98603196d4ad7a7fc47bc85824f41
Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: This may also be found at: /var/lib/jenkins/secrets/initialAdminPassword
Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: *************************************************************
Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: *************************************************************
Feb 19 14:38:57 ubuntu2404.wang.org jenkins[16864]: *************************************************************
Feb 19 14:39:08 ubuntu2404.wang.org jenkins[16864]: 2025-02-19 06:39:08.045+0000 [id=33]        INFO        jenkins.InitReactorRunner$1#onAttained: Completed initial>Feb 19 14:39:08 ubuntu2404.wang.org jenkins[16864]: 2025-02-19 06:39:08.074+0000 [id=23]        INFO        hudson.lifecycle.Lifecycle#onReady: Jenkins is fully up a>Feb 19 14:39:08 ubuntu2404.wang.org systemd[1]: Started jenkins.service - Jenkins Continuous Integration Server.
Feb 19 14:39:13 ubuntu2404.wang.org jenkins[16864]: 2025-02-19 06:39:13.422+0000 [id=48]        INFO        h.m.DownloadService$Downloadable#load: Obtained the updat>Feb 19 14:39:13 ubuntu2404.wang.org jenkins[16864]: 2025-02-19 06:39:13.422+0000 [id=48]        INFO        hudson.util.Retrier#start: Performed the action check upd>lines 1-20/20 (END)

#密码存放路径
[root@ubuntu2404 ~]#cat /var/lib/jenkins/secrets/initialAdminPassword
8ad98603196d4ad7a7fc47bc85824f41

```

