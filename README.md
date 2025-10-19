<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Colorful Dice</title>
<style>
  :root{
    --bg:#0f172a;
    --card:#071029;
    --muted:#9aa4b2;
    --accent:#6366f1;
  }
  *{box-sizing:border-box;font-family:Inter,ui-sans-serif,system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;}
  html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg),#071026);color:#e6eef8}
  .wrap{max-width:980px;margin:40px auto;padding:24px;background:linear-gradient(180deg,rgba(255,255,255,0.02),transparent);border-radius:12px;box-shadow:0 6px 30px rgba(2,6,23,0.6);text-align:center;}
  h1{margin:0 0 6px;font-size:28px;font-weight:700;letter-spacing:1px}
  p.lead{margin:0 0 18px;color:var(--muted);font-size:13px}
  .dice-grid{display:grid;grid-template-columns:repeat(4,1fr);gap:16px;margin:18px 0;justify-items:center;}
  .die{width:120px;height:120px;border-radius:20px;cursor:pointer;user-select:none;box-shadow:0 8px 18px rgba(2,6,23,0.5);transform-origin:center;transition:transform .18s ease, box-shadow .18s ease;}
  .die:active{transform:scale(.96) rotate(-2deg);}
  .controls{display:flex;gap:10px;align-items:center;justify-content:center;margin-top:18px;}
  button.primary{background:var(--accent);border:0;padding:10px 14px;border-radius:10px;color:white;font-weight:600;cursor:pointer;box-shadow:0 6px 18px rgba(99,102,241,0.15)}
  button.ghost{background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--muted);padding:9px 12px;border-radius:10px;cursor:pointer}
  button:disabled{opacity:.6;cursor:not-allowed}
  .secret-panel{position:fixed;right:18px;bottom:18px;background:var(--card);padding:8px; font-size:12px;border-radius:12px;box-shadow:0 8px 30px rgba(2,6,23,0.6);display:none;min-width:150px; max-width:180px;}
  .secret-panel h3{margin:0 0 8px;font-size:14px}
  .color-row{display:flex;align-items:center;justify-content:space-between;padding:6px 0;border-top:1px solid rgba(255,255,255,0.02)}
  .color-swatch{width:28px;height:28px;border-radius:8px;box-shadow:inset 0 -6px 18px rgba(0,0,0,0.2);margin-right:10px;border:1px solid rgba(0,0,0,0.12)}
  .row-left{display:flex;align-items:center}
  label.switch{position:relative;display:inline-block;width:42px;height:24px}
  label.switch input{display:none}
  .track{position:absolute;cursor:pointer;background:#2b313a;border-radius:24px;left:0;right:0;top:0;bottom:0;transition:background .2s}
  .thumb{position:absolute;width:18px;height:18px;border-radius:50%;top:3px;left:3px;background:white;transition:left .2s, transform .2s}
  input:checked + .track{background:linear-gradient(90deg,#16a34a,#4ade80)}
  input:checked + .track .thumb{left:21px;transform:translateX(0)}
  @media (max-width:640px){.dice-grid{grid-template-columns:repeat(2,1fr)}}
</style>
</head>
<body>
<div class="wrap" role="main">
  <h1>Colorful Dice</h1>
  <div id="dice" class="dice-grid" aria-live="polite" aria-label="Four colorful dice"></div>
  <div class="controls">
    <button id="rollBtn" class="primary">Roll</button>
    <button id="resetBtn" class="ghost">Reset</button>
  </div>
</div>

<div id="secret" class="secret-panel" role="dialog" aria-hidden="true" aria-label="Secret settings menu">
  <h3>Secret Color Controls</h3>
  <div class="color-row"><div class="row-left"><div class="color-swatch" style="background:#ef4444"></div><div>Red</div></div><label class="switch"><input type="checkbox" data-color="red" checked><span class="track"><span class="thumb"></span></span></label></div>
  <div class="color-row"><div class="row-left"><div class="color-swatch" style="background:#3b82f6"></div><div>Blue</div></div><label class="switch"><input type="checkbox" data-color="blue" checked><span class="track"><span class="thumb"></span></span></label></div>
  <div class="color-row"><div class="row-left"><div class="color-swatch" style="background:#10b981"></div><div>Green</div></div><label class="switch"><input type="checkbox" data-color="green" checked><span class="track"><span class="thumb"></span></span></label></div>
  <div class="color-row"><div class="row-left"><div class="color-swatch" style="background:#f59e0b"></div><div>Yellow</div></div><label class="switch"><input type="checkbox" data-color="yellow" checked><span class="track"><span class="thumb"></span></span></label></div>
</div>

<script>
(function(){
  const COLORS = [
    {id:'red', css:'#ef4444'},
    {id:'blue', css:'#3b82f6'},
    {id:'green', css:'#10b981'},
    {id:'yellow', css:'#f59e0b'}
  ];
  const COUNT = 4;
  document.addEventListener("keydown", e => { if(e.key==="y"||e.key==="Y"){ toggleSecret(); } });
  const diceEl = document.getElementById('dice');
  const rollBtn = document.getElementById('rollBtn');
  const resetBtn = document.getElementById('resetBtn');
  const secretPanel = document.getElementById('secret');
  const allowed = {}; COLORS.forEach(c => allowed[c.id]=true);

  let clickCount=0; let clickTimer=null;

  rollBtn.addEventListener('click',()=>{
    clickCount++;
    if(clickTimer) clearTimeout(clickTimer);
    clickTimer=setTimeout(()=>{clickCount=0;clickTimer=null;},800);
    if(clickCount>=3){clickCount=0;toggleSecret();}
    rollAll();
  });
  resetBtn.addEventListener('click',()=>{render();});

  function render(){
    diceEl.innerHTML='';
    for(let i=0;i<COUNT;i++){
      const die=document.createElement('div');
      die.className='die';
      die.style.background=randomColor();
      die.addEventListener('click',()=>{die.style.background=randomColor();});
      diceEl.appendChild(die);
    }
  }

  function randomColor(){
    const allowedColors = COLORS.filter(c=>allowed[c.id]);
    if(allowedColors.length===0) return '#64748b';
    return allowedColors[Math.floor(Math.random()*allowedColors.length)].css;
  }

  function rollAll(){
    const dies=[...document.querySelectorAll('.die')];
    dies.forEach(d=>{
      d.style.transform='scale(1.2) rotate(10deg)';
      setTimeout(()=>{
        d.style.background=randomColor();
        d.style.transform='scale(1) rotate(0deg)';
      },200);
    });
  }

  function toggleSecret(){
    const isOpen=secretPanel.style.display==='block';
    secretPanel.style.display=isOpen?'none':'block';
    secretPanel.style.transform=isOpen?'scale(0.9)':'scale(1)';
    secretPanel.setAttribute('aria-hidden',String(isOpen));
  }

  document.querySelectorAll('#secret input[type=checkbox]').forEach(cb=>{
    cb.addEventListener('change',()=>{
      const color=cb.dataset.color;
      allowed[color]=cb.checked;
      if(!Object.values(allowed).some(v=>v)){
        cb.checked=true; allowed[color]=true; alert('At least one color must stay enabled.');
      }
    });
  });

  render();
})();
</script>
</body>
</html>
[colorful_dice_small_secret.html](https://github.com/user-attachments/files/22992324/colorful_dice_small_secret.html)
