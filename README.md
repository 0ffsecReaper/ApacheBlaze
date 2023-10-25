# ApacheBlaze
![Uploaded Image](https://github.com/0ffsecReaper/ApacheBlaze/blob/README.md/Screenshot%20from%202023-10-25%2018-36-08.png)

### After Deploying the Machine you will Lend on this page Which look like this

![](https://github.com/0ffsecReaper/ApacheBlaze/blob/README.md/HackTheBox-ApacheBlaze-Website.png)

** After Downloadeing the source code please read through the whole code and analyse what you have ssen to something location to which you havemt visited 
```
from flask import Flask, request, jsonify

app = Flask(__name__)

app.config['GAMES'] = {'magic_click', 'click_mania', 'hyper_clicker', 'click_topia'}
app.config['FLAG'] = 'HTB{f4k3_fl4g_f0r_t3st1ng}'

@app.route('/', methods=['GET'])
def index():
    game = request.args.get('game')

    if not game:
        return jsonify({
            'error': 'Empty game name is not supported!.'
        }), 400

    elif game not in app.config['GAMES']:
        return jsonify({
            'error': 'Invalid game name!'
        }), 400

    elif game == 'click_topia':
        if request.headers.get('X-Forwarded-Host') == 'dev.apacheblaze.local':
            return jsonify({
                'message': f'{app.config["FLAG"]}'
            }), 200
        else:
            return jsonify({
                'message': 'This game is currently available only from dev.apacheblaze.local.'
            }), 200

    else:
        return jsonify({
            'message': 'This game is currently unavailable due to internal maintenance.'
        }), 200
```
* We can easily can see that easily we can add the path and can get the flag via seeing this code snip
  ```
  elif game == 'click_topia':
        if request.headers.get('X-Forwarded-Host') == 'dev.apacheblaze.local':
            return jsonify({
                'message': f'{app.config["FLAG"]}'
            }), 200
  ```
  ![](https://github.com/0ffsecReaper/ApacheBlaze/blob/README.md/HackTheBox-ApacheBlaze-burp-with-Forwarded-host.png)

  ### We can easily see but that not that much easy we have to see something or explore more and after more analysis we see:
  * In the Dockerfile We know that the game source code is located in the /app/ folder
  * And it uses uwsgi to start the server with two ports 8081 and 8082.
  * File httpd.conf:
```
ServerName _
ServerTokens Prod
ServerSignature Off

Listen 8080
Listen 1337

ErrorLog "/usr/local/apache2/logs/error.log"
CustomLog "/usr/local/apache2/logs/access.log" common

LoadModule rewrite_module modules/mod_rewrite.so
LoadModule proxy_module modules/mod_proxy.so
LoadModule proxy_http_module modules/mod_proxy_http.so
LoadModule proxy_balancer_module modules/mod_proxy_balancer.so
LoadModule slotmem_shm_module modules/mod_slotmem_shm.so
LoadModule lbmethod_byrequests_module modules/mod_lbmethod_byrequests.so

<VirtualHost *:1337>

    ServerName _

    DocumentRoot /usr/local/apache2/htdocs

    RewriteEngine on

    RewriteRule "^/api/games/(.*)" "http://127.0.0.1:8080/?game=$1" [P]
    ProxyPassReverse "/" "http://127.0.0.1:8080:/api/games/"

</VirtualHost>

<VirtualHost *:8080>

    ServerName _

    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/

    <Proxy balancer://mycluster>
        BalancerMember http://127.0.0.1:8081 route=127.0.0.1
        BalancerMember http://127.0.0.1:8082 route=127.0.0.1
        ProxySet stickysession=ROUTEID
        ProxySet lbmethod=byrequests
    </Proxy>

</VirtualHost>
```
In Above code we can see that "^/api/games/(.*) we forwarded through port 8080 with the ip 127.0.0.1, then it Continue to go through the balancer module with ip 127.0.0.1 and will arrive at one of two ports, 8081 or 8082.

Some after some more analysis Then I know that  the X-Forwarded-Host so we will pass in will carry 2 additional IPs as I mentioned above.
hat uwsgi uses version 2.0.22 and apache has version httpd-2.4.55. Looking for some vulnerabilities I found:
Apache HTTP Server via mod_proxy_uwsgi HTTP response smuggling
With this vulnerability We will Make and Payload 
```
GET /api/games/click_topia HTTP/1.1
Host: dev.apacheblaze.local

GET /<ANY> HTTP/1.1
Host: <IP>:<PORT>
...
```
Just above url encode it some how that the spaces and new lines will be encoded atleast and under that the asusual request will be sended like this :

![]()
and you will get the flag



Hence Pwned

