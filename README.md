# Disclaimer
**This is an alpha build and is currently under active development.** Please be advised of the following:

- This code currently is not audited by an external security auditor, use it at your own risk
- The code **has not been subjected to thorough review** by engineers at the Electric Coin Company, Pirate Chain, or otherwise
- We **are actively changing** the codebase and adding features where/when needed

🔒 Security Warnings

The Lightwalletd Server is experimental and a work in progress. Use it at your own risk. Developers should familiarize themselves with the [wallet app threat model](https://zcash.readthedocs.io/en/latest/rtd_pages/wallet_threat_model.html), since it contains important information about the security and privacy limitations of light wallets that use Lightwalletd.

---

# Overview

ARRRwallet lightwalletd is a fork of [lightwalletd](https://github.com/adityapk00/lightwalletd) from the ECC. 

It is a backend service that provides a bandwidth-efficient interface to the PIRATE blockchain for the ARRRwallet light wallet.

## Changes from upstream lightwalletd
This version of ARRRwallet lightwalletd extends lightwalletd and:
* Adds support for transparent addresses
* Adds several new RPC calls for lightclients
* Lots of perf improvements
  * Replaces SQLite with in-memory cache for Compact Blocks
  * Replace local Txstore, delegating Tx lookups to Pirated
  * Remove the need for a separate ingestor

## Running your own arrrlite lightwalletd

#### 0. First, install [Go >= 1.11](https://golang.org/dl/#stable).

#### 1. Run a PIRATE node.
Start a `pirated` with the following options:
```
server=1
daemon=1
txindex=1
addressindex=1
rpcuser=user
rpcpassword=password
rpcport=45453
rpcworkqueue=256
experimentalfeatures=1
insightexplorer=1
rpcallowip=127.0.0.1
rpcbind=127.0.0.1
```

You might need to run with `-reindex` the first time if you are enabling the `txindex` or `insightexplorer` options for the first time. The reindex might take a while. If you are using it on testnet, please also include `testnet=1`

#### 2. Get a TLS certificate

##### a. "Let's Encrypt" certificate using NGINX as a reverse proxy
If you running a public-facing server, the easiest way to obtain a certificate is to use a NGINX reverse proxy and get a Let's Encrypt certificate. [Instructions are here](https://www.nginx.com/blog/using-free-ssltls-certificates-from-lets-encrypt-with-nginx/)

Create a new section for the NGINX reverse proxy:
```
server {
    listen 443 ssl http2;
 
 
    ssl_certificate     ssl/cert.pem; # From certbot
    ssl_certificate_key ssl/key.pem;  # From certbot
    
    location / {
        # Replace localhost:9067 with the address and port of your gRPC server if using a custom port
        grpc_pass grpc://localhost:9067;
    }
}
```

##### b. Use without TLS certificate
You can run lightwalletd without TLS and server traffic over `http`. This is recommended only for local testing

#### 3. Run the frontend:
You can run the gRPC server with or without TLS, depending on how you configured step 2. If you are using NGINX as a reverse proxy and are letting NGINX handle the TLS authentication, then run the frontend with `-no-tls`

```
go run main.go --grpc-bind-addr 127.0.0.1:9067 --pirate-conf-path ~/.komodo/PIRATE/PIRATE.conf --no-tls-very-insecure
```

If you have a certificate that you want to use (either self signed, or from a certificate authority), pass the certificate to the frontend:

```
go run main.go --grpc-bind-addr 127.0.0.1:443 --pirate-conf-path ~/.komodo/PIRATE/PIRATE.conf  --tls-cert cert.pem --tls-key key.pem
```

You should start seeing the frontend ingest and cache the pirate blocks after ~15 seconds. 

#### 4. Point the `arrrrwallet-cli` to this server
Connect to your server!
```
./arrrrwallet-cli -server https://mylightwalletd.server.com:443
```

If you are using your own server running without TLS, you can also connect over `http`

```
./arrrwallet-cli --server http://127.0.0.1:9067
```
