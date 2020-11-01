<template>
    <div class="websocket">
        <h1>Welcome!</h1>
        <p>Please enter your parties host and port.</p>
        <div class="inputs">
            <input v-model="ip" placeholder="IP ADDRESS" />
            <input v-model="port" placeholder="PORT" />
        </div>
        <button @click="handleConnect">Connect</button>
    </div>
</template>

<script>
export default {
    name: 'Connection',
    data() {
        return {
            ip: 'localhost',
            port: 3000,
            hasConnected: false,
            ws: null
        };
    },
    methods: {
        handleConnect() {
            if (this.hasConnected) {
                console.log(`Already connected!`);
                return;
            }

            console.log(`Connecting...`);
            this.ws = new WebSocket(`ws://${this.ip}:${this.port}`);
            this.ws.addEventListener('open', this.handleOpen);
            this.ws.addEventListener('error', this.handleError);
            this.ws.addEventListener('close', this.handleClose);
            this.ws.addEventListener('message', this.routeMessage);
        },
        handleError(e) {
            this.removeListeners();
            this.ws.close();
            this.handleConnect();
        },
        handleOpen() {
            this.hasConnected = true;
            console.log(`Connected!`);
        },
        handleClose() {
            this.hasConnected = false;
            this.removeListeners();
            this.ws.close();
            console.log(`Retrying...`);
            this.handleConnect();
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

            this.$event.$emit(dataObject.route, dataObject.data);
        },
        removeListeners() {
            this.ws.removeEventListener('open', this.handleOpen);
            this.ws.removeEventListener('close', this.handleClose);
            this.ws.removeEventListener('error', this.handleError);
            this.ws.removeEventListener('message', this.routeMessage);
        }
    },
    mounted() {
        this.ip = 'localhost';
        this.port = `3000`;
        this.hasConnected = false;
        this.ws = null;
    }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="less">
h3 {
    margin: 40px 0 0;
}
ul {
    list-style-type: none;
    padding: 0;
}
li {
    display: inline-block;
    margin: 0 10px;
}
a {
    color: #42b983;
}
</style>
