# How to proxy web apps using nginx ?

### Step1. Nginx and two test nginx sever setting
- kubectl run nginx --image=nginx
- kubectl run nginx2 --image=nginx 
- kubectl run nginx3 --image=nginx
- kubectl expose pod nginx --target-port=80 --port=80
- kubectl expose pod nginx2 --target-port=80 --port=80 
- kubectl expose pod nginx3 --target-port=80 --port=80
- kubectl get ep
  - ```
    NAME         ENDPOINTS       AGE
    kubernetes   10.1.0.2:443    11d
    nginx        10.1.2.188:80   3h51m
    nginx2       10.1.2.189:80   22h
    nginx3       10.1.2.190:80   22h
    ```
### Step2. Edit Nginx Configuration
Let say we want to configure `nginx` to route requests for `/`, `/api`, `auth`, respectively onto `nginx.default.svc.cluster.local:80`, `nginx2.default.svc.cluster.local:80`, `nginx3.default.svc.cluster.local:80`.
```
                  +--- host --------> nginx on nginx.default.svc.cluster.local:80
                  |
users --> nginx --|--- host/api ---> nginx2 on nginx2.default.svc.cluster.local:80
                  |
                  +--- host/auth ---> nginx3 on nginx3.default.svc.cluster.local:80
```
Now, change the location section to this snippet:
```
server {
    listen       ...;
    ...
    location /api {
        proxy_pass http://nginx2.default.svc.cluster.local:80/;
    }
    location /auth {
        proxy_pass http://nginx3.default.svc.cluster.local:80/;
    }
    ...
}
```
`proxy_pass` simply tells `nginx` to forward requests to `/api` to the server listening on `http://nginx2.default.svc.cluster.local:80/`.

### Step 4. Reload nginx's Configuration
To reload `nginx`'s configuration run: `nginx -s reload` on your machine.
If you do not `reload`, the nginx configuration changes will not take effect on the nginx server.


### Hard Part
1. `nginx -s reload`
   1. `nginx configuration`을 바꾸고 요청을 보냈을 때 변경사항이 적용되지 않아 `404 error` 가 발생했었습니다.
   2. `curl command response`
      ```
      root@nginx:/etc/nginx/conf.d# curl localhost:80
      <!DOCTYPE html>
      <html>
      111111
      </html>
    
      root@nginx:/etc/nginx/conf.d# curl localhost:80/app
      <html>
      <head><title>404 Not Found</title></head>
      <body>
      <center><h1>404 Not Found</h1></center>
      <hr><center>nginx/1.23.3</center>
      </body>
      </html>
      ```
   3. `Nginx server log`
      ```
      127.0.0.1 - - [20/Jan/2023:06:44:00 +0000] "GET / HTTP/1.1" 200 38 "-" "curl/7.74.0" "-"
      127.0.0.1 - - [20/Jan/2023:06:44:02 +0000] "GET /ap HTTP/1.1" 404 153 "-" "curl/7.74.0" "-"
      ```
2. If you do not add a / after proxy_pass URL, the issue occurs.
   1. default.conf
      ```
      location /api {
        # This config would cause `/` issue
      }
    
      location /api/ {
        # This config works perfectly
      }
      ```
      
### Reference
- [Nginx Beginner's Guide](http://nginx.org/en/docs/beginners_guide.html)
- [Using NGINX reverse proxy with subpath gives "404: Not Found"](https://github.com/dani-garcia/vaultwarden/discussions/2289)