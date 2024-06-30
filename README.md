<!DOCTYPE html>
<html>
<head>
    <title>Ovládání relé</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.min.js" type="text/javascript"></script>
    <script>
        var client = new Paho.MQTT.Client("broker.hivemq.com", 8000, "webclient_" + parseInt(Math.random() * 100, 10));
        client.onConnectionLost = onConnectionLost;
        client.onMessageArrived = onMessageArrived;

        function connect() {
            client.connect({onSuccess:onConnect});
        }

        function onConnect() {
            console.log("Připojeno k MQTT brokeru");
        }

        function onConnectionLost(responseObject) {
            if (responseObject.errorCode !== 0) {
                console.log("Ztráta spojení:"+responseObject.errorMessage);
            }
        }

        function onMessageArrived(message) {
            console.log("Přijata zpráva: "+message.payloadString);
        }

        function sendMessage(state) {
            var message = new Paho.MQTT.Message(state);
            message.destinationName = "b3zp3cn3_t3ma_123/rele1";  // Bezpečnější, unikátní téma
            client.send(message);
        }
    </script>
</head>
<body onload="connect();">
    <h1>Ovládání relé</h1>
    <button onclick="sendMessage('ON')">Zapnout</button>
    <button onclick="sendMessage('OFF')">Vypnout</button>
</body>
</html>
