# Nginx_Proxy
Proxy Server &amp; Reverse Proxy Server


* Forward Proxy

大部分的代理服務，是指『正向代理』服務，如果把 WAN 想像成一個極大資源庫，資源分布在網路中的各種網站上，區域網內 LAN 的用戶端倘若要使用這個資源庫資源就要透過代理服務，方才能對各種網站進行造訪。

而通常此處的代理也會造成一部份防火牆功能，也能對於網內道往外進行監控與管理，但方向僅僅是網內至網外。

目的是用戶端造訪網路中網站資源。

![forward](https://raw.githubusercontent.com/QueenieCplusplus/Nginx_Proxy/master/Forward_Proxy.png)

![fp](https://raw.githubusercontent.com/QueenieCplusplus/Nginx_Proxy/master/f_p.png)

基本部署的設定實例


      // Forward Proxy Server


      server{

          resolver 8.8.8.8; # 設定 DNS 伺服器的虛擬網路位址為 8.8.8.8，預設 DNS 服務的通訊阜號為 53，伺服器因此能處理收到的域名。
          listen 82; # 正向代理服務的監聽阜設定為 82
          location /{  # 伺服器收到的所有請求，都由此區塊過濾處理。
              proxy_pass http://$http_host$req_uri #被代理伺服器的位址
          }

      }


* Reverse Proxy

反向代理伺服器用來讓網外的用呼端連線區域網中的網站以造訪網站中資源。

目的是伺服器把資源發佈讓其它用戶端能夠造訪（通常反向代理伺服器和網站中間有防火牆）。

![reverse](https://raw.githubusercontent.com/QueenieCplusplus/Nginx_Proxy/master/Reversed_Proxy.png)

![rp](https://raw.githubusercontent.com/QueenieCplusplus/Nginx_Proxy/master/r_p.png)


      upstream{  # 倘若被代理伺服器是一組（群）伺服器，則使用 upstream 設定後端伺服器組。

       server http://192.168.1.1:8001/uri/; 
       server http://192.168.1.2:8001/uri/; # 同阜，不同 ip，使用 bind()
       server http://192.168.1.3:8000/uri/; # 不同阜，表示不同服務  

      }
      server{

          resolver 8.8.8.8;
          listen 82;
          location /{

              proxy_pass http://$http_host$req_uri; # 被代理伺服器的位址，可能是 主機名稱 與i p+port 和 uri 等要素
              proxy_pass http://www.queenie.com/tech;
              proxy_pass http://localhost:8000/blog_article;
              proxy_pass http://unix:/tmp/backend.socket:/uri/;
              proxy_pass service_group # 倘若被代理伺服器是一組（群）伺服器，則使用 upstream 設定後端伺服器組。

          }

      }

