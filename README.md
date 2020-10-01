# Example of Janus WebRTC streaming with Vue.js

This project is a simple example of streaming using [Janus WebRTC server](https://janus.conf.meetecho.com/) as a backend and [Vue.js](https://vuejs.org/) as a frontend.

You will need Janus WebRTC server installed somewhere to run this project. You can refer to [an official document](https://github.com/meetecho/janus-gateway/blob/master/README.md) to know how to install Janus WebRTC server.

My blog might help you know about this exmample.

* https://www.mikan-tech.net/entry/janus-vue-frontend-install

## How to run

Janus WebRTC server is required to use this example.
To specify Janus WebRTC server location, please edit below line in `src/App.vue`.

```
const JANUS_URL = 'http://raspberrypi.local:8088/janus'
```

To setup this project, run

```
$ yarn install
```

and to compile and hot-reload for development, run

```
$ yarn serve
```

And if you want to compile and minifies for production, run

```
$ yarn build
```
