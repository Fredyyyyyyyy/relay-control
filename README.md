
<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ovládání relé ESP32</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js" type="text/javascript"></script>
    <style>
        /* ... (CSS zůstává stejný) ... */
    </style>
</head>
<body>
    <div class="container">
        <h1>Ovládání relé ESP32</h1>
        <button onclick="sendMessage('ON')">Zapnout</button>
        <button onclick="sendMessage('OFF')">Vypnout</button>
        <div id="status">Stav připojení: Odpojeno</div>
        <div id="esp32Status">Stav ESP32: Neznámý</div>
        <div id="relayStatus">Stav relé: Neznámý</div>
        <button onclick="reconnect()">Obnovit připojení</button>
        <div id="log"></div>
    </div>

    <script>
        const deviceId = "esp32_" + Math.random().toString(36).substring(2, 15);
        const baseTopic = "unique_relay_control/" + deviceId;
        const relayTopic = baseTopic + "/relay";
        const statusTopic = baseTopic + "/status";

        // Použití WSS (WebSocket Secure) pro MQTT připojení
        var client = new Paho.MQTT.Client("broker.emqx.io", 8084, "webclient_" + deviceId);
        client.onConnectionLost = onConnectionLost;
        client.onMessageArrived = onMessageArrived;
        var lastMessageTime = 0;
        var checkConnectionInterval;

        function connect() {
            log("Pokus o připojení k MQTT brokeru...");
            client.connect({
                useSSL: true,
                onSuccess: onConnect,
                onFailure: onConnectFailure,
                keepAliveInterval: 60
            });
        }

        function onConnect() {
            log("Připojeno k MQTT brokeru");
            document.getElementById('status').innerHTML = "Stav připojení: Připojeno";
            document.getElementById('status').style.backgroundColor = "rgba(46, 204, 113, 0.2)";
            client.subscribe(relayTopic);
            client.subscribe(statusTopic);
            checkConnectionInterval = setInterval(checkConnection, 15000);
        }

        function onConnectFailure(responseObject) {
            log("Chyba připojení k MQTT brokeru: " + responseObject.errorMessage);
            document.getElementById('status').innerHTML = "Stav připojení: Chyba připojení";
            document.getElementById('status').style.backgroundColor = "rgba(231, 76, 60, 0.2)";
        }

        function onConnectionLost(responseObject) {
            if (responseObject.errorCode !== 0) {
                log("Ztráta spojení: " + responseObject.errorMessage);
                document.getElementById('status').innerHTML = "Stav připojení: Odpojeno";
                document.getElementById('status').style.backgroundColor = "rgba(231, 76, 60, 0.2)";
                document.getElementById('esp32Status').innerHTML = "Stav ESP32: Neznámý";
                clearInterval(checkConnectionInterval);
            }
        }

        function onMessageArrived(message) {
            log("Přijata zpráva: " + message.destinationName + " - " + message.payloadString);
            lastMessageTime = Date.now();
            if (message.destinationName === relayTopic) {
                document.getElementById('relayStatus').innerHTML = "Stav relé: " + (message.payloadString === "ON" ? "Zapnuto" : "Vypnuto");
                document.getElementById('relayStatus').style.backgroundColor = message.payloadString === "ON" ? "rgba(46, 204, 113, 0.2)" : "rgba(231, 76, 60, 0.2)";
            } else if (message.destinationName === statusTopic) {
                document.getElementById('esp32Status').innerHTML = "Stav ESP32: " + message.payloadString;
                document.getElementById('esp32Status').style.backgroundColor = message.payloadString === "online" ? "rgba(46, 204, 113, 0.2)" : "rgba(231, 76, 60, 0.2)";
            }
        }

        function sendMessage(state) {
            if (!client.isConnected()) {
                log("Nelze odeslat zprávu. MQTT klient není připojen.");
                return;
            }
            var message = new Paho.MQTT.Message(state);
            message.destinationName = relayTopic;
            client.send(message);
            log("Odeslána zpráva: " + state);
        }

        function checkConnection() {
            if (Date.now() - lastMessageTime > 30000) {
                log("Žádná zpráva přijata v posledních 30 sekundách.");
                document.getElementById('esp32Status').innerHTML = "Stav ESP32: Pravděpodobně offline";
                document.getElementById('esp32Status').style.backgroundColor = "rgba(231, 76, 60, 0.2)";
            }
            if (!client.isConnected()) {
                log("MQTT klient není připojen. Pokus o obnovení připojení.");
                reconnect();
            }
        }

        function reconnect() {
            if (client.isConnected()) {
                client.disconnect();
            }
            log("Pokus o obnovení připojení...");
            connect();
        }

        function log(message) {
            var logElement = document.getElementById('log');
            logElement.innerHTML += new Date().toLocaleTimeString() + ": " + message + "<br>";
            logElement.scrollTop = logElement.scrollHeight;
        }

        connect();
    </script>
</body>
</html>
