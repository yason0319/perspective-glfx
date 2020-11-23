# trial-app

## Build Setup

```bash
# make cert file for debug on tablet or smartphone
$ brew install mkcert
$ mkcert -install

$ mkdir cert
$ cd cert
$ mkcert localhost
# localhost-key.pemとlocalhost.pemがcertディレクトリ内にあればOK

# install dependencies
$ npm install

# serve with hot reload at localhost:3000
$ npm run dev

# build for production and launch server
$ npm run build
$ npm run start

# generate static project
$ npm run generate
```

For detailed explanation on how things work, check out [Nuxt.js docs](https://nuxtjs.org).
