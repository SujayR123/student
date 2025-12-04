---
layout: opencs
title: Snake Game
permalink: /snake/
---

<style>

/* Default body + wrapper */
body {}
.wrap {
    margin-left: auto;
    margin-right: auto;
}

/* Canvas */
canvas {
    display: none;
    border-style: solid;
    border-width: 10px;
    border-color: #FFFFFF;
}
canvas:focus {
    outline: none;
}

/* Screen styles */
#gameover p, #setting p, #menu p {
    font-size: 20px;
}
#gameover a, #setting a, #menu a {
    font-size: 30px;
    display: block;
}
#gameover a:hover, #setting a:hover, #menu a:hover {
    cursor: pointer;
}
#gameover a:hover::before, #setting a:hover::before, #menu a:hover::before {
    content: ">";
    margin-right: 10px;
}
#menu { display: block; }
#gameover { display: none; }
#setting { display: none; }

#setting input {
    display: none;
}
#setting label {
    cursor: pointer;
    padding: 3px 6px;
    border-radius: 4px;
}
#setting input:checked + label {
    background-color: #FFF;
    color: #000;
}

/* ===================== */
/*        THEMES         */
/* ===================== */

.theme-classic {
    --bg: royalblue;
    --snake: #ffffff;
    --food: #ff4444;
    --border: #FFFFFF;
}

.theme-dark {
    --bg: #111111;
    --snake: #00ffcc;
    --food: #ff0066;
    --border: #444444;
}

.theme-neon {
    --bg: #000000;
    --snake: #39ff14;   /* neon green */
    --food: #ff0090;    /* neon pink */
    --border: #00eaff;  /* neon cyan */
}

</style>


<h2>Snake</h2>

<div class="container">
    <p class="fs-4">Score: <span id="score_value">0</span></p>

    <div class="container bg-secondary text-center">

        <!-- Main Menu -->
        <div id="menu" class="py-4 text-light">
            <p>Welcome to Snake, press <span style="background-color:#fff; color:#000">space</span> to begin</p>
            <a id="new_game" class="link-alert">new game</a>
            <a id="setting_menu" class="link-alert">settings</a>
        </div>

        <!-- Game Over -->
        <div id="gameover" class="py-4 text-light">
            <p>Game Over, press <span style="background-color:#fff; color:#000">space</span> to try again</p>
            <a id="new_game1" class="link-alert">new game</a>
            <a id="setting_menu1" class="link-alert">settings</a>
        </div>

        <!-- Play Screen -->
        <canvas id="snake" class="wrap" width="320" height="320" tabindex="1"></canvas>

        <!-- Settings Screen -->
        <div id="setting" class="py-4 text-light">
            <p>Settings Screen, press <span style="background-color:#fff; color:#000">space</span> to return</span></p>
            <a id="new_game2" class="link-alert">new game</a>

            <br>

            <p>
                Speed:
                <input id="speed1" type="radio" name="speed" value="140" checked/>
                <label for="speed1">Slow</label>

                <input id="speed2" type="radio" name="speed" value="90"/>
                <label for="speed2">Normal</label>

                <input id="speed3" type="radio" name="speed" value="50"/>
                <label for="speed3">Fast</label>
            </p>

            <p>
                Wall:
                <input id="wallon" type="radio" name="wall" value="1" checked/>
                <label for="wallon">On</label>

                <input id="walloff" type="radio" name="wall" value="0"/>
                <label for="walloff">Off</label>
            </p>

            <p>
                Theme:
                <input id="theme1" type="radio" name="theme" value="classic" checked/>
                <label for="theme1">Classic</label>

                <input id="theme2" type="radio" name="theme" value="dark"/>
                <label for="theme2">Dark</label>

                <input id="theme3" type="radio" name="theme" value="neon"/>
                <label for="theme3">Neon</label>
            </p>

        </div>

    </div>
</div>


<script>

(function(){

/* -------------------------------
   Refined game engine (option B)
   - Interpolated (smooth) rendering
   - requestAnimationFrame timing
   - Improved input handling (no reverse, one change per tick)
   - Safe food placement
   - Themes preserved
   - Keeps your UI exactly
---------------------------------*/

/* DOM */
const canvas = document.getElementById("snake");
const ctx = canvas.getContext("2d");

const SCREEN_SNAKE = 0;
const SCREEN_MENU = -1;
const SCREEN_GAME_OVER = 1;
const SCREEN_SETTING = 2;

const screen_snake = canvas;
const screen_menu = document.getElementById("menu");
const screen_game_over = document.getElementById("gameover");
const screen_setting = document.getElementById("setting");

const ele_score = document.getElementById("score_value");

const speed_setting = document.getElementsByName("speed");
const wall_setting  = document.getElementsByName("wall");
const theme_setting = document.getElementsByName("theme");

const button_new_game  = document.getElementById("new_game");
const button_new_game1 = document.getElementById("new_game1");
const button_new_game2 = document.getElementById("new_game2");
const button_setting_menu  = document.getElementById("setting_menu");
const button_setting_menu1 = document.getElementById("setting_menu1");

/* Game constants & state */
const BLOCK = 10; // logical grid cell size in pixels (canvas internal pixels)
const CANVAS_W = canvas.width;
const CANVAS_H = canvas.height;
const GRID_W = Math.floor(CANVAS_W / BLOCK);
const GRID_H = Math.floor(CANVAS_H / BLOCK);

let SCREEN = SCREEN_MENU;
let snake = [];              // array of {x:int, y:int} logical positions (grid cells)
let prevSnake = [];          // snapshot before last tick for interpolation
let dir = 1;                 // current direction (0 up, 1 right, 2 down, 3 left)
let nextDir = 1;             // direction requested (applies on next tick)
let inputLocked = false;     // prevent multiple direction changes within a tick
let speedMs = 140;           // milliseconds per grid step (adjusted by settings)
let accumulator = 0;         // for tick timing
let lastTime = 0;
let food = {x:0, y:0};
let score = 0;
let wall = 1;                // 1 = wall on (collide), 0 = wrap
let running = false;
let highScoreKey = "snake_v2_highscore";
let bestScore = 0;

/* Default theme */
document.body.classList.add("theme-classic");
canvas.style.borderColor = getComputedStyle(document.body).getPropertyValue('--border');

/* Load best score */
try {
    const v = localStorage.getItem(highScoreKey);
    bestScore = v ? Number(v) : 0;
} catch(e){ bestScore = 0; }

/* Utility functions */
function showScreen(screen_opt){
    SCREEN = screen_opt;
    switch(screen_opt){
        case SCREEN_SNAKE:
            screen_snake.style.display = "block";
            screen_menu.style.display = "none";
            screen_setting.style.display = "none";
            screen_game_over.style.display = "none";
            break;
        case SCREEN_GAME_OVER:
            screen_snake.style.display = "block";
            screen_menu.style.display = "none";
            screen_setting.style.display = "none";
            screen_game_over.style.display = "block";
            break;
        case SCREEN_SETTING:
            screen_snake.style.display = "none";
            screen_menu.style.display = "none";
            screen_setting.style.display = "block";
            screen_game_over.style.display = "none";
            break;
        default:
            screen_snake.style.display = "none";
            screen_menu.style.display = "block";
            screen_setting.style.display = "none";
            screen_game_over.style.display = "none";
    }
}

/* Event wiring (UI) */
window.onload = function(){
    // Buttons
    button_new_game.onclick = () => newGame();
    button_new_game1.onclick = () => newGame();
    button_new_game2.onclick = () => newGame();
    button_setting_menu.onclick = () => showScreen(SCREEN_SETTING);
    button_setting_menu1.onclick = () => showScreen(SCREEN_SETTING);

    // Speed radio wiring
    for(let i = 0; i < speed_setting.length; i++){
        speed_setting[i].addEventListener("click", function(){
            for(let j = 0; j < speed_setting.length; j++){
                if(speed_setting[j].checked){
                    setSpeed(Number(speed_setting[j].value));
                }
            }
        });
    }

    // Wall radios
    for(let i = 0; i < wall_setting.length; i++){
        wall_setting[i].addEventListener("click", function(){
            for(let j = 0; j < wall_setting.length; j++){
                if(wall_setting[j].checked){
                    setWall(Number(wall_setting[j].value));
                }
            }
        });
    }

    // Theme radios
    for(let i = 0; i < theme_setting.length; i++){
        theme_setting[i].addEventListener("click", function(){
            for(let j = 0; j < theme_setting.length; j++){
                if(theme_setting[j].checked){
                    applyTheme(theme_setting[j].value);
                }
            }
        });
    }

    // Initialize defaults
    // Set initial speed from checked radio (ensures synced)
    for(let i = 0; i < speed_setting.length; i++){
        if(speed_setting[i].checked) setSpeed(Number(speed_setting[i].value));
    }
    for(let i = 0; i < wall_setting.length; i++){
        if(wall_setting[i].checked) setWall(Number(wall_setting[i].value));
    }
    for(let i = 0; i < theme_setting.length; i++){
        if(theme_setting[i].checked) applyTheme(theme_setting[i].value);
    }

    // Space to start/try again
    window.addEventListener("keydown", function(evt) {
        if(evt.code === "Space"){
            // space starts game if menu or gameover is showing
            if(SCREEN !== SCREEN_SNAKE){
                newGame();
            }
        }
    }, true);

    // Keyboard arrows + WASD
    window.addEventListener("keydown", function(evt){
        if(SCREEN !== SCREEN_SNAKE) return; // ignore unless playing
        // Prevent default for arrows to avoid page scroll
        const code = evt.code;
        if(["ArrowUp","ArrowRight","ArrowDown","ArrowLeft","KeyW","KeyA","KeyS","KeyD"].includes(code)){
            evt.preventDefault();
            handleDirectionInput(code);
        }
    }, true);

    // Focus canvas so it receives keyboard events quickly
    screen_snake.addEventListener('click', ()=> screen_snake.focus());
};

/* Direction input handling:
   - Accepts one direction change per tick (inputLocked prevents multiple changes)
   - Prevents direct reverse (e.g., if moving right, can't go left)
*/
function handleDirectionInput(code){
    // Map to direction numbers
    let requested;
    if(code === "ArrowUp" || code === "KeyW") requested = 0;
    else if(code === "ArrowRight" || code === "KeyD") requested = 1;
    else if(code === "ArrowDown" || code === "KeyS") requested = 2;
    else if(code === "ArrowLeft" || code === "KeyA") requested = 3;
    else return;

    // If trying to reverse direction immediately, ignore
    if(Math.abs(requested - dir) === 2) return;

    // Only accept one change until next tick applies it
    if(inputLocked) {
        // queue the most recent change (overwrite nextDir)
        nextDir = requested;
    } else {
        nextDir = requested;
        inputLocked = true;
    }
}

/* Apply theme (updates CSS class and canvas border) */
function applyTheme(theme){
    document.body.classList.remove("theme-classic","theme-dark","theme-neon");
    document.body.classList.add(`theme-${theme}`);
    let borderColor = getComputedStyle(document.body).getPropertyValue("--border");
    if(borderColor) canvas.style.borderColor = borderColor;
}

/* Setters */
function setSpeed(v){
    speedMs = v;
}
function setWall(v){
    wall = v;
}

/* Game logic functions */

/* Safe random food placement: chooses random positions until finds free cell.
   Uses a deterministic cap to avoid pathological infinite loops. */
function placeFoodSafe(){
    const freeCells = [];
    for(let x = 0; x < GRID_W; x++){
        for(let y = 0; y < GRID_H; y++){
            if(!snake.some(s => s.x === x && s.y === y)){
                freeCells.push({x,y});
            }
        }
    }
    if(freeCells.length === 0){
        // Snake filled whole board — win condition: no food
        food = null;
        return;
    }
    const idx = Math.floor(Math.random() * freeCells.length);
    food = freeCells[idx];
}

/* Start new game: initialize snake, score, direction */
function newGame(){
    showScreen(SCREEN_SNAKE);
    screen_snake.focus();

    // initialize snake in center, length 3
    const startX = Math.floor(GRID_W / 2) - 1;
    const startY = Math.floor(GRID_H / 2);
    snake = [
        {x: startX + 2, y: startY},
        {x: startX + 1, y: startY},
        {x: startX    , y: startY}
    ];
    prevSnake = JSON.parse(JSON.stringify(snake)); // snapshot for interpolation
    dir = 1;
    nextDir = 1;
    inputLocked = false;
    running = true;
    accumulator = 0;
    lastTime = performance.now();
    score = 0;
    ele_score.innerHTML = String(score);

    placeFoodSafe();
    // Start render loop if not already running
    if(!animationRunning) {
        animationRunning = true;
        requestAnimationFrame(loop);
    }
}

/* End game: show game over screen, update high score */
function endGame(){
    running = false;
    showScreen(SCREEN_GAME_OVER);
    // update high score
    try {
        const prev = Number(localStorage.getItem(highScoreKey) || 0);
        if(score > prev){
            localStorage.setItem(highScoreKey, String(score));
        }
    } catch(e){}
}

/* Tick: perform a single logical step (move snake) */
function tick(){
    // move direction (apply nextDir)
    if(Math.abs(nextDir - dir) !== 2){
        dir = nextDir;
    }
    // snapshot previous positions for interpolation rendering
    prevSnake = JSON.parse(JSON.stringify(snake));

    // compute new head
    const head = { x: snake[0].x, y: snake[0].y };
    if(dir === 0) head.y -= 1;
    else if(dir === 1) head.x += 1;
    else if(dir === 2) head.y += 1;
    else if(dir === 3) head.x -= 1;

    // wall vs wrap
    if(wall === 1){
        // collide with wall -> game over
        if(head.x < 0 || head.x >= GRID_W || head.y < 0 || head.y >= GRID_H){
            endGame();
            return;
        }
    } else {
        // wrap-around
        if(head.x < 0) head.x = GRID_W - 1;
        if(head.x >= GRID_W) head.x = 0;
        if(head.y < 0) head.y = GRID_H - 1;
        if(head.y >= GRID_H) head.y = 0;
    }

    // self collision
    if(snake.some((seg, idx) => idx > 0 && seg.x === head.x && seg.y === head.y)){
        endGame();
        return;
    }

    // add head
    snake.unshift(head);

    // eat?
    if(food && head.x === food.x && head.y === food.y){
        score += 1;
        ele_score.innerHTML = String(score);
        // place new food
        placeFoodSafe();
        // don't remove tail (snake grows)
    } else {
        // normal move: remove tail
        snake.pop();
    }

    // unlock input for next tick
    inputLocked = false;
}

/* ---------- Rendering with interpolation ----------

We keep prevSnake (array of positions before last tick) and snake (current positions).
We compute an interpolation factor t = accumulator / speedMs (0..1),
and render each segment's position as lerp(prevPos, currentPos, t).
If prevSnake length differs (e.g., when eating), we handle gracefully.
--------------------------------------------------*/

function lerp(a,b,t){ return a + (b - a) * t; }

function render(){
    // background from theme
    const bg = (getComputedStyle(document.body).getPropertyValue("--bg") || "royalblue").trim();
    ctx.fillStyle = bg;
    ctx.fillRect(0,0,CANVAS_W,CANVAS_H);

    // compute interpolation t
    const t = Math.min(1, accumulator / Math.max(1, speedMs));

    // draw snake segments interpolating between prevSnake and snake
    const snakeColor = (getComputedStyle(document.body).getPropertyValue("--snake") || "#FFFFFF").trim();
    ctx.fillStyle = snakeColor;

    // For rendering, iterate over max length of prevSnake and snake
    const maxLen = Math.max(prevSnake.length, snake.length);
    for(let i = 0; i < maxLen; i++){
        // get prev pos for segment i; if missing, use closest existing
        const prev = prevSnake[i] || prevSnake[prevSnake.length - 1] || snake[snake.length - 1];
        const curr = snake[i] || snake[snake.length - 1] || prev;

        const rx = lerp(prev.x, curr.x, t) * BLOCK;
        const ry = lerp(prev.y, curr.y, t) * BLOCK;

        // small padding to reflect block look
        ctx.fillRect(Math.round(rx), Math.round(ry), BLOCK, BLOCK);
    }

    // draw food (no interpolation)
    if(food){
        const foodColor = (getComputedStyle(document.body).getPropertyValue("--food") || "#ff4444").trim();
        ctx.fillStyle = foodColor;
        ctx.fillRect(food.x * BLOCK, food.y * BLOCK, BLOCK, BLOCK);
    }
}

/* ---------- Main loop (rAF with accumulator) ---------- */

let animationRunning = false;
function loop(ts){
    if(!lastTime) lastTime = ts;
    const delta = ts - lastTime;
    lastTime = ts;

    if(running){
        accumulator += delta;
        // perform logical ticks while enough time has accumulated
        while(accumulator >= speedMs){
            accumulator -= speedMs;
            tick();
        }
    } else {
        // if not running, keep accumulator capped for rendering
        accumulator = Math.min(accumulator + delta, speedMs);
    }

    // render interpolated state
    render();

    // continue
    requestAnimationFrame(loop);
}

/* Initialize: show menu */
showScreen(SCREEN_MENU);
screen_snake.style.display = "none"; // keep canvas hidden until playing (keeps UI identical)

/* Prevent accidental double-space triggers in input fields by using keydown on window
   (we already handle space in window.onload earlier for start). */

 /* Expose minimal API (optional) */
window.SnakeRefined = {
    newGame, endGame, applyTheme
};

})();
</script>
