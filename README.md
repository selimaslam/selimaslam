<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Improved Flappy Bird</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
            background: linear-gradient(to bottom, #87CEEB, #E0F7FA);
            overflow: hidden;
        }
        
        #game-container {
            position: relative;
            width: 400px;
            height: 600px;
            background: linear-gradient(to bottom, #87CEEB, #E0F7FA);
            overflow: hidden;
            border: 3px solid #333;
            border-radius: 10px;
        }
        
        #bird {
            position: absolute;
            width: 40px;
            height: 40px;
            background: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"><circle cx="50" cy="50" r="40" fill="%23FFD700"/><circle cx="70" cy="40" r="10" fill="%23000"/><circle cx="73" cy="37" r="3" fill="%23FFF"/><path d="M30,50 Q50,30 70,50" stroke="%23000" stroke-width="3" fill="none"/></svg>') no-repeat;
            background-size: contain;
            top: 300px;
            left: 100px;
            z-index: 10;
            transition: transform 0.1s;
        }
        
        .pipe {
            position: absolute;
            width: 60px;
            background: linear-gradient(to right, #4CAF50, #2E7D32);
            border: 2px solid #1B5E20;
            border-radius: 5px;
        }
        
        .pipe-top {
            top: 0;
            border-bottom-left-radius: 0;
            border-bottom-right-radius: 0;
        }
        
        .pipe-bottom {
            bottom: 0;
            border-top-left-radius: 0;
            border-top-right-radius: 0;
        }
        
        #score-display {
            position: absolute;
            top: 20px;
            left: 20px;
            font-size: 24px;
            font-weight: bold;
            color: #000;
            z-index: 20;
            background: rgba(255, 255, 255, 0.7);
            padding: 5px 10px;
            border-radius: 5px;
        }
        
        #start-screen {
            position: absolute;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.5);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 30;
            color: white;
        }
        
        #game-over {
            position: absolute;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.7);
            display: none;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 30;
            color: white;
        }
        
        button {
            padding: 10px 20px;
            font-size: 18px;
            background: #FFD700;
            color: #000;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            margin-top: 20px;
            font-weight: bold;
        }
        
        button:hover {
            background: #FFC107;
        }
    </style>
</head>
<body>
    <div id="game-container">
        <div id="bird"></div>
        <div id="score-display">Score: 0</div>
        
        <div id="start-screen">
            <h1>Flappy Bird</h1>
            <p>Press SPACE or CLICK to flap wings</p>
            <button id="start-button">Start Game</button>
        </div>
        
        <div id="game-over">
            <h1>Game Over!</h1>
            <p id="final-score">Score: 0</p>
            <button id="restart-button">Play Again</button>
        </div>
    </div>

    <script>
        // Game variables with improved physics
        let bird = {
            x: 100,
            y: 300,
            width: 40,
            height: 40,
            velocity: 0,
            gravity: 0.4,    // Reduced gravity for smoother fall
            jump: -8,       // Reduced jump power
            maxSpeed: 10    // Added maximum fall speed
        };
        
        let pipes = [];
        let score = 0;
        let gameRunning = false;
        let lastPipeTime = 0;
        let pipeInterval = 1500;
        
        // DOM elements
        const gameContainer = document.getElementById('game-container');
        const birdElement = document.getElementById('bird');
        const scoreDisplay = document.getElementById('score-display');
        const startScreen = document.getElementById('start-screen');
        const gameOverScreen = document.getElementById('game-over');
        const finalScore = document.getElementById('final-score');
        const startButton = document.getElementById('start-button');
        const restartButton = document.getElementById('restart-button');
        
        // Event listeners
        startButton.addEventListener('click', startGame);
        restartButton.addEventListener('click', startGame);
        
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space' && gameRunning) {
                flap();
                e.preventDefault();
            }
        });
        
        document.addEventListener('click', () => {
            if (gameRunning) {
                flap();
            }
        });
        
        // Improved flap function
        function flap() {
            // Reset velocity to ensure consistent jump height
            bird.velocity = bird.jump;
            
            // Add small visual feedback
            birdElement.style.transform = 'rotate(-20deg)';
            setTimeout(() => {
                birdElement.style.transform = 'rotate(0deg)';
            }, 200);
        }
        
        // Start game
        function startGame() {
            // Reset game state
            bird.y = 300;
            bird.velocity = 0;
            pipes.forEach(pipe => pipe.topElement.remove());
            pipes.forEach(pipe => pipe.bottomElement.remove());
            pipes = [];
            score = 0;
            
            // Update display
            scoreDisplay.textContent = `Score: ${score}`;
            
            // Hide screens
            startScreen.style.display = 'none';
            gameOverScreen.style.display = 'none';
            
            gameRunning = true;
            
            // Start game loop
            requestAnimationFrame(gameLoop);
        }
        
        // Create pipe
        function createPipe() {
            const now = Date.now();
            if (now - lastPipeTime > pipeInterval) {
                const topHeight = 100 + Math.random() * 200;
                const bottomHeight = 600 - topHeight - 150; // Fixed gap size
                
                const topPipe = document.createElement('div');
                topPipe.className = 'pipe pipe-top';
                topPipe.style.height = `${topHeight}px`;
                topPipe.style.left = '400px';
                gameContainer.appendChild(topPipe);
                
                const bottomPipe = document.createElement('div');
                bottomPipe.className = 'pipe pipe-bottom';
                bottomPipe.style.height = `${bottomHeight}px`;
                bottomPipe.style.left = '400px';
                bottomPipe.style.bottom = '0';
                gameContainer.appendChild(bottomPipe);
                
                pipes.push({
                    x: 400,
                    topHeight: topHeight,
                    bottomHeight: bottomHeight,
                    topElement: topPipe,
                    bottomElement: bottomPipe,
                    passed: false
                });
                
                lastPipeTime = now;
            }
        }
        
        // Update pipes
        function updatePipes() {
            pipes.forEach(pipe => {
                pipe.x -= 2; // Consistent pipe speed
                pipe.topElement.style.left = `${pipe.x}px`;
                pipe.bottomElement.style.left = `${pipe.x}px`;
                
                // Check if bird passed the pipe
                if (!pipe.passed && pipe.x < bird.x - 60) {
                    pipe.passed = true;
                    score++;
                    scoreDisplay.textContent = `Score: ${score}`;
                }
                
                // Remove pipes that are off screen
                if (pipe.x < -60) {
                    pipe.topElement.remove();
                    pipe.bottomElement.remove();
                    pipes = pipes.filter(p => p !== pipe);
                }
            });
        }
        
        // Check collisions
        function checkCollisions() {
            // Check if bird hits the ground or ceiling
            if (bird.y > 600 - bird.height || bird.y < 0) {
                gameOver();
                return;
            }
            
            // Check if bird hits any pipes
            for (const pipe of pipes) {
                if (
                    bird.x + bird.width > pipe.x &&
                    bird.x < pipe.x + 60 &&
                    (bird.y < pipe.topHeight || bird.y + bird.height > 600 - pipe.bottomHeight)
                ) {
                    gameOver();
                    return;
                }
            }
        }
        
        // Game over
        function gameOver() {
            gameRunning = false;
            finalScore.textContent = `Score: ${score}`;
            gameOverScreen.style.display = 'flex';
        }
        
        // Main game loop with improved physics
        function gameLoop() {
            if (!gameRunning) return;
            
            // Update bird position with improved physics
            bird.velocity += bird.gravity;
            
            // Limit maximum fall speed
            if (bird.velocity > bird.maxSpeed) {
                bird.velocity = bird.maxSpeed;
            }
            
            bird.y += bird.velocity;
            birdElement.style.top = `${bird.y}px`;
            
            // Rotate bird based on velocity (more subtle now)
            birdElement.style.transform = `rotate(${bird.velocity * 2}deg)`;
            
            // Create pipes
            createPipe();
            
            // Update pipes
            updatePipes();
            
            // Check collisions
            checkCollisions();
            
            requestAnimationFrame(gameLoop);
        }
    </script>
</body>
</html>
