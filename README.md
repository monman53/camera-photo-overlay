# camera-photo-overlay

## Setup

```
$ npm install -g http-server
$ openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem
```

## Develop

```
$ http-server -S -C cert.pem
```