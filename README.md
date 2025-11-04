# breathing-wave
3D 숨 쉬는 파도 시각화
<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>3D 숨 쉬는 파도</title>
  <!-- p5.js CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.min.js"></script>
  <style>
    :root{
      --panel-bg: rgba(255,255,255,0.95);
      --panel-radius: 14px;
      --accent: #3b82f6;
      --glass: rgba(255,255,255,0.08);
    }

    html,body{
      margin:0; padding:0; height:100%; width:100%;
      background:#000; font-family: Pretendard, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
      -webkit-font-smoothing:antialiased; -moz-osx-font-smoothing:grayscale;
    }

    /* Controls */
    .controls{
      position:fixed; left:16px; top:16px; z-index:9999;
      background:var(--panel-bg); backdrop-filter: blur(6px) saturate(120%);
      padding:14px; border-radius:var(--panel-radius); box-shadow:0 8px 24px rgba(0,0,0,0.25);
      min-width:220px; max-width:360px; transition: transform .28s ease, opacity .22s ease, height .28s ease;
    }

    .controls.collapsed{ height:48px; overflow:hidden; padding:8px 12px; }

    .toolbar{ display:flex; align-items:center; justify-content:space-between; gap:8px }
    .title{ font-weight:700; font-size:14px; color:#222 }
    .small{ font-size:12px; color:#444 }

    .toggle-btn{
      background:transparent; border:none; cursor:pointer; font-size:15px; padding:6px 8px; border-radius:10px;
    }

    .control-content{ margin-top:10px; display:grid; gap:8px }
    label{ font-size:13px; color:#333 }
    input[type=number], input[type=range]{ width:100%; padding:6px 8px; border-radius:8px; border:1px solid #e6e6e6 }
    .row{ display:flex; gap:8px; align-items:center }
    .btns{ display:flex; gap:8px; margin-top:6px }
    .btn{ flex:1; padding:8px 10px; border-radius:10px; border:none; cursor:pointer; font-weight:600 }
    .apply{ background:var(--accent); color:#fff }
    .finish{ background:#111; color:#fff }

    /* small helper text */
    .hint{ font-size:12px; color:#666 }

    /* mobile tweaks */
    @media (max-width:520px){
      .controls{ left:10px; right:10px; top:10px; }
      .title{ font-size:13px }
    }

    /* hide default p5 canvas outline on some browsers */
    canvas{ display:block }
  </style>
</head>
<body>
  <div class="controls" id="controlPanel">
    <div class="toolbar">
      <div>
        <div class="title">3D 숨 쉬는 파도</div>
        <div class="small">상호작용 가능한 파형 시각화</div>
      </div>
      <button class="toggle-btn" id="toggleBtn" title="컨트롤 숨기기">⚙️</button>
    </div>

    <div class="control-content" id="controlContent">
      <div>
        <label for="amp">두려움의 크기 (amp)</label>
        <input id="amp" type="number" step="1" value="30" />
      </div>

      <div>
        <label for="waveCount">불확실한 것들의 수 (wave count)</label>
        <input id="waveCount" type="number" step="1" min="1" max="24" value="7" />
      </div>

      <div>
        <label for="speed">압박감 (speed factor)</label>
        <input id="speed" type="number" step="0.05" value="0.3" />
      </div>

      <div class="row">
        <div style="flex:1">
          <label for="cols">가로 해상도 (cols)</label>
          <input id="cols" type="number" step="1" value="200" />
        </div>
        <div style="width:92px">
          <label for="rows">세로(rows)</label>
          <input id="rows" type="number" step="1" value="50" />
        </div>
      </div>

      <div class="btns">
        <button class="btn apply" id="applyBtn">변경 적용</button>
        <button class="btn finish" id="finishBtn">완료</button>
      </div>

      <div class="hint">* '완료'를 누르면 점(point) 대신 면(vertex)으로 연결된 드로잉이 됩니다.</div>
    </div>
  </div>

  <script>
    // --- p5 sketch variables ---
    let n = 7;
    let A = [], k = [], w = [], phi = [];
    let cols = 200, rows = 50;
    let ampFactor = 30, speedFactor = 0.3;
    let baseHue, waveColor;
    let drawingComplete = false;

    function setup(){
      let cnv = createCanvas(windowWidth, windowHeight, WEBGL);
      cnv.style('display','block');
      colorMode(HSB, 360, 100, 100, 255);
      baseHue = random(0,360);
      initWaves();
    }

    function windowResized(){
      resizeCanvas(windowWidth, windowHeight);
    }

    function initWaves(){
      A = []; k = []; w = []; phi = [];
      for(let i=0;i<n;i++){
        A[i] = random(6, ampFactor);
        k[i] = random(0.03, 0.22);
        w[i] = random(0.06, speedFactor);
        phi[i] = random(TWO_PI);
      }
    }

    function draw(){
      background(0);

      // camera / orientation
      rotateX(PI/3);
      translate(-width/2, -height*0.08, -200);

      strokeWeight(drawingComplete ? 1.2 : 1);
      let hueShift = (frameCount * 0.45) % 360;
      waveColor = color((baseHue + hueShift) % 360, 80, 90, 220);

      stroke(waveColor);
      noFill();

      for(let z=0; z<rows; z++){
        if(drawingComplete) beginShape();
        for(let x=0; x<cols; x++){
          let y = 0;
          for(let i=0;i<n;i++){
            y += A[i] * sin(k[i]*x - w[i]*frameCount*0.1 + phi[i] + z*0.5);
          }
          let vx = x * (width / cols);
          let vz = z * 8;
          if(drawingComplete){
            vertex(vx, y, vz);
          }else{
            point(vx, y, vz);
          }
        }
        if(drawingComplete) endShape();
      }
    }

    // --- UI bindings ---
    const panel = document.getElementById('controlPanel');
    const toggleBtn = document.getElementById('toggleBtn');
    const controlContent = document.getElementById('controlContent');

    toggleBtn.addEventListener('click', ()=>{
      panel.classList.toggle('collapsed');
      // animate a quick hue reset so the visual changes slightly when toggling
      baseHue = (baseHue + 20) % 360;
    });

    document.getElementById('applyBtn').addEventListener('click', ()=>{
      n = int(document.getElementById('waveCount').value) || 7;
      ampFactor = float(document.getElementById('amp').value) || 30;
      speedFactor = float(document.getElementById('speed').value) || 0.3;
      cols = int(document.getElementById('cols').value) || 200;
      rows = int(document.getElementById('rows').value) || 50;
      initWaves();
      baseHue = random(0,360);
      drawingComplete = false;
    });

    document.getElementById('finishBtn').addEventListener('click', ()=>{
      drawingComplete = true;
    });

    // small helpers to mimic p5's int/float when used outside the sketch functions
    function int(v){ return Math.floor(Number(v)); }
    function float(v){ return Number(v); }

    // allow keyboard shortcuts: space = toggle drawing mode, r = randomize
    window.addEventListener('keydown', (e)=>{
      if(e.code === 'Space'){
        drawingComplete = !drawingComplete;
      }else if(e.key.toLowerCase() === 'r'){
        initWaves(); baseHue = random(0,360);
      }
    });

  </script>
</body>
</html>
