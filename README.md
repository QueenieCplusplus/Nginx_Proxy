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


(1) 關於代理伺服器服務群的設定

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
      
 (2) 關於轉址的設定
 
       server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6;

          }
          
           location /article/{

             proxy_pass http://192.168.1.7; # 此為當使用者對目的www.katesapp.com/article 發起請求，因為 代理伺服器的 url 變數不會包含 uri, 故實際轉址為 http://192.168.1.7/article

          }

      }
      
 (3) 隱藏或忽略不處理標頭資訊
 
        server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6;
             proxy_hide_header [field 標頭域]; # 此指令可在 http、server、location 區塊中進行設定。
             proxy_pass_header [field 標頭域]; # 設定可以被發送的標頭資訊。
             proxy_pass_request_header on; # 設定用戶端將請求標頭發送給代理伺服器
             proxy_pass_request_body on; # 設定用戶端將請求本體發送給代理伺服器
             
             proxy_set_header field value; # 代理伺服器接收使用者端請求之標頭後經過修改，轉發給被代理伺服器。
             proxy_set_header $Host $proxy_host;
             proxy_set_header $Host $host:$proxy_port;
             proxy_set_body value; # 代理伺服器接收使用者端請求之請求本體，經過修改後，將新請求轉發給被代理伺服器。
             proxy_ignore_headers field...; # 代理伺服器去到被代理伺服器的回應資料後，不會處理被設定的標頭域。

          }
          
      }
 
 （4) bind 繫連結
 
 
      server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_bind addr; # 強制將代理主機連接與指定ip綁定。
            
          }
          
      }
      
  (5) 逾時設定
  
      server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_connect_timeout time; # 代理伺服器與被代理伺服器嘗試建立連接的逾時間，預設為 60s。
             proxy_read_timeout time; #代理伺服器向後端被代理伺服器服務群組發出讀取請求後，等待回應的逾時間，預設為 60s。
             proxy_send_timeout time; #代理伺服器向後端被代理伺服器發出寫入請求後，等待回應的逾時間，預設為 60s。
            
          }
          
      }
      
   (6) http 支援版本
   
      server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6;
             proxy_http_version 1.0 | 1.1;
            
          }
          
      }
      
   (7) 改寫用戶端請求的方法
   
     server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6;
             proxy_method POST; # or GET, 此指令一經設定，用戶端請求方法的域值將被忽略。
            
          }
          
      }
      
   (8) 戶端請求中斷時，設定代理伺服器也中斷與來自被代理伺服器的請求。
   
        server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6;
             proxy_ignore_client_abort on;
            
          }
          
      }
   
   
   (9)重新定義使用者的請求位址
   
      設定方式(1)與設定方式(2)相同
   
      server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6; # 此指令將用戶端請求位址重新定義為被代理伺服器的位置
             proxy_redirect [redirect 被取代值] [replacement 取代值]; # 藉此指令，可以將回應標頭和用戶端請求位址相對應，而非代理伺服器的傳回的位址資訊。
             
             # 設定方式(1)
             proxy_pass http://proxyserver/;
             proxy_redirect http://proxyserver/ /server/; # 前面的值為被取的的值 後者的值為取代值。
             
             # 設定方式(2)
             proxy_pass http://proxyserver/;
             proxy_redirect default ; # 前面的值為被取的的值 後者的值為取代值。
             
            
          }
          
      }
   
   (10) 回應 http 狀態碼
   
   
         server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6; # 此指令將用戶端請求位址重新定義為被代理伺服器的位置
             proxy_intercept_errors on; # 開啟此狀態，則被代理伺服器倘若回傳狀態數字大於等於 400，則代理伺服器使用自定義的錯誤頁面，如此設定被關閉，則被代理伺服器傳回的 http 狀態直接回應給使用者端。
            
          }
          
      }
      
   （11）請求標頭之容量限制
   
   
      server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6; # 此指令將用戶端請求位址重新定義為被代理伺服器的位置
             proxy_headers_hash_max_size [size]; 此指令設定儲存 http 封包標頭的雜湊表容量，預設上限為 512 字元。
             # 此 hash table 能儲存標頭中各種資訊，如伺服器名稱、多元網路媒體類型、請求標頭之名稱等等，固定單位大小由 proxy_headers_hash_bucket_size 設定，預設為 64 字元。
              
          }
          
      }
      
      
   (12) HTTPs
   
      server{

          server_name www.katesapp.com
          listen 80;
          location /{

             proxy_pass http://192.168.1.6; # 此指令將用戶端請求位址重新定義為被代理伺服器的位置
             proxy_ssl_session reuse on; # 此指令設定用以 SSL (第五層 secure socket layer) 安全協定為基礎的階段連接被代理的伺服器。 
            
          }
          
      }
      
      
  # Load Balancing, 負載平衡
  
   (to be continued...)
   
   網路負載平衡是利用一定的分配策略將網路平衡地分攤到網路叢集，使得單一工作能夠分攤到多個單元平行處理，抑或使得大量早方資料流量分擔到多單元個別處理，避免使用者等待回應時間過久。
   
   然而，實務上的 LB 會根據 OSI 七層模型進行劃分，現在的 LB 著重在 L4 | L7 ，完全獨立與硬體裝置，成為單獨的技術裝置。而 Nginx 是 L7 的 LB 實現。
   
   透過硬體實現的 LB 雖然效果更好，但是成本較高，軟體則是依賴程式的穩定性與演算法的選擇：分為靜態與動態的 LB 演算法。
   
   * 靜態負載平衡演算法
   
     透過加權輪詢實現，適合一般網路。
   
   * 動態負載平衡演算法
   
     包含最少連接優先演算法、最快回應優先演算法、動態效能分配演算法，適合複雜網路。
       

   
   
   
   
   
   
  
