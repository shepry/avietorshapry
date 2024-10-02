<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Rocket Game</title>
    <style>
        body {
            background-color: #1f1e25;
            color: white;
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
        }
        #gameContainer {
            position: relative;
            margin-bottom: 20px;
        }
        #game {
            width: 600px;
            height: 500px;
            background-color: #2c2b33;
        }
        #currentMultiplier {
            position: absolute;
            top: 130px;
            left: 90px;
            font-size: 50px;
        }
        #infoContainer {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 10px;
        }
        #balance {
            color: lime;
            font-weight: bold;
            margin-right: 5px;
        }
        #crashedAt {
            margin-right: 5px;
        }
        #betContainer {
            display: flex;
            align-items: center;
            margin-bottom: 10px;
        }
        #betAmount {
            width: 40px;
            padding: 5px;
        }
        #buttonContainer {
            margin-bottom: 10px;
            width: 400px;
        }
        #takeProfits,
        #mpesaPayment {
            padding: 10px 20px;
            font-size: 16px;
            background-color: #4caf50;
            border: none;
            color: white;
            cursor: pointer;
            transition: background-color 0.3s ease;
        }
        #takeProfits:hover,
        #mpesaPayment:hover {
            background-color: #45a049;
        }
        #lastCrashesContainer {
            margin-bottom: 20px;
            overflow-x: auto;
            white-space: nowrap;
        }
        #lastCrashesContainer p {
            font-weight: bold;
        }
        #lastCrashes {
            display: flex;
            gap: 10px;
        }
        .crashItem {
            display: inline-block;
            padding: 5px;
            border: 1px solid #444;
            border-radius: 5px;
        }
        #countdown {
            font-size: 30px;
            color: yellow;
            margin-bottom: 20px;
        }
    </style>
</head>
<body>

    <div id="lastCrashesContainer">
        <p>Last Crashes:</p>
        <div id="lastCrashes"></div>
    </div>

    <div id="countdown">Starting in: 5</div>

    <div id="gameContainer">
        <canvas id="game" width="600" height="500"></canvas>
        <p id="currentMultiplier">0.0x</p>
    </div>

    <div id="infoContainer">
        <div style="display: flex;">
            <p id="balance">?</p>
            <p id="crashedAt">Multiplier at crash:</p>
        </div>
        <div id="betContainer">
            <label for="betAmount">Bet Amount:</label>
            <input type="number" id="betAmount" value="1">
        </div>
    </div>

    <div id="buttonContainer">
        <button id="takeProfits">Take Profits</button>
        <button id="mpesaPayment">Pay with M-Pesa</button>
    </div>

    <script>
        const canvas = document.getElementById('game');
        const ctx = canvas.getContext('2d');
        ctx.strokeStyle = 'blue';
        ctx.lineWidth = 2;

        let speed = 0.01;
        let curvePoints = [];
        let gameLoop;
        let crashed = false;
        let balance = 1000;
        let betAmount;
        let profitsTaken = false;
        let rocketPosition;

        const balanceDisplay = document.getElementById('balance');
        balanceDisplay.innerText = balance + '$';
        const betAmountInput = document.getElementById('betAmount');

        function updateCurve() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.beginPath();
            for (let i = 0; i < curvePoints.length; i++) {
                const { x, y } = curvePoints[i];
                ctx.lineTo(x, y);
            }
            ctx.stroke();
            if (!crashed) {
                const lastPoint = curvePoints[curvePoints.length - 1];
                ctx.font = '30px Arial';
                ctx.fillText('🚀', lastPoint.x, lastPoint.y - 10);
                rocketPosition = { x: lastPoint.x, y: lastPoint.y };
            } else {
                const crashPosition = { x: rocketPosition.x, y: rocketPosition.y - 10 };
                ctx.font = '30px Arial';
                ctx.fillText('💥', crashPosition.x, crashPosition.y);
            }
        }

        function updateCurvePoints() {
            if (!crashed) {
                const lastPoint = curvePoints[curvePoints.length - 1];
                const newX = lastPoint.x + 1;
                const newY = canvas.height - (Math.pow(newX, 1.75) / 100);
                document.getElementById('currentMultiplier').innerText = (curvePoints.length / 100).toFixed(2) + 'x';
                curvePoints.push({ x: newX, y: newY });
            }
        }

        function startGame() {
            curvePoints = [{ x: 0, y: canvas.height }];
            crashed = false;
            profitsTaken = false;
            updateCurve();
            gameLoop = setInterval(() => {
                updateCurvePoints();
                updateCurve();
            }, 16);
            setTimeout(() => crashCurve(betAmountInput.value), getRandomCrashTime(1, 5.5));
        }

        function getRandomCrashTime(min, max) {
            const randomDecimal = Math.random();
            return (randomDecimal * (max - min) + min) * 1000;
        }

        function crashCurve(betAmount) {
            clearInterval(gameLoop);
            crashed = true;
            const crashValue = (curvePoints.length / 100).toFixed(2);
            document.getElementById('crashedAt').innerText = `Multiplier at crash: ${crashValue}`;
            const crash = document.createElement('div');
            crash.className = 'crashItem';
            crash.innerText = crashValue;
            crash.style.color = profitsTaken ? 'lime' : 'red';
            document.getElementById('lastCrashes').appendChild(crash);
            updateCurve();

            // Remove the first crash item if it goes out of the view
            const crashItems = document.getElementsByClassName('crashItem');
            if (crashItems.length > 5) { // Adjust the number based on how many items you want to display
                crashItems[0].remove();
            }

            // Restart the game after 3 seconds
            setTimeout(initiateCountdown, 3000);
        }

        function initiateCountdown() {
            let countdownValue = 5;
            const countdownElement = document.getElementById('countdown');
            countdownElement.style.display = 'block';
            countdownElement.innerText = `Starting in: ${countdownValue}`;
            const countdownInterval = setInterval(() => {
                countdownValue -= 1;
                countdownElement.innerText = `Starting in: ${countdownValue}`;
                if (countdownValue <= 0) {
                    clearInterval(countdownInterval);
                    countdownElement.style.display = 'none';
                    startGame();
                }
            }, 1000);
        }

        document.getElementById('takeProfits').addEventListener('click', function () {
            if (!crashed && !profitsTaken) {
                profitsTaken = true;
                const profit = (betAmount * (curvePoints.length / 100)) - betAmount;
                balance += (betAmount * (curvePoints.length / 100));
                balanceDisplay.innerText = balance.toFixed(2) + '$';
                console.log(`Took profits, multiplier: ${(curvePoints.length / 100).toFixed(2)}, profit: ${profit}`);
            } else if (profitsTaken) {
                console.log('Profits already taken');
            }
        });

        document.getElementById('mpesaPayment').addEventListener('click', function() {
            const amount = betAmount; // Use the current bet amount
            const phoneNumber = prompt("Enter your phone number:");

            if (phoneNumber) {
                fetch('/initiate-mpesa-payment', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        amount: amount,
                        phone: phoneNumber
                    }),
                })
                .then(response => response.json())
                .then(data => {
                    console.log('M-Pesa payment response:', data);
                    alert("Payment successful!");
                })
                .catch(error => {
                    console.error('Error:', error);
                    alert("An error occurred while initiating the payment.");
                });
            }
        });

        // Start the countdown and then the game
        initiateCountdown();
    </script>
</body>
</html>
