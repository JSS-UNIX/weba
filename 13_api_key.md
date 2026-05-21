# API-Key

## Validation using Nginx only

A very simple example to validate api keys directly via Nginx

```bash
cd /etc/nginx/conf.d/
sudo vi myKeys.conf
```

define valid keys:

```nginx
# validating the header "Authorization"
map $http_authorization $api_client_name {
    default       "";       # no key matches
    "key_aaaaaa"  "client_a";
    "key_bbbbbb"  "client_b";
}
```

and perform validation within Nginx site configuration:

```nginx
location /api/auth/names {
        # if empty, no key was found
        if ($api_client_name = "") {
            return 401 '{"error": "Ungültiger oder fehlender API-Key"}';
        }

        # Optional: forward user to backend (using header: X-Matched-Client)
        proxy_set_header X-Auth-User $api_client_name;

        proxy_pass http://127.0.0.1:5000/names;
}
```

Verify with Postman.

## Validation via Auth-Modul

integrate to webservice source code:

```python
from flask import Flask, jsonify, request, Response

API_KEYS = {
    "key_cccccc": "User-C",
    "key_dddddd": "User-D"
}

@app.route('/validate', methods=['GET'])
def validate():
    api_key = request.headers.get('Authorization')

    if not api_key:
        return Response("Missing API Key", status=401)

    client_name = API_KEYS.get(api_key)

    if client_name:
        res = Response("Valid", status=200)
        res.headers['X-Auth-User'] = client_name
        return res

    return Response("Invalid API Key", status=401)
```

and add to nginx site configuration:

```nginx
location = /_auth_check {
        internal; # nginx internal requests only
        proxy_pass http://localhost:5000/validate; # Auth-Modul

        # skip body
        proxy_pass_request_body off;
        proxy_set_header Content-Length "";

        # pass key to auth-service
        proxy_set_header X-Original-URI $request_uri;
        proxy_set_header Authorization $http_authorization;
}

location = /api/auth2/names {
        # perform request towards Auth-Service
        auth_request /_auth_check;

        # in case of 200 OK Nginx continues
        proxy_pass http://127.0.0.1:5000/names;
}
```

Verify with Postman.