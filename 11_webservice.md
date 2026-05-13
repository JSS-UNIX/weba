# Webservice

## Webservice itself
We use flask as application server (python based).
Check if flask is installed/otherwise install it...

```bash
sudo apt list python3 python3-venv --installed
```
```bash
sudo apt install python3 python3-venv -y
```

create a folder for the source-code & python dependencies:

```bash
sudo mkdir /opt/webservice
sudo chown tux:tux /opt/webservice
```

install python dependencies and webservice:

```bash
cd /opt/webservice
python3 -m venv venv
source venv/bin/activate 
pip install flask flask-restx gunicorn
deactivate

vi webservice.py
```

and paste following code:

```python
from flask import Flask
from flask_restx import Api, Resource

app = Flask(__name__)
api = Api(app)

names_list = ["Alice", "Bob", "Charlie"]

@api.route('/names')
class AddressList(Resource):
    
    def get(self):
        return names_list

    def post(self):
        data = api.payload
        if "name" in data:
            names_list.append(data["name"])
            return {"status": "success"}, 201
        api.abort(400)

    def delete(self):
        names_list.clear()
        return {"status": "Liste geleert"}, 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, debug=True)
```

test the webservice:

```bash
/opt/webservice/venv/bin/python3 webservice.py
```

Verify the webservice using Postman!
URL: `http://<server-ip-address>:5000/names`

## Reverse-Proxy

Stop the webservice simply by pressing `Ctrl + C`, or by `kill <pid>`.
change the binding of flask to localhost only by modifing the last line of the webservice source-code:

```python
if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=True)
```

**Note:** The webservice is no longer accessible from outside the host.

Make the webservice available through nginx.

```bash
sudo vi /etc/nginx/sites-available/*site-name*
```

add to the server section:

```nginx
# API Queries are forwarded to Flask
location /api/names {
    proxy_pass http://127.0.0.1:5000/names;
}
```

reload nginx, start the webservice:

```bash
sudo systemctl reload nginx
/opt/webservice/venv/bin/python3 webservice.py
```

Verify the webservice using Postman!

URL: `http://*domain-name*/api/names`

The previous URL should **NOT** work anylonger: `http://<server-ip-address>:5000/names`

To start the webservice "production ready", we have a use a wsgi server like gunicorn (which we installed initially via pip):

```bash
/opt/webservice/venv/bin/gunicorn -w 1 -b 127.0.0.1:5000 webservice:app
```

## API Documentation Example



| Method | Ressource (Path) | Input (Body) | Datatype (In) | Possible Values | Output (JSON) | Datatype (Out) | Description |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **GET** | `/names` | *n/a* | - | - | `["Alice", "Bob", ...]` | `Array<String>` | returns the complete list of names |
| **POST** | `/names` | `{"name": "..."}` | `String` | `*` | `{"status": "success"}` | `String` | adds a new name to the list |
| **DELETE** | `/names` | *n/a* | - | - | `{"status": "Liste geleert"}` | `String` | clears the complete list |


## Webapp -> Frontend

A simple frontend for the webservice. Create a html document (e.g. `webapp2.html`) with the content:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Demo: Addressbook</title>
</head>
<body>

    <h2>Einfaches Adressbuch</h2>
    
    <input type="text" id="nameAdd" name="name" placeholder="Name eingeben..." required>
    <button onclick="addName()">Speichern</button>
    
    <hr>

    <h3>Eintr&auml;ge:</h3>
		<div id="nameList"></div>
    
    <p><small><a id="deleteLink" href="#" onclick="deleteAll();return false;">Liste leeren</a></small></p>
    
	<script>
        
        const API_URL = '/api/names';

        async function loadNames() {
            try {
                const response = await fetch(API_URL);
                const names = await response.json();
                const list = document.getElementById('nameList');
                list.innerHTML = names.map(n => `Benutzer: ${n}<br>`).join('');
            } catch (e) {
                console.error("Backend nicht erreichbar", e);
            }
        }

        async function addName() {
            const val = document.getElementById('nameAdd').value;
            if(!val) return;

            await fetch(API_URL, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ "name": val })
            });
            document.getElementById('nameList').value = '';
            loadNames();
        }

        async function deleteAll() {
            await fetch(API_URL, { method: 'DELETE' });
            loadNames();
        }

        loadNames();
    </script>

</body>
</html>
```
