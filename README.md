# modul-basuri.
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Basuri Music Studio</title>
<style>
*{box-sizing:border-box}
body{
 margin:0;background:#101522;color:white;
 font-family:Arial,sans-serif;padding:16px
}
header{text-align:center;padding:15px}
h2{margin:5px;color:#55d6ff}
.panel{
 background:#1b2536;border-radius:16px;
 padding:16px;margin-bottom:15px
}
button{
 border:0;border-radius:10px;padding:13px;
 font-weight:bold;cursor:pointer
}
.controls{display:flex;gap:9px;flex-wrap:wrap}
.controls button{flex:1}
.play{background:#18b981;color:white}
.stop{background:#ef5350;color:white}
.record{background:#3979ed;color:white}
.keys{
 display:grid;grid-template-columns:repeat(4,1fr);
 gap:9px
}
.key{
 background:#e9edf5;color:#192438;
 height:85px;font-size:18px;
 border-radius:12px;
 box-shadow:0 5px #9da9bb;
 transition:.08s
}
.key.active{
 background:#37d5ff;color:#071522;
 box-shadow:0 0 20px #37d5ff;
 transform:translateY(4px)
}
select,input{
 width:100%;padding:12px;border:0;
 border-radius:9px;margin:8px 0
}
.small{color:#9cabc2;font-size:12px}
</style>
</head>
<body>

<header>
<h2>🎹 BASURI STUDIO</h2>
<div class="small">ONLINE MUSIC MODULE</div>
</header>

<div class="panel">
<b>Kontrol Musik</b>
<div class="controls" style="margin-top:14px">
<button class="play" onclick="playSequence()">▶ PLAY</button>
<button class="stop" onclick="stopSequence()">■ STOP</button>
<button class="record" onclick="recordMode()">● REKAM</button>
</div>
<p id="status" class="small">Siap dimainkan</p>
</div>

<div class="panel">
<b>Keyboard Basuri</b>
<p class="small">Tekan tombol untuk memainkan nada.</p>
<div class="keys" id="keyboard"></div>
</div>

<div class="panel">
<b>Simpan Nada</b>
<input id="presetName" placeholder="Nama nada Basuri">
<div class="controls">
<button class="record" onclick="savePreset()">SIMPAN</button>
<button class="stop" onclick="clearSequence()">HAPUS</button>
</div>
<p class="small">Preset tersimpan maksimal 50.</p>
</div>

<div class="panel">
<b>Daftar Nada Tersimpan</b>
<div id="presetList"></div>
</div>

<script>
const names=["Do","Re","Mi","Fa","Sol","La",
 "Si","Do+","Re+","Mi+","Fa+","Sol+"];
const frequencies=[
261.63,293.66,329.63,349.23,392,440,
493.88,523.25,587.33,659.25,698.46,783.99
];

let audio;
let sequence=[];
let recording=false;
let playing=false;
let timers=[];

function initAudio(){
 if(!audio){
  audio=new (window.AudioContext||window.webkitAudioContext)();
 }
 if(audio.state==="suspended")audio.resume();
}

function tone(index,duration=0.35){
 initAudio();
 const osc=audio.createOscillator();
 const gain=audio.createGain();
 osc.type="sawtooth";
 osc.frequency.value=frequencies[index];
 gain.gain.setValueAtTime(0.0001,audio.currentTime);
 gain.gain.exponentialRampToValueAtTime(.15,audio.currentTime+.03);
 gain.gain.exponentialRampToValueAtTime(.0001,audio.currentTime+duration);
 osc.connect(gain);
 gain.connect(audio.destination);
 osc.start();
 osc.stop(audio.currentTime+duration+.03);
}

const keyboard=document.getElementById("keyboard");

names.forEach((name,i)=>{
 const btn=document.createElement("button");
 btn.className="key";
 btn.textContent=name;
 btn.onclick=()=>{
  if(playing)return;
  tone(i);
  lightKey(i);
  if(recording)sequence.push(i);
 };
 keyboard.appendChild(btn);
});

function lightKey(i){
 const keys=document.querySelectorAll(".key");
 keys[i].classList.add("active");
 setTimeout(()=>keys[i].classList.remove("active"),350);
}

function recordMode(){
 recording=!recording;
 document.getElementById("status").textContent=
 recording?"● Sedang merekam":"Rekaman selesai";
 if(recording)sequence=[];
}

function stopSequence(){
 playing=false;
 timers.forEach(clearTimeout);
 timers=[];
 document.getElementById("status").textContent="Dihentikan";
}

function clearSequence(){
 stopSequence();
 sequence=[];
 document.getElementById("status").textContent="Nada dihapus";
}

function playSequence(){
 if(!sequence.length){
  alert("Belum ada nada yang direkam!");
  return;
 }
 stopSequence();
 playing=true;
 document.getElementById("status").textContent="Sedang diputar...";
 sequence.forEach((note,i)=>{
  const timer=setTimeout(()=>{
   if(!playing)return;
   tone(note);
   lightKey(note);
   if(i===sequence.length-1){
    playing=false;
    document.getElementById("status").textContent="Selesai";
   }
  },i*400);
  timers.push(timer);
 });
}

function savePreset(){
 const name=document.getElementById("presetName").value.trim();
 if(!name||!sequence.length){
  alert("Isi nama dan rekam nada dahulu!");
  return;
 }
 let presets=JSON.parse(localStorage.getItem("basuriPresets")||"[]");
 if(presets.length>=50){
  alert("Maksimal 50 preset!");
  return;
 }
 presets.push({name,notes:[...sequence]});
 localStorage.setItem("basuriPresets",JSON.stringify(presets));
 document.getElementById("presetName").value="";
 renderPresets();
 alert("Preset berhasil disimpan!");
}

function renderPresets(){
 const list=document.getElementById("presetList");
 const presets=JSON.parse(localStorage.getItem("basuriPresets")||"[]");
 list.innerHTML="";
 presets.forEach((p,i)=>{
  const row=document.createElement("div");
  row.style.cssText="background:#29374c;padding:12px;border-radius:9px;margin:8px 0";
  const title=document.createElement("b");
  title.textContent=p.name;
  const play=document.createElement("button");
  play.textContent="▶";
  play.style.marginLeft="10px";
  play.onclick=()=>{
   sequence=p.notes.slice();
   playSequence();
  };
  const del=document.createElement("button");
  del.textContent="Hapus";
  del.style.marginLeft="8px";
  del.onclick=()=>{
   presets.splice(i,1);
   localStorage.setItem("basuriPresets",JSON.stringify(presets));
   renderPresets();
  };
  row.append(title,play,del);
  list.appendChild(row);
 });
}
renderPresets();
</script>
</body>
</html>
