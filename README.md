# n1rv4na.github.io
<!DOCTYPE html>
<html lang="lv">
<head>
<base target="_top">
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>CHRISTMAS WISHES</title>
<style>
:root{--card:rgba(255,255,255,.08);--text:#f6efe9;--muted:#a9b8c8;--accent:#ff5a4d;--accent2:#7fe3a8;--border:rgba(255,255,255,.16);color-scheme:dark}
*{box-sizing:border-box}
body{margin:0;min-height:100vh;background:radial-gradient(ellipse at top,#1d3b5c 0%,#0f1f33 55%,#08101c 100%) fixed;color:var(--text);font-family:system-ui,-apple-system,"Segoe UI",Roboto,sans-serif;line-height:1.4}
main{position:relative;z-index:1;max-width:640px;margin:0 auto;padding:72px 16px 48px}
h1{font-size:1.9rem;margin:0 0 4px;text-align:center;text-shadow:0 0 18px rgba(255,211,92,.45)}
.sub{text-align:center;color:var(--muted);margin:0 0 24px}
form{background:var(--card);border:1px solid var(--border);border-radius:14px;padding:16px;display:grid;gap:10px;backdrop-filter:blur(6px)}
input[type=text]{width:100%;padding:11px 12px;border:1px solid var(--border);border-radius:10px;font-size:1rem;background:rgba(0,0,0,.28);color:var(--text)}
button{font:inherit;cursor:pointer}
.add{background:var(--accent);color:#fff;border:0;border-radius:10px;padding:11px;font-weight:600}
.add:disabled{opacity:.6;cursor:wait}
ul{list-style:none;padding:0;margin:20px 0 0;display:grid;gap:10px}
li{background:var(--card);border:1px solid var(--border);border-radius:12px;padding:12px 14px;display:flex;align-items:center;gap:12px;backdrop-filter:blur(6px)}
li .body{flex:1;min-width:0}
li .name{font-weight:600;overflow-wrap:anywhere}
li a{color:var(--accent2);font-size:.9rem;overflow-wrap:anywhere}
li.done .name{text-decoration:line-through;color:var(--muted)}
input[type=checkbox]{width:20px;height:20px;accent-color:var(--accent2)}
.del{background:none;border:0;color:var(--muted);font-size:1.3rem;padding:4px 8px}
.del:hover{color:var(--accent)}
.empty{text-align:center;color:var(--muted);margin-top:24px}
#snow{position:fixed;inset:0;pointer-events:none;z-index:4}
#lights{position:fixed;top:0;left:0;right:0;height:56px;pointer-events:none;z-index:5;overflow:hidden}
#lights .b{filter:drop-shadow(0 0 5px var(--c));animation:tw 1.8s ease-in-out infinite}
@keyframes tw{0%,100%{opacity:1}50%{opacity:.35}}
@media (prefers-reduced-motion:reduce){#lights .b{animation:none}}
</style>
</head>
<body>
<canvas id="snow"></canvas>
<div id="lights"></div>
<main>
  <h1>🎄 CHRISTMAS WISHES</h1>
  <p class="sub">Pievieno, ko vēlies. Saraksts tiek glabāts un redzams visiem, kas atver šo lapu.</p>
  <form id="f">
    <input type="text" id="name" placeholder="Ko vēlies?" required maxlength="120">
    <input type="text" id="link" placeholder="Saite (nav obligāta)" inputmode="url" maxlength="500">
    <button class="add" id="addBtn" type="submit">Pievienot</button>
  </form>
  <ul id="list"></ul>
  <p class="empty" id="empty">Ielādē…</p>
</main>
<script>
(function(){
  var items=[],loaded=false;
  var list=document.getElementById("list"),empty=document.getElementById("empty"),btn=document.getElementById("addBtn");
  function fixUrl(u){u=String(u||"").trim();if(!u)return"";if(!/^https?:\/\//i.test(u))u="https://"+u;try{var p=new URL(u);return(p.protocol==="http:"||p.protocol==="https:")?p.href:"";}catch(e){return"";}}
  function call(fn){
    var args=[].slice.call(arguments,1);
    var r=google.script.run.withSuccessHandler(function(res){items=res||[];loaded=true;btn.disabled=false;render();}).withFailureHandler(function(err){btn.disabled=false;alert("Neizdevās saglabāt: "+(err&&err.message?err.message:err));});
    r[fn].apply(r,args);
  }
  function render(){
    list.textContent="";
    empty.textContent="Saraksts pagaidām tukšs.";
    empty.style.display=(loaded&&!items.length)?"block":(loaded?"none":"block");
    items.forEach(function(it){
      var li=document.createElement("li");if(it.done)li.className="done";
      var cb=document.createElement("input");cb.type="checkbox";cb.checked=!!it.done;cb.title="Atzīmēt kā saņemtu";
      cb.onchange=function(){call("toggleItem",it.id,cb.checked);};
      var body=document.createElement("div");body.className="body";
      var n=document.createElement("div");n.className="name";n.textContent=it.name;body.appendChild(n);
      var safe=fixUrl(it.link);
      if(safe){var a=document.createElement("a");a.href=safe;a.target="_blank";a.rel="noopener noreferrer";a.textContent=safe.replace(/^https?:\/\//,"");body.appendChild(a);}
      var d=document.createElement("button");d.className="del";d.type="button";d.textContent="×";d.title="Dzēst";
      d.onclick=function(){call("deleteItem",it.id);};
      li.appendChild(cb);li.appendChild(body);li.appendChild(d);list.appendChild(li);
    });
  }
  document.getElementById("f").addEventListener("submit",function(e){
    e.preventDefault();
    var n=document.getElementById("name"),l=document.getElementById("link");
    var name=n.value.trim();if(!name)return;
    btn.disabled=true;
    call("addItem",name,fixUrl(l.value));
    n.value="";l.value="";
  });
  var cv=document.getElementById("snow"),cx=cv.getContext("2d"),fl=[],W=0,H=0,dpr=Math.min(window.devicePixelRatio||1,2);
  var still=window.matchMedia&&matchMedia("(prefers-reduced-motion: reduce)").matches;
  function mk(top){return{x:Math.random()*W,y:top?-10:Math.random()*H,r:Math.random()*2.6+0.8,v:Math.random()*0.9+0.4,s:Math.random()*6.28,a:Math.random()*0.5+0.4};}
  function size(){W=innerWidth;H=innerHeight;cv.width=W*dpr;cv.height=H*dpr;cx.setTransform(dpr,0,0,dpr,0,0);var n=Math.max(40,Math.min(140,Math.round(W/12)));fl=[];for(var i=0;i<n;i++)fl.push(mk(false));}
  function draw(){cx.clearRect(0,0,W,H);for(var i=0;i<fl.length;i++){var f=fl[i];if(!still){f.s+=0.012;f.y+=f.v;f.x+=Math.sin(f.s)*0.5;if(f.y>H+10)fl[i]=mk(true);}cx.beginPath();cx.fillStyle="rgba(255,255,255,"+f.a+")";cx.arc(f.x,f.y,f.r,0,6.283);cx.fill();}if(!still)requestAnimationFrame(draw);}
  function lights(){var el=document.getElementById("lights");el.textContent="";var w=innerWidth,sw=120,n=Math.ceil(w/sw),ns="http://www.w3.org/2000/svg",cols=["#ff5a4d","#ffd35c","#7fe3a8","#6cc4ff","#ff8fd1"],k=0;
    var svg=document.createElementNS(ns,"svg");svg.setAttribute("width",w);svg.setAttribute("height",56);
    var d="M0,6";for(var i=0;i<n;i++)d+=" Q"+(i*sw+sw/2)+",50 "+((i+1)*sw)+",6";
    var p=document.createElementNS(ns,"path");p.setAttribute("d",d);p.setAttribute("fill","none");p.setAttribute("stroke","rgba(255,255,255,.28)");p.setAttribute("stroke-width","1.5");svg.appendChild(p);
    for(var j=0;j<n;j++){[0.125,0.375,0.625,0.875].forEach(function(t){var x=j*sw+t*sw,y=6+88*t*(1-t),c=cols[k%cols.length];var b=document.createElementNS(ns,"circle");b.setAttribute("cx",x);b.setAttribute("cy",y+6);b.setAttribute("r",5);b.setAttribute("fill",c);b.setAttribute("class","b");b.style.setProperty("--c",c);b.style.animationDelay=((k*0.37)%1.8)+"s";svg.appendChild(b);k++;});}
    el.appendChild(svg);}
  size();draw();lights();
  window.addEventListener("resize",function(){size();lights();if(still)draw();});
  call("getItems");
})();
</script>
</body>
</html>
