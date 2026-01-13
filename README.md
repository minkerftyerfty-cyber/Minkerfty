<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>لعبة ماريو</title>
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        body {
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
        }
    </style>
</head>
<body>
    <div id="root"></div>
    
    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        const MarioGame = () => {
            const canvasRef = useRef(null);
            const [gameStarted, setGameStarted] = useState(false);
            const [score, setScore] = useState(0);
            const [gameOver, setGameOver] = useState(false);
            
            const gameState = useRef({
                mario: { x: 50, y: 300, width: 30, height: 40, velocityY: 0, jumping: false },
                obstacles: [],
                coins: [],
                worms: [],
                flies: [],
                ground: 400,
                gravity: 0.6,
                jumpPower: -12,
                speed: 3,
                frame: 0
            });

            useEffect(() => {
                if (!gameStarted || gameOver) return;

                const canvas = canvasRef.current;
                const ctx = canvas.getContext('2d');
                let animationId;

                const handleKeyPress = (e) => {
                    const { mario, jumpPower } = gameState.current;
                    if ((e.code === 'Space' || e.code === 'ArrowUp') && !mario.jumping) {
                        mario.velocityY = jumpPower;
                        mario.jumping = true;
                    }
                };

                const handleTouch = (e) => {
                    e.preventDefault();
                    const { mario, jumpPower } = gameState.current;
                    if (!mario.jumping) {
                        mario.velocityY = jumpPower;
                        mario.jumping = true;
                    }
                };

                window.addEventListener('keydown', handleKeyPress);
                canvas.addEventListener('click', handleTouch);
                canvas.addEventListener('touchstart', handleTouch);

                const gameLoop = () => {
                    const state = gameState.current;
                    state.frame++;

                    ctx.fillStyle = '#87CEEB';
                    ctx.fillRect(0, 0, canvas.width, canvas.height);

                    ctx.fillStyle = '#8B4513';
                    ctx.fillRect(0, state.ground, canvas.width, canvas.height - state.ground);
                    
                    ctx.fillStyle = '#228B22';
                    ctx.fillRect(0, state.ground, canvas.width, 10);

                    state.mario.velocityY += state.gravity;
                    state.mario.y += state.mario.velocityY;

                    if (state.mario.y >= state.ground - state.mario.height) {
                        state.mario.y = state.ground - state.mario.height;
                        state.mario.velocityY = 0;
                        state.mario.jumping = false;
                    }

                    ctx.fillStyle = '#FF0000';
                    ctx.fillRect(state.mario.x, state.mario.y, state.mario.width, state.mario.height);
                    
                    ctx.fillStyle = '#FF0000';
                    ctx.fillRect(state.mario.x - 5, state.mario.y, state.mario.width + 10, 10);
                    
                    ctx.fillStyle = '#FFD700';
                    ctx.fillRect(state.mario.x + 5, state.mario.y + 10, state.mario.width - 10, 15);
                    
                    ctx.fillStyle = '#000';
                    ctx.fillRect(state.mario.x + 8, state.mario.y + 15, 4, 4);
                    ctx.fillRect(state.mario.x + 18, state.mario.y + 15, 4, 4);

                    if (state.frame % 120 === 0) {
                        state.obstacles.push({
                            x: canvas.width,
                            y: state.ground - 40,
                            width: 40,
                            height: 40
                        });
                    }

                    if (state.frame % 80 === 0) {
                        state.coins.push({
                            x: canvas.width,
                            y: state.ground - 120 - Math.random() * 80,
                            width: 20,
                            height: 20,
                            collected: false
                        });
                    }

                    if (state.frame % 150 === 0) {
                        state.worms.push({
                            x: canvas.width,
                            y: state.ground - 20,
                            width: 30,
                            height: 15,
                            segments: 5,
                            wiggle: 0
                        });
                    }

                    if (state.frame % 100 === 0) {
                        state.flies.push({
                            x: canvas.width,
                            y: state.ground - 200 - Math.random() * 100,
                            width: 15,
                            height: 15,
                            speedY: (Math.random() - 0.5) * 2,
                            wingFlap: 0
                        });
                    }

                    state.coins = state.coins.filter(coin => {
                        coin.x -= state.speed;

                        if (!coin.collected) {
                            if (
                                state.mario.x < coin.x + coin.width &&
                                state.mario.x + state.mario.width > coin.x &&
                                state.mario.y < coin.y + coin.height &&
                                state.mario.y + state.mario.height > coin.y
                            ) {
                                coin.collected = true;
                                setScore(s => s + 10);
                                return false;
                            }

                            ctx.fillStyle = '#FFD700';
                            ctx.beginPath();
                            ctx.arc(coin.x + coin.width/2, coin.y + coin.height/2, coin.width/2, 0, Math.PI * 2);
                            ctx.fill();
                            
                            ctx.fillStyle = '#FFF';
                            ctx.beginPath();
                            ctx.arc(coin.x + coin.width/2 - 3, coin.y + coin.height/2 - 3, 3, 0, Math.PI * 2);
                            ctx.fill();
                        }

                        return coin.x > -coin.width && !coin.collected;
                    });

                    state.worms = state.worms.filter(worm => {
                        worm.x -= state.speed;
                        worm.wiggle += 0.2;

                        if (
                            state.mario.x < worm.x + worm.width &&
                            state.mario.x + state.mario.width > worm.x &&
                            state.mario.y < worm.y + worm.height &&
                            state.mario.y + state.mario.height > worm.y
                        ) {
                            if (state.mario.velocityY > 0 && state.mario.y < worm.y) {
                                state.mario.velocityY = -8;
                                setScore(s => s + 5);
                                return false;
                            } else {
                                setGameOver(true);
                                return false;
                            }
                        }

                        ctx.fillStyle = '#8B4513';
                        for (let i = 0; i < worm.segments; i++) {
                            const segX = worm.x + i * 6;
                            const segY = worm.y + Math.sin(worm.wiggle + i * 0.5) * 3;
                            ctx.beginPath();
                            ctx.arc(segX, segY, 4, 0, Math.PI * 2);
                            ctx.fill();
                        }
                        
                        ctx.fillStyle = '#000';
                        ctx.beginPath();
                        ctx.arc(worm.x + 2, worm.y - 2, 1.5, 0, Math.PI * 2);
                        ctx.arc(worm.x + 6, worm.y - 2, 1.5, 0, Math.PI * 2);
                        ctx.fill();

                        return worm.x > -worm.width;
                    });

                    state.flies = state.flies.filter(fly => {
                        fly.x -= state.speed;
                        fly.y += fly.speedY;
                        fly.wingFlap += 0.3;

                        if (fly.y < 50) {
                            fly.y = 50;
                            fly.speedY *= -1;
                        }
                        if (fly.y > state.ground - 50) {
                            fly.y = state.ground - 50;
                            fly.speedY *= -1;
                        }

                        if (
                            state.mario.x < fly.x + fly.width &&
                            state.mario.x + state.mario.width > fly.x &&
                            state.mario.y < fly.y + fly.height &&
                            state.mario.y + state.mario.height > fly.y
                        ) {
                            setGameOver(true);
                            return false;
                        }

                        ctx.fillStyle = '#000';
                        ctx.beginPath();
                        ctx.ellipse(fly.x + fly.width/2, fly.y + fly.height/2, 6, 4, 0, 0, Math.PI * 2);
                        ctx.fill();

                        const wingOffset = Math.sin(fly.wingFlap) * 2;
                        ctx.fillStyle = 'rgba(200, 200, 255, 0.6)';
                        
                        ctx.beginPath();
                        ctx.ellipse(fly.x + 2, fly.y + 4 + wingOffset, 5, 8, -0.3, 0, Math.PI * 2);
                        ctx.fill();
                        
                        ctx.beginPath();
                        ctx.ellipse(fly.x + fly.width - 2, fly.y + 4 - wingOffset, 5, 8, 0.3, 0, Math.PI * 2);
                        ctx.fill();

                        ctx.fillStyle = '#FF0000';
                        ctx.beginPath();
                        ctx.arc(fly.x + 4, fly.y + 5, 1.5, 0, Math.PI * 2);
                        ctx.arc(fly.x + fly.width - 4, fly.y + 5, 1.5, 0, Math.PI * 2);
                        ctx.fill();

                        return fly.x > -fly.width;
                    });

                    state.obstacles = state.obstacles.filter(obs => {
                        obs.x -= state.speed;

                        if (
                            state.mario.x < obs.x + obs.width &&
                            state.mario.x + state.mario.width > obs.x &&
                            state.mario.y < obs.y + obs.height &&
                            state.mario.y + state.mario.height > obs.y
                        ) {
                            setGameOver(true);
                            return false;
                        }

                        ctx.fillStyle = '#2E8B57';
                        ctx.fillRect(obs.x, obs.y, obs.width, obs.height);
                        
                        ctx.fillStyle = '#228B22';
                        ctx.fillRect(obs.x, obs.y, 5, obs.height);
                        ctx.fillRect(obs.x + obs.width - 5, obs.y, 5, obs.height);
                        
                        ctx.fillStyle = '#2E8B57';
                        ctx.fillRect(obs.x - 5, obs.y - 10, obs.width + 10, 10);

                        return obs.x > -obs.width;
                    });

                    ctx.fillStyle = '#000';
                    ctx.font = 'bold 20px Arial';
                    ctx.fillText(`النقاط: ${score}`, 20, 30);

                    if (!gameOver) {
                        animationId = requestAnimationFrame(gameLoop);
                    }
                };

                gameLoop();

                return () => {
                    window.removeEventListener('keydown', handleKeyPress);
                    canvas.removeEventListener('click', handleTouch);
                    canvas.removeEventListener('touchstart', handleTouch);
                    if (animationId) cancelAnimationFrame(animationId);
                };
            }, [gameStarted, gameOver, score]);

            const startGame = () => {
                setGameStarted(true);
                setGameOver(false);
                setScore(0);
                gameState.current = {
                    mario: { x: 50, y: 300, width: 30, height: 40, velocityY: 0, jumping: false },
                    obstacles: [],
                    coins: [],
                    worms: [],
                    flies: [],
                    ground: 400,
                    gravity: 0.6,
                    jumpPower: -12,
                    speed: 3,
                    frame: 0
                };
            };

            return (
                <div className="flex flex-col items-center justify-center min-h-screen bg-gradient-to-b from-blue-400 to-blue-600 p-4">
                    <div className="bg-white rounded-lg shadow-2xl p-6 max-w-3xl w-full">
                        <h1 className="text-4xl font-bold text-center mb-4 text-red-600" style={{fontFamily: 'Arial'}}>
                            🎮 لعبة ماريو
                        </h1>
                        
                        {!gameStarted ? (
                            <div className="text-center">
                                <p className="text-xl mb-4 text-gray-700">اقفز فوق الديدان لقتلها! تجنب الذباب والأنابيب واجمع العملات!</p>
                                <button
                                    onClick={startGame}
                                    className="bg-green-500 hover:bg-green-600 text-white font-bold py-3 px-8 rounded-lg text-xl transition-all transform hover:scale-105"
                                >
                                    ابدأ اللعبة
                                </button>
                            </div>
                        ) : gameOver ? (
                            <div className="text-center">
                                <h2 className="text-3xl font-bold mb-4 text-red-600">انتهت اللعبة!</h2>
                                <p className="text-2xl mb-4 text-gray-700">النقاط النهائية: {score}</p>
                                <button
                                    onClick={startGame}
                                    className="bg-blue-500 hover:bg-blue-600 text-white font-bold py-3 px-8 rounded-lg text-xl transition-all transform hover:scale-105"
                                >
                                    العب مرة أخرى
                                </button>
                            </div>
                        ) : (
                            <>
                                <canvas
                                    ref={canvasRef}
                                    width={800}
                                    height={450}
                                    className="border-4 border-gray-800 rounded-lg w-full"
                                    style={{maxWidth: '800px', display: 'block', margin: '0 auto', touchAction: 'none'}}
                                />
                                <div className="mt-4 text-center">
                                    <p className="text-lg text-gray-700">
                                        <strong>التحكم:</strong> اضغط مسافة أو سهم للأعلى أو اضغط على الشاشة للقفز
                                    </p>
                                </div>
                            </>
                        )}
                    </div>
                </div>
            );
        };

        ReactDOM.render(<MarioGame />, document.getElementById('root'));
    </script>
</body>
</html>
