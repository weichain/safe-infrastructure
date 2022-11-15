# Safe Infrastructure

# Deployment steps

## Service infrastructure

Here is a diagram showing us how the services work
![services architecture](<https://134244847-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F-MhyEZtd5TVytPJtyS7v-2910905616%2Fuploads%2Fgit-blob-bb8415e262a22ae903604c834d5655e8f32617fc%2FArchitecture%20docs%20diagrams(4).png?alt=media>)

For more details: https://docs.gnosis-safe.io/backend/service-architecture

## Requirements

- `docker compose` (installation [guide](https://docs.docker.com/compose/install/))

## Setting up Safe Infrastructure

Rename `.env.example` to `.env`. Infrastructure is basically setted up, all you have to do is run `docker compose up`, this will spin up the [safe-client-gateway](https://github.com/safe-global/safe-client-gateway) and [safe-config-service](https://github.com/safe-global/safe-config-service) services. After this you would have to create an superuser by doing `docker compose exec cfg-web python src/manage.py createsuperuser` , this is required since we would have to configure our chain in the config service.

### Add your `ChainInfo`

We need to be able to define a `ChainInfo` object in the Safe Config service so that the Client Gateway knows the URL of the Safe Transaction service instance it needs to request against for a given safe.

You can do this in the admin interface of the Safe Config service: `http://localhost:8000/cfg/admin/chains/chain/add/`

You can verify that your `ChainInfo` was successfully added by going to `http://localhost:8000/cfg/api/v1/chains`.

Remember to edit your `ChainInfo` json fields `transaction_service_uri` and `vpc_transaction_service_uri` to point to your local instance of the transaction service. The values should be `http://nginx:8085`. Note: The only way I managed to make it work here is by using ngrok and exposing the transaction service to a public network. For testing purposes install ngrok from [here](https://ngrok.com/download) and register so that you get an auth token. When all is done just execute `ngrok http 8085` and copy the link from `Forwarding` (should look something like `https://c900-46-10-125-139.eu.ngrok.io/`). Navigate back to `http://localhost:8000/cfg/admin/chains/chain/` and replace `transactionService` with the generated url from ngrok. To test if everything is working correctly visit `http://localhost:8000/cgw/v1/chains/43214913/about/master-copies` if you see:

```bash
[
    {
        "address": "0x33D1d44F564F9900C253d665C8bE5B78B015d20f",
        "version": "1.3.0+L2"
    }
]
```

this means that gateway, config and transaction services are working with eachother correctly.

`` Note: making changes to `http://localhost:8000/cfg/admin/chains/chain/` you should always follow it with docker compose down and docker compose up so that changes take an effect. ``

An example config for our testnet

```bash
{
    "transactionService": "https://c900-46-10-125-139.eu.ngrok.io/", # change this url to your own
    "chainId": "43214913",
    "chainName": "maistestsubnet",
    "shortName": "mai",
    "l2": true,
    "description": "",
    "rpcUri": {
        "authentication": "NO_AUTHENTICATION",
        "value": "http://174.138.9.169:9650/ext/bc/VUKSzFZKckx4PoZF9gX5QAqLPxbLzvu1vcssPG5QuodaJtdHT/rpc"
    },
    "safeAppsRpcUri": {
        "authentication": "NO_AUTHENTICATION",
        "value": "http://174.138.9.169:9650/ext/bc/VUKSzFZKckx4PoZF9gX5QAqLPxbLzvu1vcssPG5QuodaJtdHT/rpc"
    },
    "publicRpcUri": {
        "authentication": "NO_AUTHENTICATION",
        "value": "http://174.138.9.169:9650/ext/bc/VUKSzFZKckx4PoZF9gX5QAqLPxbLzvu1vcssPG5QuodaJtdHT/rpc"
    },
    "blockExplorerUriTemplate": {
        "address": "http://174.138.9.169:3006/address/{{address}}",
        "txHash": "http://174.138.9.169:3006/tx/{{txHash}}",
        "api": "http://174.138.9.169:3006/"
    },
    "nativeCurrency": {
        "name": "MAI",
        "symbol": "MAI",
        "decimals": 18,
        "logoUri": "/media/chains/43214913/currency_logo.png" # download some image from the web, the limit was 500x500px I believe.
    },
    "theme": {
        "textColor": "#ffffff",
        "backgroundColor": "#000000"
    },
    "gasPrice": [],
    "disabledWallets": [],
    "features": []
}
```

## Pinned versions

This repository contains the minimum viable local setup for our backend services.
The setup presented here, assumes that only L2 safes will be used. Additionally, we have tested this setup with the following versions:

- `CFG_VERSION=v2.26.0`
- `CGW_VERSION=v3.29.0`
- `TXS_VERSION=v4.6.1`

You can change them to the version you are interested available in [docker-hub](https://hub.docker.com/u/gnosispm) but be aware that not all versions of our services are compatible with each other, so do so **at your own risk.**

- [Brief Docker Crash Course](docker_cheatsheet.md)
- [Guide to run our services locally](running_locally.md)
