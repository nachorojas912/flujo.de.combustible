<!doctype html>
<html lang="es">
<head>
  <meta charset="utf-8">
  <title>Indicador de Flujo de Combustible Realista</title>
  <style>
    body{
      margin:0;
      background:#0b1118;
      color:#fff;
      font-family:sans-serif;
      display:flex;
      flex-direction:column;
      align-items:center;
      justify-content:center;
      height:100vh;
    }
    h1{font-size:20px;margin-bottom:10px}
    svg{width:360px;height:360px;}
    input{width:360px;margin-top:20px;}
    .valor{margin-top:10px;font-size:20px;font-family:monospace}
  </style>
</head>
<body>
  <h1>Indicador de Flujo de Combustible (kg/h)</h1>
  <svg viewBox="0 0 200 200">
    <defs>
      <radialGradient id="bgGrad" cx="50%" cy="50%" r="50%">
        <stop offset="0%" stop-color="#141b23"/>
        <stop offset="100%" stop-color="#0b1118"/>
      </radialGradient>
    </defs>
    <g transform="translate(100,100)">
      <!-- fondo panel -->
      <circle cx="0" cy="0" r="98" fill="url(#bgGrad)" stroke="#555" stroke-width="4"/>
      <circle cx="0" cy="0" r="100" fill="none" stroke="#aaa" stroke-width="6"/>

      <!-- arcos de colores según rango real -->
      <path id="greenArc" fill="none" stroke-width="10"/>
      <path id="yellowArc" fill="none" stroke-width="10"/>
      <path id="redArc" fill="none" stroke-width="10"/>

      <!-- marcas y números -->
      <g id="marks"></g>

      <!-- aguja -->
      <g id="needle">
        <path d="M0,-4 L85,0 L0,4 Z" fill="#ff4b4b" stroke="#880000" stroke-width="0.5"/>
        <circle cx="0" cy="0" r="5" fill="#fff" stroke="#cc0000" stroke-width="1"/>
      </g>
    </g>
  </svg>

  <input type="range" id="slider" min="0" max="2500" value="0">
  <div class="valor"><span id="num">0</span> kg/h</div>

  <script>
    const slider = document.getElementById('slider');
    const num = document.getElementById('num');
    const needle = document.getElementById('needle');
    const marks = document.getElementById('marks');
    const MAX = 2500;

    const greenArc = document.getElementById('greenArc');
    const yellowArc = document.getElementById('yellowArc');
    const redArc = document.getElementById('redArc');

    // función auxiliar para convertir ángulo a coordenadas
    function polarToXY(deg,r){
      const rad = deg * Math.PI/180;
      return {x: r*Math.cos(rad), y: r*Math.sin(rad)};
    }

    function arcPath(startDeg,endDeg,r){
      const a1 = polarToXY(startDeg,r);
      const a2 = polarToXY(endDeg,r);
      const large = Math.abs(endDeg-startDeg)>180?1:0;
      return `M ${a1.x} ${a1.y} A ${r} ${r} 0 ${large} 1 ${a2.x} ${a2.y}`;
    }

    // convertir rango flujo → ángulo
    function flujoToAngulos(fStart,fEnd){
      const start = -120 + 240*(fStart/MAX);
      const end = -120 + 240*(fEnd/MAX);
      return {start,end};
    }

    // asignar arcos
    const g = flujoToAngulos(0,1800);
    greenArc.setAttribute("d",arcPath(g.start,g.end,100));
    greenArc.setAttribute("stroke","#2dd4bf");

    const y = flujoToAngulos(1800,2200);
    yellowArc.setAttribute("d",arcPath(y.start,y.end,100));
    yellowArc.setAttribute("stroke","#facc15");

    const r = flujoToAngulos(2200,2500);
    redArc.setAttribute("d",arcPath(r.start,r.end,100));
    redArc.setAttribute("stroke","#fb7185");

    // crear marcas y números
    for(let i=0;i<=10;i++){
      const ang = -120 + i*24;
      const rad = ang*Math.PI/180;
      const r1 = 85, r2 = 95;
      const x1 = r1*Math.cos(rad), y1 = r1*Math.sin(rad);
      const x2 = r2*Math.cos(rad), y2 = r2*Math.sin(rad);

      const line = document.createElementNS("http://www.w3.org/2000/svg","line");
      line.setAttribute("x1",x1); line.setAttribute("y1",y1);
      line.setAttribute("x2",x2); line.setAttribute("y2",y2);
      line.setAttribute("stroke", i%5===0?"#fff":"#ccc");
      line.setAttribute("stroke-width", i%5===0?2:1);
      marks.appendChild(line);

      if(i%2===0){
        const txt = document.createElementNS("http://www.w3.org/2000/svg","text");
        txt.textContent = (i*MAX/10).toFixed(0);
        txt.setAttribute("x",(r1-18)*Math.cos(rad));
        txt.setAttribute("y",(r1-18)*Math.sin(rad)+4);
        txt.setAttribute("fill","#fff");
        txt.setAttribute("font-size","9");
        txt.setAttribute("text-anchor","middle");
        marks.appendChild(txt);
      }
    }

    function angleFromValue(val){ return -120 + (val/MAX)*240; }

    function update(){
      const v = Number(slider.value);
      num.textContent = v;
      needle.setAttribute("transform",`rotate(${angleFromValue(v)})`);
    }

    slider.addEventListener('input',update);
    update();
  </script>
</body>
</html>
