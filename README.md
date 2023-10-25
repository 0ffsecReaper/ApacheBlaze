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
