<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Desert Journey: Snake Game</title>
    <style>
        body {
            background-color: #e6c280; /* Desert sand color */
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
            color: #5c4033;
        }

        h1 {
            margin-bottom: 5px;
            font-size: 2.5rem;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.1);
        }

        p {
            margin-top: 0;
            font-style: italic;
            color: #7a5844;
        }

        #score-board {
            font-size: 1.2rem;
            font-weight: bold;
            margin-bottom: 15px;
        }

        canvas {
            background-color: #f4d090; /* Lighter sand for the board */
            border: 4px solid #8b5a2b;
            box-shadow: 0px 10px 20px rgba(0, 0, 0, 0.15);
            border-radius: 8px;
        }

        .controls {
            margin-top: 15px;
            font-size: 0.9rem;
            text-align: center;
            max-width: 400px;
            line-height: 1.4;
        }
    </style>
</head>
<body>

    <h1>Desert Journey</h1>
    <p>Guide the camel to gather the palm branches.</p>
    
    <div id="score-board">Branches Gathered: <span id="score">0</span></div>
    
    <canvas id="gameCanvas" width="400" height="400"></canvas>

    <div class="controls">
        <strong>Controls:</strong> Use the <strong>Arrow Keys</strong> or <strong>WASD</strong> to guide the camel. Avoid hitting the desert walls or yourself!
    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        const gridSize = 20;
        const tileCount = canvas.width / gridSize;

        // Game variables
        let camel = [
            {x: 10, y: 10},
            {x: 9, y: 10},
            {x: 8, y: 10}
        ];
        let food = {x: 5, y: 5};
        let dx = 1;
        let dy = 0;
        let score = 0;
        let gameSpeed = 120; // milliseconds per frame
        let gameLoopId;

        // Start game
        main();
        generateFood();

        function main() {
            if (hasGameEnded()) {
                alert(`Journey complete. You gathered ${score} palm branches.`);
                resetGame();
            }

            gameLoopId = setTimeout(function onTick() {
                clearCanvas();
                drawFood();
                moveCamel();
                drawCamel();
                main();
            }, gameSpeed);
        }

        function clearCanvas() {
            ctx.fillStyle = "#f4d090";
            ctx.fillRect(0, 0, canvas.width, canvas.height);
        }

        function drawCamel() {
            camel.forEach((part, index) => {
                if (index === 0) {
                    // Camel Head (Darker Brown, distinct shape)
                    ctx.fillStyle = "#8b5a2b";
                    ctx.fillRect(part.x * gridSize, part.y * gridSize, gridSize - 2, gridSize - 2);
                    
                    // Small eye
                    ctx.fillStyle = "#ffffff";
                    ctx.fillRect(part.x * gridSize + 12, part.y * gridSize + 4, 3, 3);
                } else {
                    // Camel Humps / Body (Alternating textures or lighter brown)
                    ctx.fillStyle = index % 2 === 0 ? "#a06d3b" : "#b37d4a";
                    
                    // Draw rounded top to look like a hump
                    ctx.beginPath();
                    ctx.arc(
                        part.x * gridSize + gridSize/2, 
                        part.y * gridSize + gridSize/2, 
                        gridSize/2 - 1, 
                        0, 
                        2 * Math.PI
                    );
                    ctx.fill();
                }
            });
        }

        function moveCamel() {
            const head = {x: camel[0].x + dx, y: camel[0].y + dy};
            camel.unshift(head);

            const hasEatenFood = camel[0].x === food.x && camel[0].y === food.y;
            if (hasEatenFood) {
                score += 1;
                document.getElementById("score").innerText = score;
                generateFood();
            } else {
                camel.pop();
            }
        }

        function drawFood() {
            // Drawing a Palm Branch / Leaf
            ctx.fillStyle = "#2e7d32"; // Deep green
            
            let cx = food.x * gridSize + gridSize / 2;
            let cy = food.y * gridSize + gridSize / 2;

            // Draw a stylized leaf shape
            ctx.beginPath();
            ctx.ellipse(cx, cy, 8, 4, Math.PI / 4, 0, 2 * Math.PI);
            ctx.fill();
            
            // Stem
            ctx.strokeStyle = "#1b5e20";
            ctx.lineWidth = 2;
            ctx.beginPath();
            ctx.moveTo(cx - 6, cy + 6);
            ctx.lineTo(cx + 6, cy - 6);
            ctx.stroke();
        }

        function generateFood() {
            food.x = Math.floor(Math.random() * tileCount);
            food.y = Math.floor(Math.random() * tileCount);

            // Ensure food doesn't spawn on top of the camel
            camel.forEach(part => {
                const hasEaten = part.x === food.x && part.y === food.y;
                if (hasEaten) generateFood();
            });
        }

        function hasGameEnded() {
            for (let i = 4; i < camel.length; i++) {
                if (camel[i].x === camel[0].x && camel[i].y === camel[0].y) return true;
            }

            const hitLeftWall = camel[0].x < 0;
            const hitRightWall = camel[0].x >= tileCount;
            const hitToptWall = camel[0].y < 0;
            const hitBottomWall = camel[0].y >= tileCount;

            return hitLeftWall || hitRightWall || hitToptWall || hitBottomWall;
        }

        function resetGame() {
            camel = [
                {x: 10, y: 10},
                {x: 9, y: 10},
                {x: 8, y: 10}
            ];
            dx = 1;
            dy = 0;
            score = 0;
            document.getElementById("score").innerText = score;
            generateFood();
        }

        // Handle keyboard inputs
        document.addEventListener("keydown", changeDirection);

        function changeDirection(event) {
            const keyPressed = event.keyCode;
            const LEFT_KEY = 37;
            const UP_KEY = 38;
            const RIGHT_KEY = 39;
            const DOWN_KEY = 40;
            
            const A_KEY = 65;
            const W_KEY = 87;
            const D_KEY = 68;
            const S_KEY = 83;

            const goingUp = dy === -1;
            const goingDown = dy === 1;
            const goingRight = dx === 1;
            const goingLeft = dx === -1;

            if ((keyPressed === LEFT_KEY || keyPressed === A_KEY) && !goingRight) {
                dx = -1;
                dy = 0;
            }
            if ((keyPressed === UP_KEY || keyPressed
