<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Weather Pro</title>

<link rel="manifest" href="manifest.json">

<style>
body {
    font-family: Arial;
    text-align: center;
    padding: 20px;
    color: white;
    background: linear-gradient(to right,#4facfe,#00f2fe);
    overflow-x: hidden;
}

canvas {
    position: fixed;
    top:0; left:0;
    pointer-events:none;
}

.app {
    max-width: 350px;
    margin: auto;
    background: rgba(0,0,0,0.3);
    padding: 20px;
    border-radius: 20px;
}

button,input {
    padding:10px;
    margin:5px;
    border-radius:8px;
    border:none;
}

button { background:orange; color:white; }

.forecast {
    display:flex;
    overflow-x:auto;
    gap:10px;
}

.card {
    background: rgba(255,255,255,0.2);
    padding:10px;
    border-radius:10px;
}
</style>
</head>

<body>

<canvas id="rain"></canvas>

<div class="app">
<h2>🌦 Weather App</h2>

<input id="city" placeholder="Enter city">
<br>
<button onclick="getWeather()">Search</button>

<div id="result"></div>
<canvas id="chart" height="150"></canvas>
<div class="forecast" id="forecast"></div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

<script>
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('sw.js');
}

function icon(c){
    if(c==0)return"☀️";
    if(c<=3)return"⛅";
    if(c<=67)return"🌧️";
    return"⛈️";
}

// Rain animation
const canvas=document.getElementById("rain");
const ctx=canvas.getContext("2d");
canvas.width=window.innerWidth;
canvas.height=window.innerHeight;

let drops=[];
for(let i=0;i<100;i++){
    drops.push({x:Math.random()*canvas.width,y:Math.random()*canvas.height,l:Math.random()*20});
}

function drawRain(){
    ctx.clearRect(0,0,canvas.width,canvas.height);
    ctx.strokeStyle="rgba(255,255,255,0.5)";
    drops.forEach(d=>{
        ctx.beginPath();
        ctx.moveTo(d.x,d.y);
        ctx.lineTo(d.x,d.y+d.l);
        ctx.stroke();
        d.y+=5;
        if(d.y>canvas.height)d.y=0;
    });
    requestAnimationFrame(drawRain);
}
drawRain();

// Weather
async function getWeather(){
    let city=document.getElementById("city").value;

    let geo=await fetch(`https://geocoding-api.open-meteo.com/v1/search?name=${city}`);
    let g=await geo.json();

    let {latitude,longitude,name}=g.results[0];

    let res=await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${latitude}&longitude=${longitude}&hourly=temperature_2m&daily=weathercode,temperature_2m_max&current_weather=true&timezone=auto`);
    let d=await res.json();

    let w=d.current_weather;

    document.getElementById("result").innerHTML=`
        <h3>${name}</h3>
        <h1>${icon(w.weathercode)}</h1>
        <p>${w.temperature}°C</p>
    `;

    // Graph
    new Chart(document.getElementById("chart"),{
        type:"line",
        data:{
            labels:d.hourly.time.slice(0,12),
            datasets:[{
                label:"Temp",
                data:d.hourly.temperature_2m.slice(0,12)
            }]
        }
    });

    // Forecast
    let html="";
    d.daily.time.forEach((day,i)=>{
        html+=`<div class="card">
            <p>${new Date(day).toDateString().slice(0,3)}</p>
            <p>${icon(d.daily.weathercode[i])}</p>
            <p>${d.daily.temperature_2m_max[i]}°</p>
        </div>`;
    });

    document.getElementById("forecast").innerHTML=html;
}
</script>

</body>
</html>
