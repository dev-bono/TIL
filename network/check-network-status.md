### ping 테스트

ip가 문제인지 체크할때 ping을 사용

```
$ ping 172.0.0.1
```

응답이 없으면 방화벽에 등록이 안된것


### telnet ip[domain] port

특정 IP의 특정 port가 열려있는지 확인하기 위해서는 telnet을 사용한다.
즉, ping은 되는데 telnet이 안된다면, 방화벽으로 port가 막혔기 때문일 것이다.

```
$ telnet 172.0.0.1 80

// ACL이 안열려있을때
Trying 172.0.0.1... 

// ACL이 열려있지만(방화벽 오픈), 프로세스가 안떠있을때
Trying 172.0.0.1... 
telnet: Unable to connect to remote host: Connection refused 

// 정상적인 상태
Trying 172.0.0.1...
Connected to 172.0.0.1
Escape character is '^]'.
```
