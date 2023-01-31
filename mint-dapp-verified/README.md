# Example Verified Mint DApp using Concordium Identity Feature
The service are not full production ready services. They are examples to
demonstrate some specific use-cases. These are provided in the hope that they
can be expanded or evolved into full-fledged services, or just to serve as a
guide in what needs to be done.

## Setup

Some projects uses submodules, to initialize or update those, run:
```
git submodule update --init --recursive
```

## Run
### Run the [backend](./backend/verifier/README.md) using [docker compose](https://docs.docker.com/compose/)
```back
docker-compose build
docker-compose up
```
This runs the backend and the concordium testnet node.
**Allow the node to catchup fully. This might take a few days**

### Run the [frontend](./mint-ui/README.md).

