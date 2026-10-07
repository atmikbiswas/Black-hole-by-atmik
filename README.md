# Black-hole-by-atmik
Pls watch interstellar 
<!DOCTYPE html>
<html lang="en"><head><meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Black Hole</title>
<style>
:root{box-sizing:border-box;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
html,body{height:100%;margin:0;background:#000;color:#e8e6e3;font-family:system-ui,-apple-system,sans-serif;overflow:hidden}
canvas{display:block;position:fixed;inset:0;width:100%;height:100%;touch-action:none;cursor:grab}
#ui{position:fixed;left:0;right:0;bottom:0;padding:12px 12px calc(12px + env(safe-area-inset-bottom,0px));pointer-events:none;text-align:center}
#cap{font-size:13px;line-height:1.4;max-width:460px;margin:0 auto 10px;color:#cfc9c2;text-shadow:0 1px 6px #000}
#cap b{color:#ffb866}
button{pointer-events:auto;background:rgba(255,255,255,.08);color:#fff;border:1px solid rgba(255,255,255,.25);border-radius:20px;padding:8px 14px;margin:2px;font-size:13px}
button.on{background:#ff9a3c;color:#000;border-color:#ff9a3c}
#boom{background:#ff4b2b;border-color:#ff4b2b}button:disabled{opacity:.5}
#t{position:fixed;top:calc(10px + env(safe-area-inset-top,0px));left:14px;font-size:12px;opacity:.6;letter-spacing:.08em}
</style></head><body>
<div id="t">GARGANTUA · drag to orbit · pinch/scroll to zoom</div>
<div id="ui"><div id="cap"></div>
<button data-p="0.06">Edge-on</button><button data-p="0.4">Tilted</button><button data-p="1.5">Face-on</button><button id="spin" class="on">Auto-spin</button><button id="boom">💥 Explode</button></div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
<script id="fs" type="x-shader/x-fragment">
precision highp float;
uniform vec2 uRes;uniform vec3 uCam;uniform mat3 uRot;uniform float uTime;uniform float uExp,uFlash,uShellR,uShellK;
float hash(vec3 p){p=fract(p*0.3183099+.1);p*=17.;return fract(p.x*p.y*p.z*(p.x+p.y+p.z));}
float noise(vec3 x){vec3 i=floor(x),f=fract(x);f=f*f*(3.-2.*f);
return mix(mix(mix(hash(i),hash(i+vec3(1,0,0)),f.x),mix(hash(i+vec3(0,1,0)),hash(i+vec3(1,1,0)),f.x),f.y),
mix(mix(hash(i+vec3(0,0,1)),hash(i+vec3(1,0,1)),f.x),mix(hash(i+vec3(0,1,1)),hash(i+vec3(1,1,1)),f.x),f.y),f.z);}
vec3 stars(vec3 d){vec3 c=vec3(0.);
for(int k=0;k<2;k++){float sc=k==0?70.:150.;vec3 q=d*sc,id=floor(q);float h=hash(id);
if(h>0.96){vec3 o=vec3(hash(id+1.),hash(id+2.),hash(id+3.));float b=smoothstep(.4,0.,length(fract(q)-o))*(h-.96)*25.;
c+=b*mix(vec3(1.,.8,.6),vec3(.7,.8,1.),hash(id+4.));}}
float band=exp(-pow(d.y*2.2+.35*sin(d.x*2.),2.));
c+=vec3(.55,.45,.6)*band*noise(d*7.)*noise(d*18.)*.35;return c;}
void main(){
vec2 uv=(gl_FragCoord.xy-.5*uRes)/uRes.y;
if(uExp>=0.){float w=uExp,len=length(uv),R=w*.3*(1.+w*.25);float x=len-R;
float env=exp(-x*x*(x<0.?4.:40.))*smoothstep(0.,.5,w)*exp(-w*.35);
uv+=uv/max(len,1e-4)*sin(x*(26.+9.*w))*env*.045*(.4+len);}
vec3 v0=normalize(uRot*vec3(uv*.9,-1.));vec3 v=v0;vec3 p=uCam;
vec3 acc=vec3(0.);float T=1.;bool hit=false;
for(int i=0;i<320;i++){
float r=length(p);
if(r<1.){hit=true;break;}
if(r>45.&&dot(p,v)>0.)break;
float dt=clamp(.03+.04*(r-1.),.03,.7);dt=min(dt,.12+abs(p.y)*.5);
if(r>2.6&&r<15.){
float h=.05+.045*r;float dens=exp(-pow(p.y/h,2.));
float fade=smoothstep(2.6,3.4,r)*(1.-smoothstep(9.,15.,r));
if(dens*fade>.002){
float ph=atan(p.z,p.x)-uTime*1.8/(r*sqrt(r));
vec3 np=vec3(cos(ph)*r*.7,sin(ph)*r*.7,r*2.6);
float tex=.35+noise(np)+.6*noise(np*2.4+3.);
vec3 tg=vec3(-p.z,0.,p.x)/r;
float beta=clamp(sqrt(.5/r)*1.5,0.,.8);
float D=clamp(1./(1.+dot(tg,v)*beta),.3,2.6);
float g=sqrt(max(1.-1./r,0.));
float Te=pow(3./r,.75)*D*g*1.25;
vec3 c=mix(vec3(1.,.3,.06),vec3(1.,.82,.6),smoothstep(.15,.85,Te));
c=mix(c,vec3(.8,.88,1.),smoothstep(.95,1.5,Te));
float I=pow(D,3.)*pow(3./r,1.8)*tex*g*3.2;
float a=dens*fade*dt;
acc+=T*c*I*a*1.6;
T*=exp(-a*2.);
}}
vec3 cr=cross(p,v);
v+=-1.5*dot(cr,cr)*p/pow(r,5.)*dt;v=normalize(v);
p+=v*dt;
}
vec3 col=acc;
if(!hit)col+=stars(v)*T;
col*=1.+uFlash*1.5;
if(uShellR>0.){float b=length(cross(uCam,v0));float Rs=uShellR,th=Rs*.22+.4;
float s1=sqrt(max(Rs*Rs-b*b,0.)),s2=sqrt(max((Rs-th)*(Rs-th)-b*b,0.));
float fil=.35+1.3*noise(v0*7.+uTime*.05)*noise(v0*17.);
col+=mix(vec3(1.,.4,.1),vec3(1.,.85,.6),uShellK)*(s1-s2)*fil*uShellK*.3;}
col+=vec3(1.,.85,.6)*uFlash*(exp(-dot(uv,uv)*5.)*3.+.15);
col=1.-exp(-col*1.15);
col=pow(col,vec3(.5));
gl_FragColor=vec4(col,1.);}
</script>
<script>
const texts={
"0.06":"<b>Edge-on:</b> the <b>halo</b> — light from the disk's far side bends over the top — and the <b>underbelly arc</b> bending beneath the hole. The left side glows brighter from <b>Doppler beaming</b>.",
"0.4":"<b>Tilted:</b> lensing wraps the back of the disk above the shadow, while the near side sweeps in front. Gas moving toward you is brighter and bluer.",
"1.5":"<b>Face-on:</b> the disk is seen flat, with no halo or arc split. The bent far-side light appears as a thin ring hugging the black shadow."};
const cap=document.getElementById('cap');
const R=new THREE.WebGLRenderer({antialias:false});R.setPixelRatio(Math.min(devicePixelRatio,1.25));R.autoClear=false;
document.body.prepend(R.domElement);
const sc=new THREE.Scene(),cam=new THREE.OrthographicCamera(-1,1,1,-1,0,1);
const U={uExp:{value:-1},uFlash:{value:0},uShellR:{value:0},uShellK:{value:0},uRes:{value:new THREE.Vector2()},uCam:{value:new THREE.Vector3()},uRot:{value:new THREE.Matrix3()},uTime:{value:0}};
sc.add(new THREE.Mesh(new THREE.PlaneGeometry(2,2),new THREE.ShaderMaterial({uniforms:U,vertexShader:'void main(){gl_Position=vec4(position.xy,0.,1.);}',fragmentShader:document.getElementById('fs').textContent})));
const pcam=new THREE.PerspectiveCamera(48.46,innerWidth/innerHeight,.1,400),ps=new THREE.Scene();
const N=16000,dirs=new Float32Array(N*3),pp=new Float32Array(N*4);
for(let i=0;i<N;i++){let d,sp,sz;
if(i<N*.07){const a=Math.random()*.16,ph=Math.random()*6.283,sg=Math.random()<.5?1:-1;d=[Math.sin(a)*Math.cos(ph),sg*Math.cos(a),Math.sin(a)*Math.sin(ph)];sp=9+Math.random()*5;sz=.12+Math.random()*.25}
else{let x,y,z;do{x=Math.random()*2-1;y=Math.random()*2-1;z=Math.random()*2-1;const l=Math.hypot(x,y,z);if(l>1||l<.1)continue;x/=l;y/=l;z/=l;
const f=.5+.5*Math.sin(x*5+1.3)*Math.sin(y*4+.7)*Math.sin(z*6);if(Math.random()<.3+.7*f)break}while(true);
d=[x,y,z];sp=(2.5+Math.pow(Math.random(),.7)*7)*(1+.4*(1-Math.abs(y)));sz=Math.random()<.3?.07+Math.random()*.12:.2+Math.random()*.45}
dirs.set(d,i*3);pp.set([sp,7+Math.random()*9,sz,Math.random()],i*4)}
const pg=new THREE.BufferGeometry();pg.setAttribute('position',new THREE.BufferAttribute(dirs,3));pg.setAttribute('aP',new THREE.BufferAttribute(pp,4));
const PM=new THREE.ShaderMaterial({uniforms:{uT:{value:0},uScale:{value:500}},transparent:true,blending:THREE.AdditiveBlending,depthTest:false,depthWrite:false,
vertexShader:`attribute vec4 aP;uniform float uT,uScale;varying float vAge,vVis;
void main(){float travel=aP.x*(1.-exp(-.2*uT))/.2;vec3 pos=position*(1.6+travel);
pos+=vec3(sin(aP.w*40.+uT*.7),cos(aP.w*31.+uT*.5),sin(aP.w*17.+uT*.6))*.012*travel;
vec4 mv=modelViewMatrix*vec4(pos,1.);gl_Position=projectionMatrix*mv;
float age=uT/aP.y;vAge=age;
vec3 rv=pos-cameraPosition;float L=length(rv);vec3 rn=rv/L;float tc=-dot(cameraPosition,rn);float dm=length(cameraPosition+rn*tc);
vVis=((tc>0.&&tc<L)?smoothstep(2.4,3.3,dm):1.)*smoothstep(1.5,6.,-mv.z);
gl_PointSize=clamp(aP.z*(1.+age*2.)*uScale/max(-mv.z,.1),1.5,64.);}`,
fragmentShader:`uniform float uT;varying float vAge,vVis;
void main(){float a=smoothstep(.5,.05,length(gl_PointCoord-.5));
float heat=clamp(1.-vAge*1.3,0.,1.);
vec3 c=mix(vec3(.55,.08,.03),vec3(1.,.45,.1),smoothstep(0.,.5,heat));
c=mix(c,vec3(1.,.9,.7),smoothstep(.5,.9,heat));c=mix(c,vec3(.75,.85,1.),smoothstep(.92,1.,heat));
float fade=pow(clamp(1.-vAge,0.,1.),1.4)*min(1.,uT*5.);
gl_FragColor=vec4(c*(.5+heat*1.5),a*fade*vVis*.55);}`});
const pts3=new THREE.Points(pg,PM);pts3.frustumCulled=false;ps.add(pts3);
let ex=-1,tb=-1,burst=false,capBefore='';
texts.wave="<b>Gravitational waves:</b> spacetime ripples spread outward and bend the starlight and disk behind them as the star's core collapses…";
texts.burst="<b>Supernova:</b> the blast front and ejecta rush outward, cooling from white-hot to deep red, while two fast jets punch out along the spin axis.";
const boom=document.getElementById('boom');
boom.onclick=()=>{if(ex>=0)return;capBefore=cap.innerHTML;ex=performance.now();boom.disabled=true;boom.textContent='…';cap.innerHTML=texts.wave;td=Math.max(td,26)};
function size(){R.setSize(innerWidth,innerHeight);const s=R.getPixelRatio();U.uRes.value.set(innerWidth*s,innerHeight*s);pcam.aspect=innerWidth/innerHeight;pcam.updateProjectionMatrix();PM.uniforms.uScale.value=innerHeight*s/.9}
addEventListener('resize',size);size();
let yaw=0,pitch=.06,tp=.06,dist=24,td=24,spin=true,drag=false,lx=0,ly=0,pts={},pd=0;
const cv=R.domElement;
function setPreset(p){tp=p;document.querySelectorAll('button[data-p]').forEach(b=>b.classList.toggle('on',b.dataset.p==p));cap.innerHTML=texts[p]}
document.querySelectorAll('button[data-p]').forEach(b=>b.onclick=()=>setPreset(+b.dataset.p));
document.getElementById('spin').onclick=e=>{spin=!spin;e.target.classList.toggle('on',spin)};
cv.addEventListener('pointerdown',e=>{pts[e.pointerId]=e;lx=e.clientX;ly=e.clientY;drag=true;cv.setPointerCapture(e.pointerId)});
cv.addEventListener('pointermove',e=>{
if(!pts[e.pointerId])return;pts[e.pointerId]=e;const k=Object.values(pts);
if(k.length==2){const d=Math.hypot(k[0].clientX-k[1].clientX,k[0].clientY-k[1].clientY);if(pd)td=Math.min(60,Math.max(8,td*pd/d));pd=d;return}
yaw-=(e.clientX-lx)*.006;tp=Math.max(-1.5,Math.min(1.5,tp+(e.clientY-ly)*.006));pitch=tp;lx=e.clientX;ly=e.clientY;
document.querySelectorAll('button[data-p]').forEach(b=>b.classList.remove('on'))});
const up=e=>{delete pts[e.pointerId];pd=0;drag=Object.keys(pts).length>0};
cv.addEventListener('pointerup',up);cv.addEventListener('pointercancel',up);
cv.addEventListener('wheel',e=>{e.preventDefault();td=Math.min(60,Math.max(8,td*(1+e.deltaY*.001)))},{passive:false});
setPreset(.06);
const clock=new THREE.Clock();
function frame(){
const dt=clock.getDelta();U.uTime.value+=dt;
let shake=0;
if(ex>=0){const t=(performance.now()-ex)/1000;tb=t-3.2;U.uExp.value=t;
shake=tb<0?.0012*Math.pow(t/3.2,2):.006*Math.exp(-tb*2.5);
if(tb>=0){if(!burst){burst=true;cap.innerHTML=texts.burst;td=Math.max(td,30)}
PM.uniforms.uT.value=tb;U.uFlash.value=Math.min(1,tb*12)*Math.exp(-tb*1.6);
U.uShellR.value=2+18*(1-Math.exp(-.35*tb));U.uShellK.value=.9*Math.exp(-.35*tb)*Math.min(1,tb*4)}
if(t>19){ex=-1;tb=-1;burst=false;U.uExp.value=-1;U.uFlash.value=0;U.uShellR.value=0;boom.disabled=false;boom.textContent='💥 Explode';cap.innerHTML=capBefore}}
if(spin&&!drag)yaw+=dt*.08;
pitch+=(tp-pitch)*Math.min(1,dt*5);dist+=(td-dist)*Math.min(1,dt*6);
const cp=Math.cos(pitch);
const pos=new THREE.Vector3(cp*Math.sin(yaw),Math.sin(pitch),cp*Math.cos(yaw)).multiplyScalar(dist).add(new THREE.Vector3(Math.random()-.5,Math.random()-.5,Math.random()-.5).multiplyScalar(shake*dist));
const f=pos.clone().negate().normalize(),r=new THREE.Vector3().crossVectors(f,new THREE.Vector3(0,1,0)).normalize(),u=new THREE.Vector3().crossVectors(r,f),b=f.clone().negate();
U.uCam.value.copy(pos);U.uRot.value.set(r.x,u.x,b.x,r.y,u.y,b.y,r.z,u.z,b.z);
pcam.position.copy(pos);pcam.lookAt(0,0,0);R.clear();R.render(sc,cam);if(tb>=0)R.render(ps,pcam);requestAnimationFrame(frame)}
frame();
</script></body></html>