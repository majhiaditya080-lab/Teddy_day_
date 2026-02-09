<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<title>Teddy Catch for Janki</title>

<style>
html, body { margin: 0; padding: 0; background: #000; color: #fff; font-family: sans-serif; overflow: hidden; height: 100%; width: 100%; position: fixed; }

.screen { display: none; height: 100%; width: 100%; position: absolute; flex-direction: column; justify-content: center; align-items: center; text-align: center; padding: 20px; box-sizing: border-box; }
.active { display: flex; }

.glitch { font-size: 1.8rem; color: #ff4d6d; text-transform: uppercase; text-shadow: 2px 2px #00fffc; font-weight: bold; }

#game-box { width: 100%; max-width: 360px; height: 55vh; background: #111; border: 3px solid #333; position: relative; overflow: hidden; border-radius: 10px; }

#basket { position: absolute; bottom: 20px; width: 90px; height: 60px; background: #4a2c2c; border-radius: 10px; font-size: 35px; display: flex; justify-content: center; align-items: center; border: 2px solid #ff4d6d; z-index: 100; transition: background 0.1s; }

.falling { position: absolute; font-size: 35px; width: 40px; height: 40px; display: flex; justify-content: center; align-items: center; z-index: 50; }

.controls { width: 100%; max-width: 360px; display: flex; gap: 15px; margin-top: 20px; }
.btn { flex: 1; padding: 20px; background: #222; border: 2px solid #ff4d6d; color: #fff; font-size: 2rem; border-radius: 15px; touch-action: none; -webkit-user-select: none; }

.start-btn { padding: 18px 40px; background: #ff4d6d; border: none; color: #fff; font-size: 1.2rem; border-radius: 50px; font-weight: bold; cursor: pointer; }

/* Floating Points Effect */
.pts-pop { position: absolute; font-weight: bold; color: #00fffc; animation: fly 0.8s forwards; pointer-events: none; z-index: 200; }
@keyframes fly { from { transform: translateY(0); opacity: 1; } to { transform: translateY(-60px); opacity: 0; } }
</style>
</head>

<body>

<audio id="snd-click" src="https://assets.mixkit.co/active_storage/sfx/2568/2568-preview.mp3"></audio>
<audio id="snd-catch" src="https://assets.mixkit.co/active_storage/sfx/2003/2003-preview.mp3"></audio>
<audio id="snd-bomb" src="https://assets.mixkit.co/active_storage/sfx/2747/2747-preview.mp3"></audio>
<audio id="snd-win" src="https://assets.mixkit.co/active_storage/sfx/1435/1435-preview.mp3"></audio>

<section id="s1" class="screen active">
    <h1 id="intro-txt" class="glitch"></h1>
</section>

<section id="s2" class="screen">
    <h2 class="glitch">READY MY LADY?</h2>
    <button class="start-btn" onclick="play('snd-click'); show(3)">I AM READY</button>
</section>

<section id="s3" class="screen">
    <h3 style="color:#ff4d6d">MISSION BRIEF</h3>
    <p>Catch Teddies: 100–300 pts<br>Avoid Bombs: -500 pts<br>Goal: 2500 Points</p>
    <button class="start-btn" onclick="play('snd-click'); startGame()">START GAME</button>
</section>

<section id="s4" class="screen">
    <div id="score-val" style="color:#ff4d6d;font-size:1.8rem;font-weight:bold;margin-bottom:10px;">SCORE: 0</div>
    <div id="game-box">
        <div id="basket">🧺</div>
    </div>
    <div class="controls">
        <button class="btn" id="lBtn">⬅️</button>
        <button class="btn" id="rBtn">➡️</button>
    </div>
</section>

<section id="s5" class="screen">
    <div id="quote-area" class="glitch" style="font-size:1.3rem; padding: 20px;"></div>
</section>

<section id="s6" class="screen">
    <h1 class="glitch">COMING SOON</h1>
    <div id="timer" style="font-size:3.5rem;color:#ff4d6d;font-family:monospace;font-weight:bold;"></div>
    <p>Until Promise Day</p>
</section>

<script>
let score = 0, bX = 135, gameOn = false;
let spawnInt = null, moveInt = null;

const basket = document.getElementById("basket");
const box = document.getElementById("game-box");
const scoreVal = document.getElementById("score-val");
const quoteArea = document.getElementById("quote-area");
const timer = document.getElementById("timer");

let fallSpeed = 3.5; // Fixed speed for better control

function play(id){
    const a = document.getElementById(id);
    if(!a) return;
    a.currentTime = 0;
    a.play().catch(()=>{});
}

const days = ["7th Feb: Rose Day", "8th Feb: Propose Day", "9th Feb: Chocolate Day", "10th Feb: TEDDY DAY"];
let d = 0;
(function intro(){
    document.getElementById("intro-txt").innerText = days[d];
    d++;
    if(d < days.length) setTimeout(intro, 3000); // Shorter intro for testing
    else setTimeout(()=>show(2), 3000);
})();

function show(n){
    document.querySelectorAll(".screen").forEach(s=>s.classList.remove("active"));
    document.getElementById("s"+n).classList.add("active");
}

function move(dir){
    clearInterval(moveInt);
    moveInt = setInterval(()=>{
        const maxX = box.clientWidth - 90;
        bX += dir * 9;
        if(bX < 0) bX = 0;
        if(bX > maxX) bX = maxX;
        basket.style.left = bX + "px";
    }, 20);
}

document.getElementById('lBtn').onpointerdown = ()=>move(-1);
document.getElementById('rBtn').onpointerdown = ()=>move(1);
window.onpointerup = ()=>clearInterval(moveInt);

function startGame(){
    show(4);
    score = 0;
    gameOn = true;
    scoreVal.innerText = "SCORE: 0";
    bX = (box.clientWidth - 90) / 2;
    basket.style.left = bX + "px";
    spawnInt = setInterval(spawn, 1000);
    requestAnimationFrame(update);
}

function spawn(){
    if(!gameOn) return;
    const items = [
        {i:"✨",p:250},{i:"🐶",p:200},{i:"🧸",p:100},
        {i:"👩‍❤️‍👨",p:300},{i:"💣",p:-500}
    ];
    const data = items[Math.floor(Math.random()*items.length)];
    const f = document.createElement("div");
    f.className = "falling";
    f.innerHTML = data.i;
    f.dataset.p = data.p;
    f.style.left = Math.random()*(box.clientWidth-45)+"px";
    f.style.top = "-50px";
    box.appendChild(f);
}

function update(){
    if(!gameOn) return;
    document.querySelectorAll(".falling").forEach(f=>{
        let y = (parseFloat(f.style.top)||-50) + fallSpeed;
        f.style.top = y+"px";

        const fLeft = parseFloat(f.style.left);
        const fBottom = y + 40;
        const basketTop = box.clientHeight - 80;

        // ✅ IMPROVED HITBOX (More forgiving for Janki)
        if(fBottom >= basketTop && fBottom <= basketTop + 30){
            if(fLeft + 35 >= bX && fLeft <= bX + 85){
                const pts = parseInt(f.dataset.p);
                score = Math.max(0, score + pts);
                play(pts > 0 ? "snd-catch" : "snd-bomb");
                scoreVal.innerText = "SCORE: " + score;
                
                // Show floating text
                const pop = document.createElement('div');
                pop.className = 'pts-pop';
                pop.innerText = pts > 0 ? "+"+pts : pts;
                pop.style.left = f.style.left;
                pop.style.top = basketTop + "px";
                box.appendChild(pop);
                setTimeout(()=>pop.remove(), 800);

                basket.style.background = pts > 0 ? "#ff4d6d" : "#555";
                setTimeout(()=>basket.style.background="#4a2c2c",100);
                f.remove();
                if(score >= 2500) win();
            }
        }
        if(y > box.clientHeight) f.remove();
    });
    requestAnimationFrame(update);
}

function win(){
    gameOn = false;
    clearInterval(spawnInt);
    play("snd-win");
    show(5);
    const q = ["Pure Magic ❤️","Warmest Hugs!","Aditya Loves Janki","Always Yours","Teddy Day Joy","Love You Infinity","My Cuddle Queen","Softest Heart","Forever Together","Wait for it..."];
    let i = 0;
    const qi = setInterval(()=>{
        if(i < q.length) quoteArea.innerText = q[i++];
        else { clearInterval(qi); show(6); startFinal(); }
    }, 3000);
}

function startFinal(){
    const target = new Date("Feb 11, 2026 00:00:00").getTime();
    setInterval(()=>{
        let diff = target - Date.now();
        if(diff <= 0){ timer.innerText = "00:00:00"; return; }
        let h = Math.floor(diff/36e5);
        let m = Math.floor(diff%36e5/6e4);
        let s = Math.floor(diff%6e4/1000);
        timer.innerText = h+"h "+m+"m "+s+"s";
    }, 1000);
}
</script>
</body>
</html>
