# Github Structure

We decided to use a GitHub organisation so that we could create more than one repository and not couple it to single user account.

To keep things simple with this solution we kept a simple naming convetion for the repositories, frontend for the website and backend for the api and llm backend for the api responsible for serving the LLM, we then decided that we should a 4th repository hosting all repositories as submodules and providing a docker-compose file to make running it simple for a developer to start all components in one place and just stop the once that are needed to be run outside of docker for development purpoposes.

## Frontend

The frontend is a Vue app and has largely follwed common conventions for creating a vue app in layout and structure.

## Backend

The backend is written with rust and has several times gone through the code formater and static analyzer to make sure the quality is up to spec with Rust coding conventions, we also have a clear structure of the project grouping as many things in single places making it simpler to find or refactor single components of the backend.

## Llm backend

Smaller scale of the backend structure.

---

# Short introduction

We are a team of five MSc students in computer security at Blekinge Institute of Technology: Emma Ehn Paulsson, Muaath Awad, Gustav Karlsson, Emil Schütt, and Oskar Ohvo. Our project, ThreatMapper, is an LLM-powered app designed to improve threat modeling by supporting text input and providing more robust security analysis.

---

# Installation & Usage

There are two ways of running this application, natively or in Docker. When running in Docker we cant use a AMD or Apple accelerator or device, so we need to run the LLM on the CPU unfortunately.

## Clone Repositories

*Note: This guide assumes you have setup a SSH key on GitHub, else you will need to use a PAT (Personel Access Token) and replace the SSH clone URLs with HTTPS equivalent from each repository site on GitHub*

First step for either Native or Docker install is almost the same, we need clone the repositories. The difference is how we clone each repository.

### Native

Setting the project up natively requires us to manually clone all repos.

```bash
> git clone git@github.com:DV1512/backend.git
> git@github.com:DV1512/llm_backend.git
> git@github.com:DV1512/frontend.git && git submodule update --init --recursive
```

### In Docker

When running on Docker its a bit simpler with only one repo needed to be downloaded.

```bash
> git@github.com:DV1512/dev.git && git submodule update --init --recursive
```

---

## On Native

When running natively we need to consider a few things, first the amount of tools need to compile and run the binaries are significantly more compared to running on Docker but the potential performance improvements are night and day.

### Requirements

- [Rust](#rust)
- [Bun](#bun)
- [wasm-pack](#wasm)

There might be some packages that are needed and are platform specific, in that case we would recommend either contacting us for support in debugging which packages are needed, best way to do that is to send an email to <emsu21@student.bth.se>, or make sure you have the standard platform dev tools, for common platforms see below.

##### Ubuntu

`> sudo apt install ca-certificates build-essential pkg-config libssl-dev libclang-dev curl`

##### MacOS

```bash
> xcode-select --install
> brew install openssl
```

### Backend

To start we need to setup the backend with its needed .env variables before running the backend.

#### Create a new `.env` file

create a file called `.env` and insert the following to it:

```env
GOOGLE_CLIENT_ID="<YOUR GOOGLE CLIENT ID>"
GOOGLE_CLIENT_SECRET="<YOUR GOOGLE CLIENT SECRET>"
GITHUB_CLIENT_ID="<YOUR GITHUB CLIENT ID>"
GITHUB_CLIENT_SECRET="<YOUR GITHUB CLIENT SECRET>"
```

It is not needed to have these populated with actual credentials, not populating them will only make it so login/signup using google or github will fail, but not including them at all will cause the program to exit since its needed internally.

#### Start Database

To make things simpler we have created a docker compose script to run database and potentially the backend. To only run the database run the command

`docker compose start db`

which will start a database instance.

#### Running

To run it now we run the command:

`> cargo run --release`

This will compile the binary and run it directly afterwards, the compiled binary is located at `target/release` and can be manually run as long as the three `.env` files are present in the current directory.

### LLM Backend

The LLM backend is a bit special since it needs to support multiple platforms and therefore needs some more configurations.

#### Enable GPU Acceleration (Optional)

If you want to run on a GPU we need to change a feature flag on one of the dependencies.

1. Open the `Cargo.toml` file
2. Find the line `kalosm = { git = "https://github.com/floneum/floneum", version = "0.3.2", features = ["language"]}
`
3. Add the feature `metal` or `cuda` based on the accelerator you have. *Note: it might require external packages or tooling to run on the accelerator*

##### Example

Here we have activated the `metal` accelerator for apple devices using the M-series chip

```toml
kalosm = { git = "https://github.com/floneum/floneum", version = "0.3.2", features = ["language", "metal"]}
```

#### Running

Similar to the backend we run it using

`cargo run --release`

### Frontend

#### Building SDK library

We are partially using a custom web-assembly package to communicate with the backend and to install it locally we need to first build it locally

1. Move to the `sdk` directory
2. Run `wasm-pack build --features wasm --no-default-features`
3. Move back to the root of the repository

#### Installing Dependencies

To install dependencies we need to run

`> bun install`

This will install all needed dependencies

#### Running

##### Debug mode

To run in debug mode we run

`> bun dev`

this will host a frontend on  [http://localhost:5173](http://localhost:5173) and will have extra debug tools attached to help inspect internal data stored on the client

##### Production mode

To run in production we first need to bundle the app

`> bun run build`

Then we need to serve it

`> cd dist && bunx serve`

---

## Docker

Running on Docker is significantly simpler to do and will require two simple steps, setting up dependencies and running/building the containers.

### Requirements

- [Rust](#rust)
- [wasm-pack](#wasm-pack)
- [Docker](#docker)
- [Docker Compose](#docker-compose)

### Building SDK library

Navigate to the directory `frontend/sdk` and run

`wasm-pack build --features wasm --no-default-features`

then move back to the root of repository.

### Build & run the containers

To build all containers and start them run  

`docker compose up --build`

This will take quite some time to build all images since it needs compile both backends which takes up to 30 minutes depending on hardware.

## External Tools

### Rust

Follow the instructions found [here](https://www.rust-lang.org/learn/get-started)

### Wasm-pack

Make sure you have [Rust](#Rust-install-guide) installed and then run `cargo install wasm-pack`

### Docker

Follow the instructions found [here](https://docs.docker.com/engine/install/) or [here](https://docs.docker.com/desktop/)

### Docker Compose

*Note: Docker compose is included in most cases with [Docker](#Docker-install-guide)*

Follow the instructions found [here](https://docs.docker.com/compose/install/)

### Bun

Follow the instructions found [here](https://bun.sh/)

---

# OpenAPI

We have implemented OpenAPI documentation for our backend API, this allowed us to share API structure between the frontend team and backend team making integration simple and straight forward.

There are several versions of the documentation, to allow the user to decide for themselves what UI to use for the OpenAPI documentation we decided to host 4 different ones, Swagger, ReDoc, RapiDoc and Scalar.

This documentation has allowed the frontend team to make sure their requests are structured correctly and resolve any issues with incorrect urls and contents being sent without talking to the backend team or looking it up themself in the source code.

## Endpoints

All documentation is hosted with the Main backend.

- [Swagger (http://localhost:9999/api/docs/swagger/)](http://localhost:9999/api/docs/swagger/)
- [Scalar (http://localhost:9999/api/docs/scalar)](http://localhost:9999/api/docs/scalar)
- [ReDoc (http://localhost:9999/api/docs/redoc)](http://localhost:9999/api/docs/redoc)
- [RapiDoc (http://localhost:9999/api/docs/rapidoc)](http://localhost:9999/api/docs/rapidoc)
