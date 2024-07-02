
<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ovládání relé ESP32</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js" type="text/javascript"></script>
    <style>
        /* ... (štýly zostávajú nezmenené) ... */
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

        var client = new Paho.MQTT.Client("broker.emqx.io", 8083, "webclient_" + deviceId);
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
                keepAliveInterval: 10
            });
        }

        function onConnect() {
            log("Připojeno k MQTT brokeru");
            document.getElementById('status').innerHTML = "Stav připojení: Připojeno";
            client.subscribe(relayTopic);
            client.subscribe(statusTopic);
            checkConnectionInterval = setInterval(checkConnection, 15000);
        }

        function onMessageArrived(message) {
            log("Přijata zpráva: " + message.destinationName + " - " + message.payloadString);
            lastMessageTime = Date.now();
            if (message.destinationName === relayTopic) {
                document.getElementById('relayStatus').innerHTML = "Stav relé: " + (message.payloadString === "ON" ? "Zapnuto" : "Vypnuto");
            } else if (message.destinationName === statusTopic) {
                document.getElementById('esp32Status').innerHTML = "Stav ESP32: " + message.payloadString;
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

        // ... (ostatné funkcie zostávajú nezmenené) ...

        connect();
    </script>
</body>
</html>
