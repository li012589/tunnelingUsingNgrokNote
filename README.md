# Using  ngrok to tunneling 

## Install go complier

```bash
sudo apt install golang
```

double check by run `go env`, you should see:

```bash
GOARCH="amd64"
GOBIN=""
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="linux"
GOOS="linux"
GOPATH=""
GORACE=""
GOROOT="/usr/lib/go-1.7"
GOTOOLDIR="/usr/lib/go-1.7/pkg/tool/linux_amd64"
CC="gcc"
GOGCCFLAGS="-fPIC -m64 -pthread -fmessage-length=0 -fdebug-prefix-map=/tmp/go-build571672580=/tmp/go-build -gno-record-gcc-switches"
CXX="g++"
CGO_ENABLED="1"
```

## Git clone repos

```bash
git clone https://github.com/mamboer/ngrok.git
```

Then, configure your domain name by adding `NGROK_DOMAIN=ngrok.yourdomain.com` to your _bashrc_, and add **A record of "ngrok" to your Domain Server, and CNAME record of "*.ngrok"**.

## Generate SSL certificate

```bash
cd ~/ngrok
openssl genrsa -out base.key 2048
openssl req -new -x509 -nodes -key base.key -days 10000 -subj "/CN=$NGROK_DOMAIN" -out rootCA.pem
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
openssl x509 -req -in server.csr -CA rootCA.pem -CAkey base.key -CAcreateserial -days 10000 -out server.crt

```

Then move them

```bash
cp rootCA.pem assets/client/tls/ngrokroot.crt
cp server.crt assets/server/tls/snakeoil.crt 
cp server.key assets/server/tls/snakeoil.key
```

## Compling 

```bash
cd ~/ngrok
export GOOS=linux # or windows or darwin
export GOARCH=amd64 # or arm for pi, or 386
make release-server
make release-client
```

You will find your server at `ngrok/bin/linux_amd64` (or corresponding folder according to your architecture) named `ngrokd`, and your client named `ngrok`.

## Run server

Run:

```bash
nohup path-to/ngrokd -domain="ngrok.v2nobel.com" -httpAddr=":6060" -httpsAddr=":6061" -tunnelAddr=":6062" >ngrokd.log &
```

## Run client

First configure shadowsocks at your server as mentioned at my other repos. Here are some brief config files

```bash
# /etc/shadowsocks-libev/config.json
{
    "server":"0.0.0.0",
    "server_port":8888,
    "password":"passwd",
    "timeout":600,
    "method":"aes-256-cfb"
}
```

Then write the following config files for `ngrok` , and save it as `ngrok.yml`.

```bash
server_addr: "ngrok.v2nobel.com:6062"
trust_host_root_certs: false
tunnels:
	ss:
  		remote_port: 8888
      	proto:
        	tcp: 8888
    ssh:
    	remote_port: 222
    	proto:
      		tcp: 22
```

Then 

```bash
path-to/ngrok -log=ngrok.log -config=ngrok.yml start ss
```

Note that start from ngrok2, you may need _screen_ or _tmux_ to run it in background.