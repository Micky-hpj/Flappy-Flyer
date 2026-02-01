<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Flappy Flyer - COMPLETE</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            height: 100vh;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background: linear-gradient(135deg, #87CEEB, #98D8C8);
            font-family: Arial, sans-serif;
            overflow: hidden;
            touch-action: manipulation;
        }
        canvas {
            border: 3px solid #333;
            border-radius: 10px;
            box-shadow: 0 8px 25px rgba(0,0,0,0.3);
            max-width: 95vw;
            max-height: 95vh;
            touch-action: none;
        }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        
        let scale = 1;
        function resizeCanvas() {
            const maxW = window.innerWidth * 0.95;
            const maxH = window.innerHeight * 0.85;
            scale = Math.min(maxW / 400, maxH / 600);
            canvas.width = 400 * scale;
            canvas.height = 600 * scale;
            canvas.style.width = `${canvas.width}px`;
            canvas.style.height = `${canvas.height}px`;
            return scale;
        }
        
        scale = resizeCanvas();
        const groundY = canvas.height - 120;

        // ULTRA EASY SETTINGS - 50% EASIER THAN ORIGINAL
        const PIPE_SPEED = 2.2 * scale;
        const PIPE_GAP = 240 * scale;        // 50% WIDER than original
        const PIPE_WIDTH = 60 * scale;
        const BIRD_GRAVITY = 0.35 * scale;   // Gentler gravity
        const BIRD_JUMP = -10 * scale;

        // Game state
        let gameState = 'start';
        let score = 0;
        let bestScore = parseInt(localStorage.getItem('flappyFlyerBestScore')) || 0;

        const bird = {
            x: 100 * scale,
            y: 250 * scale,
            r: 20 * scale,
            vy: 0,
            rot: 0
        };

        let pipes = [];

        function jump() {
            if (gameState === 'playing') {
                bird.vy = BIRD_JUMP;
            } else if (gameState === 'start' || gameState === 'gameOver') {
                reset();
            }
        }

        canvas.addEventListener('click', jump);
        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                e.preventDefault();
                jump();
            }
        });

        function reset() {
            gameState = 'playing';
            bird.y = 250 * scale;
            bird.vy = 0;
            bird.rot = 0;
            pipes = [];
            score = 0;
        }

        function addPipe() {
            const minHeight = 120 * scale;
            const maxHeight = groundY - PIPE_GAP - 80 * scale;
            const topHeight = Math.random() * (maxHeight - minHeight) + minHeight;
            
            pipes.push({
                x: canvas.width,
                top: topHeight,
                bottom: topHeight + PIPE_GAP,
                passed: false
            });
        }

        function update() {
            if (gameState !== 'playing') return;

            bird.vy += BIRD_GRAVITY;
            bird.y += bird.vy * 0.95;
            bird.rot = Math.max(-0.3, Math.min(bird.vy * 0.05, 0.3));

            if (bird.y + bird.r > groundY - 15 * scale) {
                gameOver();
                return;
            }

            if (bird.y - bird.r < 0) {
                bird.y = bird.r;
                bird.vy = 0;
            }

            if (pipes.length === 0 || pipes[pipes.length-1].x < canvas.width - 380 * scale) {
                addPipe();
            }

            for (let i = pipes.length - 1; i >= 0; i--) {
                const pipe = pipes[i];
                pipe.x -= PIPE_SPEED;

                if (!pipe.passed && pipe.x + PIPE_WIDTH < bird.x) {
                    pipe.passed = true;
                    score++;
                }

                if (pipe.x + PIPE_WIDTH < 0) {
                    pipes.splice(i, 1);
                    continue;
                }

                const birdBox = {
                    left: bird.x - bird.r * 0.75,
                    right: bird.x + bird.r * 0.75,
                    top: bird.y - bird.r * 0.75,
                    bottom: bird.y + bird.r * 0.75
                };

                const pipeTop = { left: pipe.x, right: pipe.x + PIPE_WIDTH, top: 0, bottom: pipe.top };
                const pipeBottom = { left: pipe.x, right: pipe.x + PIPE_WIDTH, top: pipe.bottom, bottom: canvas.height };

                if (checkCollision(birdBox, pipeTop) || checkCollision(birdBox, pipeBottom)) {
                    gameOver();
                    return;
                }
            }
        }

        function checkCollision(r1, r2) {
            return !(r1.right < r2.left || r1.left > r2.right || 
                     r1.bottom < r2.top || r1.top > r2.bottom);
        }

        function gameOver() {
            if (score > bestScore) {
                bestScore = score;
                localStorage.setItem('flappyFlyerBestScore', bestScore);
            }
            gameState = 'gameOver';
        }

        function getMedal(score) {
            if (score >= 40) return '🏆';
            if (score >= 30) return '🥇';
            if (score >= 20) return '🥈';
            if (score >= 10) return '🥉';
            return '';
        }

        function drawBird() {
            ctx.save();
            ctx.translate(bird.x, bird.y);
            ctx.rotate(bird.rot);
            
            const gradient = ctx.createRadialGradient(0, 0, 0, 0, 0, bird.r);
            gradient.addColorStop(0, '#FFFEC1');
            gradient.addColorStop(1, '#FFD700');
            ctx.fillStyle = gradient;
            ctx.shadowColor = '#FFD700';
            ctx.shadowBlur = 10 * scale;
            ctx.beginPath();
            ctx.arc(0, 0, bird.r, 0, Math.PI * 2);
            ctx.fill();

            ctx.shadowBlur = 6 * scale;
            ctx.shadowColor = '#FFA500';
            ctx.fillStyle = '#FFA500';
            ctx.beginPath();
            ctx.ellipse(-8 * scale, 0, 14 * scale, 10 * scale, 0, 0, Math.PI * 2);
            ctx.fill();

            ctx.shadowBlur = 0;
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.arc(10 * scale, -5 * scale, 6 * scale, 0, Math.PI * 2);
            ctx.fill();
            ctx.fillStyle = 'black';
            ctx.beginPath();
            ctx.arc(12 * scale, -4 * scale, 3 * scale, 0, Math.PI * 2);
            ctx.fill();

            ctx.shadowColor = '#FF8C00';
            ctx.shadowBlur = 5 * scale;
            ctx.fillStyle = '#FF8C00';
            ctx.beginPath();
            ctx.moveTo(18 * scale, 0);
            ctx.lineTo(28 * scale, -3 * scale);
            ctx.lineTo(28 * scale, 3 * scale);
            ctx.closePath();
            ctx.fill();

            ctx.restore();
            ctx.shadowBlur = 0;
        }

        function drawPipes() {
            ctx.shadowColor = '#228B22';
            ctx.shadowBlur = 8 * scale;
            
            pipes.forEach(pipe => {
                ctx.fillStyle = '#228B22';
                ctx.fillRect(pipe.x, 0, PIPE_WIDTH, pipe.top);
                ctx.fillRect(pipe.x, pipe.bottom, PIPE_WIDTH, canvas.height - pipe.bottom);
                
                ctx.fillStyle = '#32CD32';
                ctx.fillRect(pipe.x - 5 * scale, pipe.top - 20 * scale, PIPE_WIDTH + 10 * scale, 20 * scale);
                ctx.fillRect(pipe.x - 5 * scale, pipe.bottom, PIPE_WIDTH + 10 * scale, 20 * scale);
            });
            
            ctx.shadowBlur = 0;
        }

        function drawScore(x, y, score, isBest = false) {
            ctx.save();
            ctx.shadowColor = isBest ? '#FFD700' : 'rgba(0,0,0,0.5)';
            ctx.shadowBlur = isBest ? 20 * scale : 12 * scale;
            ctx.fillStyle = isBest ? '#FFD700' : 'rgba(255,255,255,0.95)';
            ctx.font = `bold ${36 * scale}px Arial`;
            ctx.textAlign = 'center';
            ctx.fillText(score.toString(), x, y);
            ctx.restore();
        }

        function draw() {
            // Sky
            const skyGradient = ctx.createLinearGradient(0, 0, 0, canvas.height);
            skyGradient.addColorStop(0, '#87CEEB');
            skyGradient.addColorStop(0.6, '#98D8E8');
            skyGradient.addColorStop(1, '#90EE90');
            ctx.fillStyle = skyGradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Clouds
            ctx.fillStyle = 'rgba(255,255,255,0.8)';
            const time = Date.now() * 0.002;
            for (let i = 0; i < 3; i++) {
                const x = ((time + i * 2) * 30 * scale) % canvas.width;
                ctx.beginPath();
                ctx.arc(x, 60 * scale + i * 25 * scale, 18 * scale, 0, Math.PI * 2);
                ctx.arc(x + 22 * scale, 60 * scale + i * 25 * scale, 22 * scale, 0, Math.PI * 2);
                ctx.arc(x + 44 * scale, 60 * scale + i * 25 * scale, 18 * scale, 0, Math.PI * 2);
                ctx.fill();
            }

            if (gameState === 'playing') {
                // Ground
                ctx.fillStyle = '#90EE90';
                ctx.fillRect(0, groundY, canvas.width, canvas.height - groundY);
                ctx.fillStyle = '#7CB342';
                for (let x = 0; x < canvas.width; x += 35 * scale) {
                    ctx.fillRect(x, groundY, 30 * scale, 18 * scale);
                }

                drawPipes();
                drawBird();

                // CURRENT SCORE (TOP CENTER)
                drawScore(canvas.width * 0.5, 55 * scale, score);
                
                // BEST SCORE (TOP RIGHT - SMALLER)
                drawScore(canvas.width * 0.85, 45 * scale, bestScore, true);
            }

            // Start/Game Over screens - CHANGED TO "FLAPPY FLYER"
            if (gameState === 'start' || gameState === 'gameOver') {
                ctx.fillStyle = 'rgba(0,0,0,0.75)';
                ctx.fillRect(0, 0, canvas.width, canvas.height);
                
                ctx.fillStyle = 'white';
                ctx.textAlign = 'center';
                ctx.font = `bold ${52 * scale}px Arial`;
                ctx.shadowBlur = 25 * scale;
                ctx.shadowColor = 'rgba(0,0,0,0.8)';
                
                if (gameState === 'start') {
                    // CHANGED: "FLAPPY FLYER" instead of "FLAPPY BIRD"
                    ctx.fillText('FLAPPY FLYER', canvas.width/2, canvas.height/2 - 40 * scale);
                    ctx.font = `bold ${32 * scale}px Arial`;
                    ctx.shadowBlur = 15 * scale;
                    ctx.fillText('Click or SPACE to Play', canvas.width/2, canvas.height/2 + 10 * scale);
                    ctx.shadowBlur = 0;
                    
                    ctx.fillStyle = '#FFD700';
                    ctx.font = `${28 * scale}px Arial`;
                    ctx.fillText(`🏆 BEST: ${bestScore}`, canvas.width/2, canvas.height/2 + 60 * scale);
                } else {
                    ctx.fillText('GAME OVER', canvas.width/2, canvas.height/2 - 20 * scale);
                    
                    // Current score BIG
                    ctx.fillStyle = 'white';
                    ctx.font = `bold ${44 * scale}px Arial`;
                    ctx.shadowBlur = 20 * scale;
                    ctx.shadowColor = 'rgba(0,0,0,0.8)';
                    ctx.fillText(`Score: ${score}`, canvas.width/2, canvas.height/2 + 15 * scale);
                    
                    // Medal
                    const medal = getMedal(score);
                    if (medal) {
                        ctx.font = `${60 * scale}px Arial`;
                        ctx.fillText(medal, canvas.width/2, canvas.height/2 + 75 * scale);
                    }
                    
                    // Best score comparison
                    ctx.shadowBlur = 0;
                    ctx.font = `${32 * scale}px Arial`;
                    if (score > bestScore) {
                        ctx.fillStyle = '#FFD700';
                        ctx.fillText('🎉 NEW RECORD! 🎉', canvas.width/2, canvas.height/2 + 85 * scale);
                        ctx.font = `${28 * scale}px Arial`;
                        ctx.fillText(`Previous Best: ${bestScore}`, canvas.width/2, canvas.height/2 + 115 * scale);
                    } else {
                        ctx.fillStyle = '#FFD700';
                        ctx.font = `${32 * scale}px Arial`;
                        ctx.fillText(`Best: ${bestScore}`, canvas.width/2, canvas.height/2 + 75 * scale);
                    }
                    
                    ctx.fillStyle = 'white';
                    ctx.font = `bold ${28 * scale}px Arial`;
                    ctx.shadowBlur = 10 * scale;
                    ctx.shadowColor = 'rgba(0,0,0,0.5)';
                    ctx.fillText('Click or SPACE to Restart', canvas.width/2, canvas.height/2 + 155 * scale);
                }
            }

            ctx.textAlign = 'left';
            ctx.shadowBlur = 0;
        }

        function gameLoop() {
            update();
            draw();
            requestAnimationFrame(gameLoop);
        }

        window.addEventListener('resize', () => {
            scale = resizeCanvas();
        });

        gameLoop();
    </script>
</body>
</html>
