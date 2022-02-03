<template>
    <h2>Welcome to Websockets</h2>
    <p>You are: {{ connectedStatus }}</p>
    <p>Your message is: {{ message }}</p>
</template>

<script>

export default {
    name: 'App',
    data() {
        return {
            socket: {},
            connectedStatus: 'Not connected!',
            message: 'No message yet!'
        }
    },
    async mounted() {
        // Calculate the URL for the websocket. If you have a fixed URL, then you can remove all this and simply put in
        // ws://your-url-here.com or wss:// for secure websockets.
        const socketProtocol = (window.location.protocol === 'https:' ? 'wss:' : 'ws:')
        const port = ':3000';
        const echoSocketUrl = socketProtocol + '//' + window.location.hostname + port + '/ws'
    
        // Define socket and attach it to our data object
        this.socket = await new WebSocket(echoSocketUrl); 

        // When it opens, console log that it has opened. and send a message to the server to let it know we exist
        this.socket.onopen = () => {
            console.log('Websocket connected.');
            this.connectedStatus = 'Connected';
            this.sendMessage(JSON.stringify({"message" : "Hello, server."}));
        }

        // When we receive a message from the server, we can capture it here in the onmessage event.
        this.socket.onmessage = (event) => {
            // We can parse the data we know to be JSON, and then check it for data attributes
            let parsedMessage = JSON.parse(event.data);
            // If those data attributes exist, we can then console log or show data to the user on their web page.
            console.log(parsedMessage);
            if(typeof parsedMessage.message !== "undefined" && parsedMessage.message == "hello") {
                this.message = parsedMessage.message;
                console.log('We have received a message from the server!')
            }
        }
    },
    methods: {
        waitForOpenConnection: function() {
            // We use this to measure how many times we have tried to connect to the websocket server
            // If it fails, it throws an error.
            return new Promise((resolve, reject) => {
                const maxNumberOfAttempts = 10
                const intervalTime = 200 

                let currentAttempt = 0
                const interval = setInterval(() => {
                    if (currentAttempt > maxNumberOfAttempts - 1) {
                        clearInterval(interval)
                        reject(new Error('Maximum number of attempts exceeded.'));
                    } else if (this.socket.readyState === this.socket.OPEN) {
                        clearInterval(interval)
                        resolve()
                    }
                    currentAttempt++
                }, intervalTime)
            })
        },
        sendMessage: async function(message) {
            // We use a custom send message function, so that we can maintain reliable connection with the
            // websocket server.
            if (this.socket.readyState !== this.socket.OPEN) {
                try {
                    await this.waitForOpenConnection(this.socket)
                    this.socket.send(message)
                } catch (err) { console.error(err) }
            } else {
                this.socket.send(message)
            }
        }
    }
}
</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
</style>
