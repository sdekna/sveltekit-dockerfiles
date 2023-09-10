
# SvelteKit Dockerfiles

The following are dockerfiles that could be used to build sveltekit apps images using different adapters.
## Table of Contents 
- [Using Node/Auto Adapter](#using-nodeauto-adapter)
- [Using Bun Adapter](#using-bun-adapter)
- [Using Static Adapter (and nginx)](#using-static-adapter-and-nginx)
### Using Node/Auto Adapter

1. Create a Dockerfile with the following content:
```Dockerfile
FROM node:alpine

WORKDIR /app
COPY package.json ./
RUN npm install

COPY . .
RUN npm run build

CMD ["node", "build"]

EXPOSE 3000
```

2. Build the Docker image:
`docker build -t sveltekit-node ./adapter-node.Dockerfile`

3. Run the Docker container:
`docker run -p 3000:3000 sveltekit-node`
App should now be accessible at `http://localhost:3000`.


### Using Bun Adapter

1. Create a Dockerfile with the following content:
```Dockerfile
FROM oven/bun

WORKDIR /app
COPY package.json package.json
RUN bun install

COPY . .
RUN bun run build

EXPOSE 3000
ENTRYPOINT ["bun", "./build"]
```

2. Build the Docker image:
`docker build -t sveltekit-bun ./adapter-bun.Dockerfile`

3. Run the Docker container:
`docker run -p 3000:3000 sveltekit-bun`
App should now be accessible at `http://localhost:3000`.


### Using Static Adapter (and nginx)
Since static adapter outputs static html/js files/assets, we can use `nginx` to serve the project. The project is first built using `node` image, and then uses `nginx` image to serve the built files... giving a much smaller image size.

1. Create a Dockerfile with the following content:
```Dockerfile
FROM node:alpine as build

ADD . /app
WORKDIR /app

RUN npm install
RUN npm run build

FROM nginx:stable
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/build /usr/share/nginx/html
```
2. Create an `nginx.conf` file in the same directory as the `Dockerfile` with the following content:
```
server {
    listen 80;
    listen [::]:80;
    server_name _;

    location / {
      root /usr/share/nginx/html;
      try_files $uri $uri/index.html $uri.html /index.html;
    }

    include mime.types;
    types {
        application/javascript js mjs;
    }
}
```
2. Build the Docker image:
`docker build -t sveltekit-static ./adapter-static.Dockerfile`

The image exposes port 80, so to run it on port 3000 locally:
`docker run -p 3000:80 sveltekit-static`
App should now be accessible at `http://localhost:3000`.

Different projects may require different dependencies and dockerization techniques, the above is one of the ways you can achieve this.

Please feel very welcome to contribute/correct/update/enhance.
