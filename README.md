<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8" />
  <title>เข็นของให้ถึงบ้าน | Force Grid Game</title>
  <style>
    body { font-family: Arial, sans-serif; background:#f6f6f6; text-align:center; }
    h1 { margin-top:10px; }
    .game { display:flex; justify-content:center; gap:20px; margin-top:20px; }
    .grid { display:grid; grid-template-columns:repeat(6,60px); grid-template-rows:repeat(6,60px); gap:2px; }
    .cell { width:60px; height:60px; background:#fff; border:1px solid #333; display:flex; align-items:center; justify-content:center; font-size:26px; }
    .wall { background:#555; }
    .home { background:#c8f7c5; }
    .panel { background:#fff; padding:10px; border:1px solid #ccc; width:260px; }
    button { margin:4px; padding:6px 10px; }
    select { margin:5px; }
  </style>
</head>
<body>

<!-- หน้าใส่ชื่อ -->
<div id="namePage" style="max-width:400px;margin:40px auto;background:#fff;padding:20px;border:1px solid #ccc;">
  <img src="https://i.ibb.co/GKTz1fP/start.png" style="width:220px;display:block;margin:0 auto 10px;" />
<h2> กรอกข้อมูลก่อนเข้าเกมสิจ๊ะ</h2>
  <p>ชื่อ–สกุล</p>
  <input id="studentName" placeholder="เช่น นายจักรินทร์ ใจหมั่น" style="width:100%;padding:6px;" />
  <p style="margin-top:10px;">เลขที่</p>
  <input id="studentNo" type="number" placeholder="เช่น 2" style="width:100px;padding:6px;" /><br><br>
  <button onclick="startGame()">▶️ เริ่มเกม</button>
</div>

<!-- หน้าเกม -->
<div id="gamePage" style="display:none;">
<h1>เข็นของให้ถึงบ้านของเจ้า😎</h1>
<div style="margin:10px auto;max-width:420px;background:#fff;padding:10px;border:1px solid #ccc;">
</div>
  
<p>สาระที่ 2 วิทยาศาสตร์กายภาพ | แรงและการเคลื่อนที่</p>
<p id="levelInfo"></p>

<div class="game">
  <div class="grid" id="grid"></div>

  <div class="panel">
    <h3>ตั้งค่า</h3>
    วัตถุ:
    <select id="weight">
      <option value="light">เบา</option>
      <option value="medium">ปานกลาง</option>
      <option value="heavy">หนัก</option>
    </select><br>

    แรง:
    <select id="force">
      <option value="1">แรงเบา</option>
      <option value="2">แรงปานกลาง</option>
      <option value="3">แรงมาก</option>
    </select><br>

    ทิศทาง:<br>
    <button onclick="move('up')">⬆</button>
    <button onclick="move('down')">⬇</button><br>
    <button onclick="move('left')">⬅</button>
    <button onclick="move('right')">➡</button><br><br>

    <button onclick="resetGame()">🔄 เริ่มด่านใหม่</button>
    <div id="forceCounter" style="margin-top:10px;font-size:14px;"></div>
    <p id="status"></p>
    <div id="scoreBoard" style="margin-top:10px;font-weight:bold;">คะแนนรวม: 0</div>
  </div>
</div>

<script>
  const grid = document.getElementById('grid');
  const statusText = document.getElementById('status');
  const levelInfo = document.getElementById('levelInfo');

  const allLevels = [
    // ด่านที่ 1 (ปกติ)
    { start:{x:0,y:0}, home:{x:5,y:5}, walls:[{x:2,y:2},{x:3,y:2}], maxMoves:8, weight:'light', score:10 },

    // ด่านที่ 2 (บังคับวัตถุปานกลาง + จำกัดการใช้แรง)
    { start:{x:0,y:5}, home:{x:5,y:0}, walls:[{x:1,y:3},{x:2,y:3},{x:3,y:3}], maxMoves:6, weight:'medium', forceLimit:{1:2,2:2,3:2}, score:15 },

    // ด่านที่ 3 (ต้องเข้าจุดเป้าหมายก่อน)
    { start:{x:5,y:0}, home:{x:0,y:5}, target:{x:3,y:3}, walls:[{x:2,y:1},{x:2,y:2},{x:2,y:3}], maxMoves:6, weight:'medium', score:20 },

    // ด่านที่ 4 (กำแพงซิกแซก)
    { start:{x:0,y:2}, home:{x:5,y:2}, walls:[{x:1,y:1},{x:1,y:2},{x:1,y:3},{x:3,y:1},{x:3,y:2},{x:3,y:3}], maxMoves:7, weight:'medium', score:25 },

    // ด่านที่ 5 (กำแพงเยอะ + เดินจำกัด)
    { start:{x:0,y:5}, home:{x:5,y:0}, walls:[{x:1,y:4},{x:2,y:4},{x:3,y:4},{x:4,y:4},{x:2,y:2},{x:3,y:2}], maxMoves:6, weight:'heavy', score:30 },

    // ด่านที่ 6 (สุดท้าย: เป้าหมาย + กำแพง)
    { start:{x:5,y:5}, home:{x:0,y:0}, target:{x:2,y:3}, walls:[{x:1,y:1},{x:2,y:1},{x:3,y:1},{x:3,y:2},{x:3,y:3}], maxMoves:7, weight:'medium', score:40 }
  ];

  // สุ่มด่าน 4 ด่านจากทั้งหมด 6 ด่าน
  function shuffle(array){
    for(let i=array.length-1;i>0;i--){
      const j = Math.floor(Math.random()*(i+1));
      [array[i],array[j]] = [array[j],array[i]];
    }
    return array;
  }

  const levels = shuffle([...allLevels]).slice(0,4);

  let currentLevel = 0;
  let player = {x:0,y:0};
  let home = {};
  let walls = [];
  let movesLeft = 0;
  let target = null;
  let targetReached = false;
  let forceUsage = {1:0,2:0,3:0};
  let forceLimit = null;
  let totalScore = 0;

  function loadLevel(){
    const lv = levels[currentLevel];
    const weightSelect = document.getElementById('weight');

    forceUsage = {1:0,2:0,3:0};
    forceLimit = lv.forceLimit || null;

    player = {...lv.start};
    home = lv.home;
    target = lv.target || null;
    targetReached = false;
    walls = lv.walls;
    movesLeft = lv.maxMoves;

    weightSelect.value = lv.weight || 'light';
    statusText.textContent = '';
    levelInfo.textContent = `ด่านที่ ${currentLevel+1} | เดินได้อีก ${movesLeft} ครั้ง`;
    document.getElementById('scoreBoard').textContent = `คะแนนรวม: ${totalScore}`;
    updateForceCounter();
    drawGrid();
  }

  function drawGrid() {
    grid.innerHTML = '';
    for(let y=0;y<6;y++){
      for(let x=0;x<6;x++){
        const cell = document.createElement('div');
        cell.className = 'cell';
        if(x===player.x && y===player.y) cell.textContent='🧳';
        if(target && x===target.x && y===target.y) cell.textContent='🎯';
        if(x===home.x && y===home.y) cell.textContent='🏢';
        walls.forEach(w=>{ if(w.x===x && w.y===y) cell.classList.add('wall'); });
        grid.appendChild(cell);
      }
    }
  }

  function move(dir){
    updateForceCounter();
    if(movesLeft<=0){ statusText.textContent='❌ เดินครบจำนวนแล้ว'; return; }

    let force = parseInt(document.getElementById('force').value);
    if(forceLimit){
      if(forceUsage[force] >= forceLimit[force]){
        statusText.textContent = '❌ แรงนี้ใช้ครบแล้ว ต้องใช้แรงอื่น';
        return;
      }
    }
    const weight = document.getElementById('weight').value;

    if(weight==='heavy') force = Math.max(1, Math.floor(force/2));
    if(weight==='light') force = force + 1;

    let dx=0, dy=0;
    if(dir==='up') dy=-1;
    if(dir==='down') dy=1;
    if(dir==='left') dx=-1;
    if(dir==='right') dx=1;

    for(let i=0;i<force;i++){
      const nx = player.x + dx;
      const ny = player.y + dy;

      // ตกออกนอกตาราง
      if(nx<0||ny<0||nx>5||ny>5){
        statusText.textContent = '💥 ตกออกนอกตาราง! กลับไปจุดเริ่มต้น';
        player = {...levels[currentLevel].start};
        drawGrid();
        return;
      }

      // ชนกำแพง
      if(walls.some(w=>w.x===nx&&w.y===ny)) break;

      player.x = nx;
      player.y = ny;
    }

    forceUsage[force] = (forceUsage[force]||0) + 1;
    updateForceCounter();
    movesLeft--;
    levelInfo.textContent = `ด่านที่ ${currentLevel+1} | เดินได้อีก ${movesLeft} ครั้ง`;

    if(target && !targetReached && player.x===target.x && player.y===target.y){
      targetReached = true;
      statusText.textContent = '✅ ถึงจุดเป้าหมายแล้ว ไปบ้านต่อ!';
    }

    if(player.x===home.x && player.y===home.y){
      if(target && !targetReached){
        statusText.textContent = '⚠️ ต้องไปจุดเป้าหมายก่อน!';
        drawGrid();
        return;
      }
      totalScore += levels[currentLevel].score;
      statusText.textContent = `🎉 ผ่านด่าน! ได้ ${levels[currentLevel].score} คะแนน`;
      currentLevel++;
      if(currentLevel < levels.length){
        setTimeout(loadLevel,1000);
      } else {
        statusText.innerHTML = `🏆 จบเกมแล้ว!<br>เล่นทั้งหมด 4 ด่าน<br>คะแนนรวม ${totalScore} คะแนน<br><br><img src="https://i.ibb.co/5hCJTrXF/finish.png" style="width:200px;" />`;
      }
    }
    drawGrid();
  }

  function resetGame(){ loadLevel(); }

  function updateForceCounter(){
    const box = document.getElementById('forceCounter');
    if(!forceLimit){ box.innerHTML = ''; return; }
    box.innerHTML = `
      <b>การใช้แรง (ด่านนี้)</b><br>
      🌟 แรงเบา: ${forceUsage[1]}/${forceLimit[1]}<br>
      💥 แรงปานกลาง: ${forceUsage[2]}/${forceLimit[2]}<br>
      🔥 แรงมาก: ${forceUsage[3]}/${forceLimit[3]}
    `;
  }

  loadLevel();
</script>

</div>

<script>
function startGame(){
  const name = document.getElementById('studentName').value.trim();
  const no = document.getElementById('studentNo').value.trim();
  if(!name || !no){
    alert('กรุณากรอกชื่อ–สกุล และเลขที่');
    return;
  }
  document.getElementById('namePage').style.display='none';
  document.getElementById('gamePage').style.display='block';
  
}
</script>

</body>
</html>
