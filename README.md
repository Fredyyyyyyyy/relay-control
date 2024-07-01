<!DOCTYPE html>
<html lang="cs">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ovládání relé ESP32</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js" type="text/javascript"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
        }
        .container {
            text-align: center;
            background-color: white;
            padding: 2rem;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }
        button {
            font-size: 1rem;
            padding: 0.5rem 1rem;
            margin: 0.5rem;
            cursor: pointer;
            background-color: #4CAF50;
            color: white;
            border: none;
            border-radius: 5px;
            transition: background-color 0.3s;
        }
        button:hover {
            background-color: #45a049;
        }
        #status, #relayStatus, #esp32Status {
            margin-top: 1rem;
            font-weight: bold;
        }
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
    </div>

    <script>
        var client = new Paho.MQTT.Client("broker.hivemq.com", 8000, "webclient_" + parseInt(Math.random() * 100, 10));
        client.onConnectionLost = onConnectionLost;
        client.onMessageArrived = onMessageArrived;
        var lastMessageTime = 0;
        var checkConnectionInterval;

        function connect() {
            client.connect({onSuccess:onConnect});
        }

        function onConnect() {
            console.log("Připojeno k MQTT brokeru");
            document.getElementById('status').innerHTML = "Stav připojení: Připojeno";
            client.subscribe("vasetema/rele1");
            client.subscribe("vasetema/rele1/status");
            checkConnectionInterval = setInterval(checkConnection, 15000);
        }

        function onConnectionLost(responseObject) {
            if (responseObject.errorCode !== 0) {
                console.log("Ztráta spojení: " + responseObject.errorMessage);
                document.getElementById('status').innerHTML = "Stav připojení: Odpojeno";
                document.getElementById('esp32Status').innerHTML = "Stav ESP32: Neznámý";
                clearInterval(checkConnectionInterval);
            }
        }

        function onMessageArrived(message) {
            console.log("Přijata zpráva: " + message.payloadString);
            lastMessageTime = Date.now();
            if (message.destinationName === "vasetema/rele1") {
                document.getElementById('relayStatus').innerHTML = "Stav relé: " + (message.payloadString === "ON" ? "Zapnuto" : "Vypnuto");
            } else if (message.destinationName === "vasetema/rele1/status") {
                document.getElementById('esp32Status').innerHTML = "Stav ESP32: " + message.payloadString;
            }
        }

        function sendMessage(state) {
            var message = new Paho.MQTT.Message(state);
            message.destinationName = "vasetema/rele1";
            client.send(message);
        }

        function checkConnection() {
            if (Date.now() - lastMessageTime > 30000) {
                document.getElementById('esp32Status').innerHTML = "Stav ESP32: Offline";
            }
        }

        connect();
    </script>
</body>
</html>
