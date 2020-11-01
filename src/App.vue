<template>
    <div id="app">
        <div class="websocket" v-if="!hasConnected">
            <img alt="Vue logo" src="./assets/logo.svg" width="400" />
            <h1>Welcome!</h1>
            <p v-if="errorMessage">{{ errorMessage }}</p>
            <p>Your Local IP is: {{ ip }}</p>
            <template v-if="!connecting">
                <p>Please enter your parties host and port.</p>
                <div class="inputs">
                    <input v-model="name" placholder="NAME" />
                    <input v-model="ip" placeholder="IP ADDRESS" />
                    <input v-model="port" placeholder="PORT" />
                </div>
                <button @click="handleConnect">Connect</button>
            </template>
            <template v-else>
                <p>You are now connecting... please wait.</p>
                <img alt="loading" src="./assets/loading.gif" width="25" />
                <br />
                <button @click="stopConnection">Stop Connection</button>
            </template>
        </div>
        <router-view v-else :session="session"></router-view>
    </div>
</template>

<script>
    function setIP(json) {
        console.log(json);
    }

    export default {
        name: 'App',
        data() {
            return {
                name: 'scrublord',
                ip: 'localhost',
                port: 3000,
                hasConnected: false,
                connecting: false,
                ws: null,
                errorMessage: null,
                session: null
            };
        },
        computed: {
            getSession() {
                return this.session;
            }
        },
        methods: {
            handleConnect() {
                this.$errorMessage = null;

                if (this.hasConnected) {
                    console.log(`Already connected!`);
                    return;
                }

                if (!this.connecting) {
                    this.connecting = true;
                }

                console.log(`Connecting...`);
                this.ws = new WebSocket(`ws://${this.ip}:${this.port}`);
                this.ws.addEventListener('open', this.handleOpen);
                this.ws.addEventListener('error', this.handleError);
                this.ws.addEventListener('close', this.handleClose);
                this.ws.addEventListener('message', this.routeMessage);
            },
            stopConnection() {
                this.closeSession();
            },
            handleError(e) {
                this.removeListeners();
                this.ws.close();
                this.handleConnect();
            },
            handleOpen() {
                this.hasConnected = true;
                this.sendData(`session`, { name: this.name });
                console.log(`Connected!`);
            },
            handleClose() {
                this.closeSession();
                this.ws.close();
            },
            sendData(route, dataObject) {
                this.ws.send(JSON.stringify({ route, ...dataObject }));
            },
            routeMessage(event) {
                if (!event.data) {
                    return;
                }

                let dataObject;

                try {
                    dataObject = JSON.parse(event.data);
                } catch (err) {
                    throw err;
                }

                if (!dataObject.route) {
                    console.error(`No route was provided.`);
                    return;
                }

                if (dataObject.route === 'session' && dataObject.status) {
                    console.log(dataObject.msg);
                    this.session = dataObject.hash;
                    return;
                }

                if (dataObject.route === 'navigate') {
                    try {
                        this.$router.push(dataObject.data);
                    } catch (err) {}
                    return;
                }

                this.$event.$emit(dataObject.route, dataObject.data);
            },
            removeListeners() {
                this.ws.removeEventListener('open', this.handleOpen);
                this.ws.removeEventListener('close', this.handleClose);
                this.ws.removeEventListener('error', this.handleError);
                this.ws.removeEventListener('message', this.routeMessage);
            },
            closeSession() {
                this.removeListeners();
                this.hasConnected = false;
                this.connecting = false;
                this.errorMessage = `Lost Connection!`;
                this.session = null;
                try {
                    this.$router.push('/');
                } catch (err) {}
            },
            async fetchIP() {
                const response = await fetch(`https://api.ipify.org?format=json`);
                const data = await response.json();
                this.ip = data.ip;
            },
            setIP(json) {
                console.log(json);
                this.ip = json.ip;
            }
        },
        mounted() {
            this.ip = 'localhost';
            this.port = `3000`;
            this.hasConnected = false;
            this.$event.$on('send', this.sendData);
            this.fetchIP();

            if (!this.session && !window.location.href !== '/#/') {
                window.location.href = '/#/';
            }
        }
    };
</script>

<style lang="less">
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
    }

    #nav {
        padding: 30px;

        a {
            font-weight: bold;
            color: #2c3e50;

            &.router-link-exact-active {
                color: #42b983;
            }
        }
    }
</style>
