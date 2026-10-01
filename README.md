<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no, maximum-scale=1.0">
<title>Obby Speed Run 3D: Race Simulator</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; }
  body, html { width: 100%; height: 100%; overflow: hidden; background: #38bdf8; font-family: 'Segoe UI', system-ui, sans-serif; }
  #canvas-container { width: 100%; height: 100%; position: absolute; left: 0; top: 0; z-index: 1; }

  /* Top HUD */
  #top-hud {
    position: absolute; top: 12px; left: 12px; right: 12px;
    display: flex; justify-content: space-between; align-items: center;
    z-index: 10; pointer-events: none;
  }
  .hud-box {
    background: rgba(15, 23, 42, 0.88); border: 2px solid rgba(255, 255, 255, 0.2);
    border-radius: 12px; padding: 6px 14px; color: #fff; pointer-events: auto;
    display: flex; align-items: center; gap: 10px; font-weight: 800; font-size: 13px;
    box-shadow: 0 4px 15px rgba(0,0,0,0.3);
  }
  .gold-badge { color: #facc15; }
  .speed-badge { color: #38bdf8; }

  /* Right Side Action Strip */
  #side-menu {
    position: absolute; right: 12px; top: 75px; display: flex; flex-direction: column; gap: 8px; z-index: 10;
  }
  .menu-btn {
    background: #3b82f6; border: 2px solid #fff; border-radius: 10px; color: #fff;
    padding: 8px 12px; font-weight: 800; font-size: 11px; cursor: pointer;
    box-shadow: 0 4px 0 #1d4ed8; transition: transform 0.08s ease; text-align: center;
  }
  .menu-btn:active { transform: translateY(3px); box-shadow: 0 1px 0 #1d4ed8; }
  .btn-ad { background: linear-gradient(135deg, #10b981, #059669); border-color: #a7f3d0; box-shadow: 0 4px 0 #047857; }
  .btn-egg { background: linear-gradient(135deg, #f59e0b, #d97706); border-color: #fde68a; box-shadow: 0 4px 0 #b45309; }
  .btn-rebirth { background: linear-gradient(135deg, #ec4899, #be185d); border-color: #fbcfe8; box-shadow: 0 4px 0 #9d174d; }

  /* Giant Click-to-Train Center Button */
  #train-zone {
    position: absolute; bottom: 85px; left: 50%; transform: translateX(-50%);
    z-index: 10; text-align: center; pointer-events: auto;
  }
  #btn-click {
    background: radial-gradient(circle, #facc15, #eab308); border: 4px solid #fff;
    color: #1e293b; font-size: 16px; font-weight: 900; padding: 14px 32px;
    border-radius: 40px; box-shadow: 0 8px 25px rgba(234, 179, 8, 0.6); cursor: pointer;
    transition: transform 0.05s ease;
  }
  #btn-click:active { transform: scale(0.92); }

  /* Bottom Controls */
  #bottom-bar {
    position: absolute; bottom: 12px; left: 16px; right: 16px;
    display: flex; justify-content: space-between; align-items: center; z-index: 10; pointer-events: none;
  }
  .race-btn {
    background: #10b981; color: #fff; border: 3px solid #fff; border-radius: 30px;
    font-size: 14px; font-weight: 900; padding: 10px 24px; cursor: pointer;
    box-shadow: 0 6px 16px rgba(16, 185, 129, 0.4); pointer-events: auto;
  }
  .auto-btn {
    background: #64748b; color: #fff; border: 2px solid #fff; border-radius: 20px;
    font-size: 11px; font-weight: 800; padding: 8px 14px; cursor: pointer; pointer-events: auto;
  }

  /* Modals */
  .modal {
    position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
    background: rgba(15, 23, 42, 0.96); border: 3px solid #facc15; border-radius: 18px;
    padding: 20px; color: #fff; width: 88%; max-width: 320px; z-index: 50; display: none;
    text-align: center; box-shadow: 0 10px 40px rgba(0,0,0,0.6);
  }
  .modal h3 { color: #facc15; font-size: 18px; margin-bottom: 12px; font-weight: 900; }
  .modal-close { background: #ef4444; margin-top: 15px; border-radius: 8px; border: none; color:#fff; font-weight: 800; padding: 6px 16px; cursor: pointer; }

  #banner {
    position: absolute; top: 80px; left: 50%; transform: translateX(-50%);
    background: #10b981; color: #fff; font-size: 18px; font-weight: 900;
    padding: 10px 24px; border-radius: 30px; border: 2px solid #fff; z-index: 30; display: none;
  }
</style>

<!-- CrazyGames SDK v3 -->
<script src="https://sdk.crazygames.com/crazygames-sdk-v3.js"></script>
<!-- Three.js -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
</head>
<body>

<div id="canvas-container"></div>

<div id="top-hud">
  <div class="hud-box">
    <span class="speed-badge">⚡ SPEED: <span id="spd-val">10</span></span>
    <span class="gold-badge">🪙 <span id="gold-val">0</span></span>
  </div>
  <div class="hud-box">
    <span>🏆 WINS: <span id="win-val">0</span></span>
  </div>
</div>

<div id="side-menu">
  <button class="menu-btn btn-ad" id="ad-btn">📺 2x SPEED (FREE)</button>
  <button class="menu-btn btn-egg" id="egg-btn">🥚 HATCH PET (50 🪙)</button>
  <button class="menu-btn btn-rebirth" id="rebirth-btn">⭐ REBIRTH</button>
</div>

<div id="train-zone">
  <button id="btn-click">⚡ TAP TO TRAIN (+1 SPEED)</button>
</div>

<div id="bottom-bar">
  <button class="auto-btn" id="auto-btn">AUTO-TAP: OFF</button>
  <button class="race-btn" id="race-btn">🏁 START RACE</button>
</div>

<div id="banner">SPEED SURGE!</div>

<!-- Pet Modal -->
<div class="modal" id="egg-modal">
  <h3>EGG SHOP</h3>
  <p style="font-size:12px; margin-bottom: 12px; color:#cbd5e1;">Hatch pets to multiply training speed permanently!</p>
  <button class="menu-btn btn-egg" id="buy-pet-btn" style="width:100%;">HATCH COMMON PET (50 🪙)</button>
  <button class="modal-close" onclick="document.getElementById('egg-modal').style.display='none'">CLOSE</button>
</div>

<script>
/* ==================== CRAZYGAMES REVENUE SDK ==================== */
let csdk = null;
async function initSDK() {
  try {
    if (window.CrazyGames && window.CrazyGames.SDK) {
      csdk = window.CrazyGames.SDK;
      await csdk.init();
      csdk.game.gameplayStart();
    }
  } catch(e) { console.log('SDK Offline / GitHub Pages Mode'); }
}
initSDK();

function playRewarded(onReward) {
  if (csdk && csdk.ad) {
    csdk.ad.requestAd('rewarded', {
      adStarted: () => {},
      adFinished: () => onReward(),
      adError: () => onReward()
    });
  } else {
    onReward(); // Grant reward instantly on GitHub Pages for testing
  }
}

function playMidgame(onDone) {
  if (csdk && csdk.ad) {
    csdk.ad.requestAd('midgame', {
      adStarted: () => {},
      adFinished: () => onDone(),
      adError: () => onDone()
    });
  } else {
    onDone();
  }
}

/* ==================== ECONOMY & STATS ==================== */
let speed = parseInt(localStorage.getItem('ob_spd') || '10');
let gold = parseInt(localStorage.getItem('ob_gold') || '0');
let wins = parseInt(localStorage.getItem('ob_wins') || '0');
let petMultiplier = parseFloat(localStorage.getItem('ob_pet') || '1.0');
let autoTap = false;
let isRacing = false;
let raceDist = 0;

function updateStats() {
  document.getElementById('spd-val').textContent = speed;
  document.getElementById('gold-val').textContent = gold;
  document.getElementById('win-val').textContent = wins;
  localStorage.setItem('ob_spd', speed);
  localStorage.setItem('ob_gold', gold);
  localStorage.setItem('ob_wins', wins);
  localStorage.setItem('ob_pet', petMultiplier);
}

// Tap to Train Clicker
document.getElementById('btn-click').onclick = () => {
  speed += Math.round(1 * petMultiplier);
  updateStats();
};

// Rewarded Ad (2x Speed Boost)
document.getElementById('ad-btn').onclick = () => {
  playRewarded(() => {
    speed *= 2;
    updateStats();
    showBanner("⚡ 2X SPEED ACTIVATED!");
  });
};

// Pet Hatching
document.getElementById('egg-btn').onclick = () => {
  document.getElementById('egg-modal').style.display = 'block';
};
document.getElementById('buy-pet-btn').onclick = () => {
  if (gold >= 50) {
    gold -= 50;
    petMultiplier += 0.5;
    updateStats();
    showBanner(`🥚 PET HATCHED! (+0.5x Multiplier)`);
    document.getElementById('egg-modal').style.display = 'none';
  } else {
    alert("Need 50 Gold! Win races to earn gold.");
  }
};

// Rebirth System
document.getElementById('rebirth-btn').onclick = () => {
  if (speed >= 500) {
    speed = 10;
    gold += 200;
    wins += 1;
    petMultiplier += 1.0;
    updateStats();
    showBanner("⭐ REBIRTH SUCCESS! +1x PERMANENT MULTIPLIER");
  } else {
    alert("Need 500 Speed to Rebirth!");
  }
};

// Auto Tap Feature
document.getElementById('auto-btn').onclick = () => {
  autoTap = !autoTap;
  document.getElementById('auto-btn').textContent = autoTap ? "AUTO-TAP: ON" : "AUTO-TAP: OFF";
  document.getElementById('auto-btn').style.background = autoTap ? "#10b981" : "#64748b";
};
setInterval(() => {
  if (autoTap && !isRacing) {
    speed += Math.round(1 * petMultiplier);
    updateStats();
  }
}, 300);

function showBanner(text) {
  const b = document.getElementById('banner');
  b.textContent = text;
  b.style.display = 'block';
  setTimeout(() => { b.style.display = 'none'; }, 1600);
}

/* ==================== 3D WORLD SETUP ==================== */
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x38bdf8);
scene.fog = new THREE.Fog(0x38bdf8, 30, 200);

const camera = new THREE.PerspectiveCamera(65, window.innerWidth / window.innerHeight, 0.1, 500);
const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio || 1, 2));
document.getElementById('canvas-container').appendChild(renderer.domElement);

scene.add(new THREE.AmbientLight(0xffffff, 0.8));
const sun = new THREE.DirectionalLight(0xffffff, 0.7);
sun.position.set(20, 40, 20);
scene.add(sun);

// Endless Race Track Runway
const trackGeo = new THREE.PlaneGeometry(16, 1000);
const trackMat = new THREE.MeshLambertMaterial({ color: 0x1e293b });
const track = new THREE.Mesh(trackGeo, trackMat);
track.rotation.x = -Math.PI / 2;
track.position.z = 450;
scene.add(track);

// Checkpoint gates with Gold rewards
const gates = [];
for (let i = 1; i <= 20; i++) {
  const gateZ = i * 45;
  const gMesh = new THREE.Mesh(
    new THREE.BoxGeometry(16, 6, 0.5),
    new THREE.MeshLambertMaterial({ color: i % 2 === 0 ? 0xef4444 : 0xf59e0b })
  );
  gMesh.position.set(0, 3, gateZ);
  scene.add(gMesh);
  gates.push({ mesh: gMesh, z: gateZ, passed: false, reward: i * 15 });
}

// Player Mesh (Speed Avatar)
const playerGroup = new THREE.Group();
const body = new THREE.Mesh(new THREE.BoxGeometry(1.2, 1.4, 0.8), new THREE.MeshLambertMaterial({ color: 0x2563eb }));
body.position.y = 1.2;
playerGroup.add(body);
const head = new THREE.Mesh(new THREE.BoxGeometry(0.8, 0.8, 0.8), new THREE.MeshLambertMaterial({ color: 0xfacc15 }));
head.position.y = 2.2;
playerGroup.add(head);

// Mini Pet Floating next to Player
const petMesh = new THREE.Mesh(new THREE.SphereGeometry(0.4, 16, 16), new THREE.MeshLambertMaterial({ color: 0xec4899 }));
petMesh.position.set(1.4, 2.0, 0);
playerGroup.add(petMesh);

scene.add(playerGroup);

/* ==================== RACE MECHANIC ==================== */
const raceBtn = document.getElementById('race-btn');
raceBtn.onclick = () => {
  if (!isRacing) {
    startRace();
  }
};

function startRace() {
  isRacing = true;
  raceDist = 0;
  playerGroup.position.set(0, 0, 0);
  raceBtn.textContent = "🏃 RACING...";
  raceBtn.style.background = "#f59e0b";
  gates.forEach(g => { g.passed = false; });
}

function finishRace() {
  isRacing = false;
  raceBtn.textContent = "🏁 START RACE";
  raceBtn.style.background = "#10b981";
  playerGroup.position.set(0, 0, 0);
  showBanner(`🏁 RACE FINISHED! EARNED GOLD!`);

  // Trigger Midgame Video Ad every 2 races
  wins++;
  updateStats();
  if (wins % 2 === 0) {
    playMidgame(() => {});
  }
}

/* ==================== LOOP ==================== */
let lastT = performance.now();
function animate() {
  requestAnimationFrame(animate);
  const now = performance.now();
  const dt = Math.min((now - lastT) / 1000, 0.08);
  lastT = now;

  if (isRacing) {
    const runVelocity = Math.max(12, speed * 0.45);
    raceDist += runVelocity * dt;
    playerGroup.position.z = raceDist;

    // Check gate collisions
    for (const g of gates) {
      if (!g.passed && raceDist >= g.z) {
        g.passed = true;
        gold += g.reward;
        updateStats();
      }
    }

    if (raceDist >= 900) {
      finishRace();
    }
  } else {
    // Idle bobbing
    playerGroup.position.y = Math.sin(now * 0.005) * 0.15;
  }

  // Camera follow
  camera.position.lerp(new THREE.Vector3(playerGroup.position.x, playerGroup.position.y + 4.5, playerGroup.position.z - 8), 0.1);
  camera.lookAt(playerGroup.position.x, playerGroup.position.y + 1.5, playerGroup.position.z + 5);

  renderer.render(scene, camera);
}

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});

updateStats();
animate();
</script>
</body>
</html>
