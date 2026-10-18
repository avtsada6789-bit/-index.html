<!DOCTYPE html>
<html lang="th">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>NEON KI 3D BATTLE</title>
<style>
*{box-sizing:border-box}
body{margin:0;overflow:hidden;background:#050816;color:white;font-family:Arial,sans-serif;touch-action:none}
canvas{display:block}
#hud{position:fixed;top:10px;left:10px;right:10px;display:flex;justify-content:space-between;gap:10px;pointer-events:none}
.panel{width:42%;font-weight:bold;text-shadow:0 0 8px cyan}
.bar{height:14px;border:1px solid white;background:#222;margin-top:5px}
.fill{height:100%;width:100%;background:linear-gradient(90deg,cyan,blue)}
.enemy .fill{background:linear-gradient(90deg,orange,red)}
#timer{font-size:28px;font-weight:bold}
#msg{position:fixed;top:36%;width:100%;text-align:center;font-size:38px;font-weight:900;color:#fff;text-shadow:0 0 20px cyan;pointer-events:none}
#controls{position:fixed;bottom:18px;left:12px;right:12px;display:flex;justify-content:space-between;align-items:end}
.pad,.skills{display:grid;gap:6px}
.pad{grid-template-columns:repeat(3,48px);grid-template-rows:repeat(3,45px)}
.skills{grid-template-columns:repeat(2,62px)}
button{color:white;background:#101e45dd;border:1px solid #00eaff;border-radius:14px;font-weight:bold;font-size:17px;touch-action:none}
.skills button{height:57px;font-size:13px;border-radius:50%;box-shadow:0 0 12px #008cff}
.blank{visibility:hidden}
#help{position:fixed;top:75px;width:100%;text-align:center;font-size:11px;color:#9ceaff;pointer-events:none}
</style>
</head>
<body>
<div id="hud">
 <div class="panel">⚡ RAI<div class="bar"><div id="hp" class="fill"></div></div></div>
 <div id="timer">99</div>
 <div class="panel enemy" style="text-align:right">🔥 KAGE<div class="bar"><div id="ehp" class="fill"></div></div></div>
</div>
<div id="help">ลากนิ้วบนฉากเพื่อหมุนกล้อง</div>
<div id="msg">FIGHT!</div>
<div id="controls">
 <div class="pad">
  <button class="blank"></button><button data-m="f">▲</button><button class="blank"></button>
  <button data-m="l">◀</button><button data-m="b">▼</button><button data-m="r">▶</button>
  <button class="blank"></button><button data-a="jump">⤒</button><button class="blank"></button>
 </div>
 <div class="skills">
  <button data-a="hit">👊<br>ต่อย</button>
  <button data-a="kick">🦵<br>เตะ</button>
  <button data-a="dash">⚡<br>พุ่ง</button>
  <button data-a="special">🌩️<br>สกิล</button>
 </div>
</div>
<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js';

const scene=new THREE.Scene();
scene.background=new THREE.Color(0x050816);
scene.fog=new THREE.Fog(0x050816,12,35);

const camera=new THREE.PerspectiveCamera(55,innerWidth/innerHeight,.1,100);
const renderer=new THREE.WebGLRenderer({antialias:true});
renderer.setSize(innerWidth,innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio,1.5));
renderer.shadowMap.enabled=true;
document.body.prepend(renderer.domElement);

scene.add(new THREE.HemisphereLight(0x8bdcff,0x160022,2));
const light=new THREE.DirectionalLight(0xffffff,3);
light.position.set(4,8,5);
light.castShadow=true;
scene.add(light);

const floor=new THREE.Mesh(
 new THREE.CylinderGeometry(7,7,.3,48),
 new THREE.MeshStandardMaterial({color:0x101a35,metalness:.6,roughness:.35})
);
floor.position.y=-.2;
floor.receiveShadow=true;
scene.add(floor);

const grid=new THREE.GridHelper(14,28,0x00cfff,0x182b55);
grid.position.y=-.04;
scene.add(grid);

const ring=new THREE.Mesh(
 new THREE.TorusGeometry(5.5,.035,8,80),
 new THREE.MeshBasicMaterial({color:0x00ddff})
);
ring.rotation.x=Math.PI/2;
ring.position.y=0;
scene.add(ring);

function material(color,glow=0){
 return new THREE.MeshStandardMaterial({
  color,emissive:glow,emissiveIntensity:1.7,
  metalness:.4,roughness:.3
 });
}

function fighter(x,isEnemy){
 const g=new THREE.Group();
 scene.add(g);
 g.position.set(x,0,0);

 const suit=material(isEnemy?0x30132b:0x14254a,
                     isEnemy?0x5c0030:0x002c66);
 const skin=material(0xe0b394);
 const auraMat=new THREE.MeshBasicMaterial({
  color:isEnemy?0xff174f:0x00cfff,
  wireframe:true,transparent:true,opacity:.28
 });

 function part(geo,mat,x,y,z,sx=1,sy=1,sz=1){
  const o=new THREE.Mesh(geo,mat);
  o.position.set(x,y,z);
  o.scale.set(sx,sy,sz);
  o.castShadow=true;
  g.add(o);
  return o;
 }

 const sphere=new THREE.SphereGeometry(1,16,12);
 part(sphere,suit,0,1.25,0,.36,.5,.23);
 part(sphere,skin,0,1.9,0,.27,.29,.25);
 part(sphere,material(0x111827),0,2.1,0,.28,.13,.27);

 for(const s of [-1,1]){
  part(sphere,suit,s*.43,1.35,0,.14,.38,.15);
  part(sphere,material(isEnemy?0xff245d:0x00eaff,
                       isEnemy?0x990022:0x0055ff),
       s*.44,1.03,0,.15,.13,.15);
  part(sphere,suit,s*.19,.58,0,.16,.4,.17);
  part(sphere,material(0x111827),s*.19,.16,.06,.21,.12,.3);
 }

 const aura=part(new THREE.SphereGeometry(.8,18,14),
                 auraMat,0,1.1,0);
 return {g,aura,hp:100,isEnemy};
}

const p=fighter(-1.8,false);
const e=fighter(1.8,true);

let hp=100,ehp=100,time=99,ended=false;
const held=new Set(),particles=[];
let angle=0,zoom=1;
let msg=document.querySelector('#msg');

function message(t){
 msg.textContent=t;
 clearTimeout(message.t);
 message.style.opacity=1;
 message.t=setTimeout(()=>msg.style.opacity=0,800);
}

function burst(pos,color,n=20){
 for(let i=0;i<n;i++){
  const m=new THREE.Mesh(
   new THREE.SphereGeometry(.035+Math.random()*.04,6,6),
   new THREE.MeshBasicMaterial({color,transparent:true})
  );
  m.position.copy(pos);
  scene.add(m);
  particles.push({
   m,life:.4+Math.random()*.5,
   v:new THREE.Vector3(
    (Math.random()-.5)*5,
    Math.random()*4,
    (Math.random()-.5)*5
   )
  });
 }
}

function attack(type){
 if(ended)return;
 const d=p.g.position.distanceTo(e.g.position);

 if(type==='dash'){
  p.g.position.x+=Math.sign(e.g.position.x-p.g.position.x)*.8;
  burst(p.g.position,0x00eaff,12);
  return;
 }
 if(type==='jump'){
  p.g.position.y=.7;
  setTimeout(()=>p.g.position.y=0,350);
  return;
 }

 const special=type==='special';
 const damage=special?22:type==='kick'?13:9;
 message(special?'⚡ THUNDER BREAK!':
         type==='kick'?'KICK!':'HIT!');

 burst(e.g.position.clone().add(new THREE.Vector3(0,1,0)),
       special?0xffffff:0x00cfff,special?50:22);

 if(d<3.2){
  ehp=Math.max(0,ehp-damage);
  document.querySelector('#ehp').style.width=ehp+'%';
  e.g.position.x+=Math.sign(e.g.position.x-p.g.position.x)*.3;
  if(ehp<=0){ended=true;message('YOU WIN! ⚡');}
 }else message('MISS!');
}

document.querySelectorAll('[data-m]').forEach(b=>{
 const k=b.dataset.m;
 b.addEventListener('pointerdown',ev=>{
  ev.preventDefault();held.add(k);
 });
 ['pointerup','pointercancel','lostpointercapture'].forEach(ev=>
  b.addEventListener(ev,()=>held.delete(k))
);
});

document.querySelectorAll('[data-a]').forEach(b=>{
 b.addEventListener('pointerdown',ev=>{
  ev.preventDefault();attack(b.dataset.a);
 });
});

let dragging=false,lastX=0,lastY=0;
renderer.domElement.addEventListener('pointerdown',ev=>{
 dragging=true;lastX=ev.clientX;lastY=ev.clientY;
});
window.addEventListener('pointerup',()=>dragging=false);
window.addEventListener('pointermove',ev=>{
 if(!dragging)return;
 angle-=(ev.clientX-lastX)*.008;
 zoom=Math.max(.75,Math.min(1.4,zoom+(ev.clientY-lastY)*.003));
 lastX=ev.clientX;lastY=ev.clientY;
});

let last=performance.now(),elapsed=0;
function animate(now){
 requestAnimationFrame(animate);
 const dt=Math.min((now-last)/1000,.04);
 last=now;elapsed+=dt;

 if(!ended){
  time-=dt;
  if(time<=0){
   time=0;ended=true;
   message(ehp<hp?'YOU WIN!':'CPU WINS!');
  }
  document.querySelector('#timer').textContent=Math.ceil(time);

  const speed=dt*2.7;
  if(held.has('l'))p.g.position.x-=speed;
  if(held.has('r'))p.g.position.x+=speed;
  if(held.has('f'))p.g.position.z-=speed;
  if(held.has('b'))p.g.position.z+=speed;

  p.g.position.x=THREE.MathUtils.clamp(p.g.position.x,-5,5);
  p.g.position.z=THREE.MathUtils.clamp(p.g.position.z,-4,4);

  const dx=p.g.position.x-e.g.position.x;
  const dz=p.g.position.z-e.g.position.z;
  const d=Math.hypot(dx,dz);

  e.g.rotation.y=Math.atan2(dx,dz);
  if(d>1.5){
   e.g.position.x+=dx/d*dt*.7;
   e.g.position.z+=dz/d*dt*.7;
  }else if(Math.random()<dt*.7){
   hp=Math.max(0,hp-4);
   document.querySelector('#hp').style.width=hp+'%';
   burst(p.g.position.clone().add(new THREE.Vector3(0,1,0)),0xff2455,7);
   if(hp<=0){ended=true;message('CPU WINS!');}
  }

  p.g.rotation.y=Math.atan2(-dx,-dz);
  p.aura.rotation.y+=dt*1.5;
  e.aura.rotation.y-=dt*1.8;
  p.aura.scale.setScalar(1+.08*Math.sin(elapsed*8));
  e.aura.scale.setScalar(1+.08*Math.cos(elapsed*7));
 }

 for(let i=particles.length-1;i>=0;i--){
  const q=particles[i];
  q.life-=dt;
  q.m.position.addScaledVector(q.v,dt);
  q.v.y-=dt*3;
  q.m.material.opacity=Math.max(0,q.life*2);
  if(q.life<=0){
   scene.remove(q.m);
   q.m.geometry.dispose();
   q.m.material.dispose();
   particles.splice(i,1);
  }
 }

 const center=new THREE.Vector3(
  (p.g.position.x+e.g.position.x)/2, .8,
  (p.g.position.z+e.g.position.z)/2
 );
 const radius=10*zoom;
 camera.position.set(
  center.x+Math.sin(angle)*radius,
  5,
  center.z+Math.cos(angle)*radius
 );
 camera.lookAt(center);
 renderer.render(scene,camera);
}
requestAnimationFrame(animate);

addEventListener('resize',()=>{
 camera.aspect=innerWidth/innerHeight;
 camera.updateProjectionMatrix();
 renderer.setSize(innerWidth,innerHeight);
});
</script>
</body>
</html>

