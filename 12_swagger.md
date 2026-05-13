# Webservice - Documentation

## Swagger
include the metadata/description to the webapplication

```bash
vi /opt/webservice/webservice.py
```

```python
from flask import Flask
from flask_restx import Api, Resource, fields
from werkzeug.middleware.proxy_fix import ProxyFix

app = Flask(__name__)
app.wsgi_app = ProxyFix(app.wsgi_app, x_prefix=1)

api = Api(app, 
    version='1.0', 
    title='Addressbook API',
    description='Mega Fancy Addressbok'
)

name_model = api.model('NameInput', {
    'name': fields.String(required=True, description='Neuer Name')
})

names_list = ["Alice", "Bob", "Charlie"]

@api.route('/names')
class AddressList(Resource):
    
    @api.doc('get_names')
    def get(self):
        """Gibt die gesamte Liste zurück"""
        return names_list

    @api.expect(name_model) # Validierung & Dokumentation für den POST-Body
    @api.doc('add_name')
    @api.response(201, 'Erfolgreich hinzugefügt')
    @api.response(400, 'Ungültige Daten')
    def post(self):
        """Fügt einen neuen Namen hinzu"""
        data = api.payload
        if "name" in data:
            names_list.append(data["name"])
            return {"status": "success"}, 201
        api.abort(400, "Ungültige Daten")

    @api.doc('delete_all_names')
    def delete(self):
        """Leert die gesamte Liste"""
        names_list.clear()
        return {"status": "Liste geleert"}, 200

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=True)
```

## Reverse-Proxy

Make the webservice documentation available through nginx.

```bash
sudo vi /etc/nginx/sites-available/*site-name*
```

add to the server section:

```nginx
# API Queries are forwarded to Flask
location /api/doc {
    proxy_pass http://127.0.0.1:5000/;
    proxy_set_header X-Forwarded-Prefix /api/doc;
}
```

reload nginx, start the webservice:

```bash
sudo systemctl reload nginx
/opt/webservice/venv/bin/gunicorn -w 1 -b 127.0.0.1:5000 webservice:app
```

Verify the webservice Documentation via your browser.

URL: `https://*domain-name*/api/doc`
