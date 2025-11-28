# speech

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Speech Therapy Training</title>

  <style>
    :root{
      --bg1:#1e1e2f; --bg2:#2d2d44; --panel:#2c2c3e; --muted:#aaa;
      --accent:#4a90e2; --gold:#ffda79;
    }
    html,body{height:100%}
    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg,var(--bg1) 0%,var(--bg2) 100%);
      color: #f0f0f0;
      margin: 0;
      padding: 12px;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
    }
    .container {
      background: var(--panel);
      padding: 20px;
      border-radius: 20px;
      box-shadow: 0 10px 30px rgba(0,0,0,0.45);
      text-align: center;
      width: 520px;
      max-width: 100%;
    }
    h1 { color: var(--gold); margin-bottom: 12px; font-size: 26px; }
    label{display:block; margin-bottom:6px; color:var(--muted); font-size:13px}
    .top-row{
      display:flex; 
      gap:8px; 
      align-items:center; 
      justify-content:center; 
      flex-wrap:wrap;
      width:100%;
    }

    select, input[type="text"], button {
      margin: 6px 4px;
      padding: 12px 14px;
      font-size: 16px;
      border-radius: 10px;
      border: none;
      outline: none;
      transition: all .18s ease;
    }
    select, input[type="text"]{
      background:#3a3a50; color:#fff; min-width:180px;
      width:100%;
      box-sizing:border-box;
    }
    select:hover, input[type="text"]:hover{ background:#44445a }

    h2#targetWord{
      font-size:22px; color:#9cdcfe; margin:18px 0;
      min-height:60px; text-transform:lowercase;
    }

    .progress-bar { width:100%; height:8px; background:#3a3a50; border-radius:6px; overflow:hidden; }
    .progress { height:100%; background:var(--accent); width:0%; transition: width .45s }

    .controls{
      display:flex; gap:8px; justify-content:center; flex-wrap:wrap; margin-top:12px;
      width:100%;
    }

    button { background:var(--accent); color:#fff; font-weight:600; cursor:pointer; }
    button:hover{ transform:translateY(-2px) }
    button.secondary{ background:#6a6a80 }

    #result {
      margin-top:18px;
      font-size:17px;
      padding:12px 14px;
      border-radius:10px;
      display:inline-block;
      min-width:240px;
      max-width:100%;
      word-wrap:break-word;
    }
    .correct{ background:#2d6a4f; border:2px solid #28a745 }
    .almost{ background:#735d32; border:2px solid #ffb703 }
    .wrong{ background:#6a1e1e; border:2px solid #dc3545 }

    .recording-indicator{ display:none; color:#ff6b6b; margin-top:10px; animation:pulse 1.5s infinite; font-size:16px; }
    @keyframes pulse{0%{opacity:.6}50%{opacity:1}100%{opacity:.6}}

    .confidence{ font-size:13px; color:var(--muted); margin-top:6px }
    .score-box{ margin-top:10px; font-size:17px; color:var(--gold) }

    .instructions{ font-size:13px; color:var(--muted); margin-top:20px }

    /* -------------------------
       📱 RESPONSIVE FIXES
       ------------------------- */

    @media (max-width: 480px){
      body {
        padding: 6px;
      }
      .container{
        padding:16px;
        border-radius:16px;
      }
      h1{
        font-size:22px;
      }
      h2#targetWord{
        font-size:18px;
        min-height:40px;
      }
      select, input[type="text"]{
        min-width:100%;
        font-size:15px;
        padding:10px;
      }
      button{
        width:100%;
        padding:12px;
        font-size:16px;
      }
      #result{
        font-size:16px;
        width:100%;
        min-width:auto;
      }
      .controls{
        flex-direction:column;
        gap:10px;
      }
      .top-row{
        flex-direction:column;
      }
    }

  </style>
</head>

<body>
  <div class="container">

    <h1>Speech Therapy Training</h1>

    <div class="top-row">
      <div style="width:100%">
        <label>Choose training</label>
        <select id="mode" onchange="setTarget()">
          <option value="english">English words</option>
          <option value="hindi">Hindi words</option>
          <option value="twister">Tongue twisters</option>
        </select>
      </div>

      <div style="width:100%">
        <label>Custom sample</label>
        <input id="customSample" type="text" placeholder="type your own phrase...">
      </div>
    </div>

    <h2 id="targetWord">select a mode and click "new"</h2>

    <div class="progress-bar"><div id="progress" class="progress"></div></div>

    <div class="controls">
      <button onclick="newTarget()">🔄 New</button>
      <button onclick="speakSample()">🎧 Hear Sample</button>
      <button onclick="startListening()">🎤 Speak</button>
      <button class="secondary" id="tryBtn" onclick="tryAgain()" style="display:none">❌ Try Again</button>
    </div>

    <div id="recordingIndicator" class="recording-indicator">🎤 recording...</div>

    <div id="result">say what's shown above!</div>
    <div id="confidence" class="confidence"></div>

    <div class="score-box">⭐ Score: <span id="score">0</span></div>

    <div class="instructions">
      1. Choose mode → 2. Click <b>New</b> → 3. (Optional) type custom text →  
      4. Click <b>Hear Sample</b> → 5. Say it using <b>Speak</b>
    </div>
  </div>

<script>
/* ========== Word Lists ========== */
const englishWords = [
  "hello","good morning","thank you","how are you","i am fine",
  "what is your name","nice to meet you","please help me",
  "i love learning","good night"
];
const hindiWords = [
  "नमस्ते (namaste)","कैसे हो (kaise ho)","धन्यवाद (dhanyavaad)",
  "मैं ठीक हूँ (main theek hoon)","क्या कर रहे हो (kya kar rahe ho)",
  "आपका नाम क्या है (aapka naam kya hai)","मुझे हिंदी पसंद है (mujhe hindi pasand hai)"
];
const twisters = [
  "she sells seashells by the seashore",
  "red lorry yellow lorry",
  "peter piper picked a peck of pickled peppers",
  "chandu ke chacha ne chandu ki chachi ko chandi ke chamach se chatni chatai",
  "kaccha papita pakta nahi, pakka papita kaccha nahi"
];

/* ========== STATE ========== */
let target = "";
let recognitionLang = "en-US";
let score = +localStorage.getItem("therapyScore") || 0;
let attempts = +localStorage.getItem("therapyAttempts") || 0;
let correctAttempts = +localStorage.getItem("therapyCorrect") || 0;

let recognition = null;
let listening = false;

document.getElementById("score").innerText = score;

/* ========== HELPERS ========== */

function updateScoreStorage(){
  document.getElementById("score").innerText = score;
  localStorage.setItem("therapyScore", score);
  localStorage.setItem("therapyAttempts", attempts);
  localStorage.setItem("therapyCorrect", correctAttempts);
}

function updateProgress() {
  const percent = attempts ? Math.round((correctAttempts/attempts)*100) : 0;
  document.getElementById("progress").style.width = percent + "%";
}

/* ========== TARGET LOGIC ========== */

function setTarget(){ newTarget(); }

function newTarget(){
  const mode = document.getElementById("mode").value;
  let list = englishWords;

  if(mode === "hindi"){ list = hindiWords; recognitionLang = "hi-IN"; }
  else if(mode === "twister"){ list = twisters; recognitionLang = "en-US"; }
  else{ recognitionLang = "en-US"; }

  target = list[Math.random()*list.length|0];
  document.getElementById("targetWord").innerText = target.toLowerCase();

  resetUI();
}

function resetUI(){
  document.getElementById("result").innerText = "say what's shown above!";
  document.getElementById("result").className = "";
  document.getElementById("confidence").innerText = "";
  document.getElementById("recordingIndicator").style.display = "none";
  document.getElementById("tryBtn").style.display = "none";
}

/* ========== SPEAK SAMPLE ========== */
function speakSample(){
  let text = document.getElementById("customSample").value.trim() || target;
  text = text.replace(/\(.*?\)/g,"").trim();

  speechSynthesis.cancel();
  const u = new SpeechSynthesisUtterance(text.toLowerCase());
  u.lang = recognitionLang;
  u.rate = 0.95;
  speechSynthesis.speak(u);
}

/* ========== SIMILARITY ENGINE ========== */
function editDistance(a,b){
  a = a.toLowerCase(); b = b.toLowerCase();
  const dp = Array.from({length:a.length+1},()=>Array(b.length+1).fill(0));
  for(let i=0;i<=a.length;i++) dp[i][0]=i;
  for(let j=0;j<=b.length;j++) dp[0][j]=j;

  for(let i=1;i<=a.length;i++){
    for(let j=1;j<=b.length;j++){
      dp[i][j] = Math.min(
        dp[i-1][j]+1,
        dp[i][j-1]+1,
        dp[i-1][j-1] + (a[i-1]===b[j-1]?0:1)
      );
    }
  }
  return dp[a.length][b.length];
}
function similarity(a,b){
  const long = a.length >= b.length ? a : b;
  const short = a.length >= b.length ? b : a;
  return (long.length - editDistance(long,short)) / long.length;
}
function wordSimilarity(s,t){
  let S = s.toLowerCase().split(/\s+/);
  let T = t.toLowerCase().split(/\s+/);
  let matches=0;
  T.forEach(tw=>{
    if(S.some(sw=>similarity(sw,tw)>0.82)) matches++;
  });
  return matches/T.length;
}

/* ========== SPEECH RECOGNITION ========== */

function startListening(){
  const SpeechRec = window.SpeechRecognition || window.webkitSpeechRecognition;
  if(!SpeechRec){ alert("Speech recognition unsupported"); return; }

  if(recognition && listening){ try{ recognition.stop() }catch(e){} }

  recognition = new SpeechRec();
  recognition.lang = recognitionLang;
  recognition.interimResults = false;
  recognition.maxAlternatives = 3;

  document.getElementById("recordingIndicator").style.display = "block";
  document.getElementById("result").innerText = "listening...";
  listening = true;

  recognition.onresult = (event)=>{
    listening = false;
    document.getElementById("recordingIndicator").style.display = "none";

    const best = event.results[0][0];
    let spoken = best.transcript.toLowerCase().trim();
    let confidence = Math.max(best.confidence || 0, 0.01);

    let compareTarget =
      (document.getElementById("customSample").value.trim() || target)
        .toLowerCase()
        .replace(/\(.*?\)/g,"")
        .trim();

    attempts++;

    if(spoken === compareTarget){
      correct(spoken);
      return;
    }

    const sim = wordSimilarity(spoken, compareTarget);

    if(sim > 0.80){
      correct(spoken);
    }
    else if(sim > 0.65){
      almost(spoken);
    }
    else{
      wrong(spoken);
    }

    document.getElementById("confidence").innerText =
      `recognition confidence: ${(confidence*100).toFixed(1)}%`;

    updateScoreStorage();
    updateProgress();
  };

  recognition.onerror = e=>{
    listening=false;
    document.getElementById("recordingIndicator").style.display="none";
    document.getElementById("result").innerText =
      "⚠️ error: " + (e.error || "unknown");
    document.getElementById("result").className = "wrong";
  };

  recognition.start();
}

function correct(s){
  document.getElementById("result").innerText = `✅ great! you said '${s}'`;
  document.getElementById("result").className = "correct";
  score++;
  correctAttempts++;
}
function almost(s){
  document.getElementById("result").innerText = `⚠️ almost: you said '${s}'`;
  document.getElementById("result").className = "almost";
  correctAttempts++;
}
function wrong(s){
  document.getElementById("result").innerText = `❌ try again: you said '${s}'`;
  document.getElementById("result").className = "wrong";
  if(score>0) score--;
  document.getElementById("tryBtn").style.display="inline-block";
}

function tryAgain(){
  document.getElementById("tryBtn").style.display="none";
  startListening();
}

updateProgress();
</script>

</body>
</html>
