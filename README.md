# HAProxy setup for Xilriws
--------------------------------------------------
This guide should help you to setup haproxy as a load balancer for multiple proxy endpoints to use with Xilriws.

HAProxy SHOULD be installed on the same machine as Xilriws.

--------------------------------------------------
**HAProxy Docker stack setup**

 1. `git clone https://github.com/andi2022/HAProxy-Xilriws.git`
 2. `cd HAProxy-Xilriws`
 3.  `cp bancheck_ptc.sh.example bancheck_ptc.sh`
 4. `chmod +x bancheck_ptc.sh`
 5. `cp docker-compose.yml.example docker-compose.yml`
 6. `cp haproxy.cfg.example haproxy.cfg`
 7. `cp squid.conf.example squid.conf`
 8. `chmod +x curl`

Add your porxies to the haproxy.cfg in the backend_ptc section. See the haproxy.cfg if your proxies use authentication.
```
  server proxy1 1.2.3.4:3128 check inter 20s fall 3 rise 2
  server proxy2 5.6.7.8:3128 check inter 20s fall 3 rise 2
```
If you install your haproxy on a dedicated server or vps you should secure the stats site.
default username is `stats` and the password is `haproxy`.

In the following section in the haproxy.cfg you can change the username and password.
Use this [documentaion](https://www.haproxy.com/documentation/haproxy-configuration-tutorials/authentication/basic-authentication/).
```
userlist mycredentials
  user stats password $5$eD.zS8VZq0MaisXv$MAnVQCdWObOxpPEX2Jozyq5Fzc.3nOE/Tzm0SSmLgH/
```

If your porxies are IP whitelisted and not use authentication you need to adjust the bancheck_ptc script. bancheck.sh
comment out the first bancheck cmd and delete the # on the second bancheck command.
Like this example:
```
#Bancheck for proxy with authentication.
#cmd=`curl -I -s -U proxyuser:proxypasswort -x ${RIP}:${RPT} https://access.pokemon.com/login 2>/dev/null | grep -e "HTTP/1.1 200"`

#Bancheck for proxy without authentication. IP whitelisted.
cmd=`curl -I -s -x ${RIP}:${RPT} https://access.pokemon.com/login 2>/dev/null | grep -e "HTTP/1.1 200"`
```
Now you can start the docker stack.
`docker compose up -d`

You can check now your Haproxy Stats site in your browser.
http://serverip:9101/stats
For better security you should expose this with a reverse proxy like nginx and make it available over https. (Don't forget to delete the port forwarding to the docker container if you do this!)

**HAProxy stats example**
![HAProxy stats example](https://raw.githubusercontent.com/andi2022/HAProxy-Xilriws/main/images/haproxy-stats-example.png)

--------------------------------------------------
**Configure HAProxy as proxy in the Xilriws docker compose**
```
environment:
  - https_proxy=http://haproxy:9100
```
It's important that Xilriws an the HAProxy run in the same docker network.
You can put Xilriws and the HAProxy stack in the same docker-compose.yaml or add the network to the HAProxy stack.

Add this at the end of the HAProxy docker-compose.yml to add it to the scanner network from the UnownStack. If Xilriws is also on the scanner network.
```
networks:
  default:
    name: scanner
    external: true #needs to be created by other file
```

--------------------------------------------------
**Final notes**
The curl executable is needed because the official HAProxy docker image don't have curl included.

Xilriws is a tool developed by the [UnownHash](https://github.com/UnownHash) team.
[Github repository](https://github.com/UnownHash/Xilriws-Public)
