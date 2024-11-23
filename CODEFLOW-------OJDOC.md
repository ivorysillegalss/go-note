### CODEFLOW-------OJDOC

1. docker compose启动时报错

```
[root@iZf8ziqpw8ull82kxe9gbnZ OnlineJudgeDeploy]# docker-compose up -d
[+] Running 2/0
 ✔ Container onlinejudgedeploy-oj-redis-1     Running                                                                               0.0s
 ✔ Container onlinejudgedeploy-oj-judge-1     Running                                                                               0.0s
 ⠋ Container onlinejudgedeploy-oj-postgres-1  Starting                                                                              0.1s
Error response from daemon: driver failed programming external connectivity on endpoint onlinejudgedeploy-oj-postgres-1 (124350aea1e83690688da45a34aba8374d96d79e1821129d5480272497a8bacf):  (iptables failed: iptables --wait -t nat -A DOCKER -p tcp -d 0/0 --dport 5432 -j DNAT --to-destination 172.18.0.3:5432 ! -i br-0c189f12d847: iptables: No chain/target/match by that name.
 (exit status 1))
```

重启docker

```
[root@iZf8ziqpw8ull82kxe9gbnZ OnlineJudgeDeploy]# sudo service docker restart
Redirecting to /bin/systemctl restart docker.service
[root@iZf8ziqpw8ull82kxe9gbnZ OnlineJudgeDeploy]# docker-compose up -d
[+] Running 4/4
 ✔ Container onlinejudgedeploy-oj-judge-1     Running                                                                               0.0s
 ✔ Container onlinejudgedeploy-oj-redis-1     Running                                                                               0.0s
 ✔ Container onlinejudgedeploy-oj-postgres-1  Started                                                                               0.3s
 ✔ Container onlinejudgedeploy-oj-backend-1   Started                                                                               0.5s
```

