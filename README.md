<html lang="fr">
<head>
<meta name="viewport" content="width=device-width, height=device-height, initial-scale=1.0">
<meta charset="UTF-8">
<title>Parking Game</title>
<style>
body{margin:0;overflow:hidden;background:#000}
#ui{
position:absolute;
top:10px;left:10px;
color:white;font-family:Arial;
background:rgba(0,0,0,.6);
padding:10px;border-radius:8px
}

/* MOBILE CONTROLS */
#mobile{
position:absolute;
bottom:20px;
width:100%;
display:none;
justify-content:space-between;
padding:0 20px;
box-sizing:border-box
}
.btn{
width:70px;height:70px;
background:rgba(255,255,255,.2);
border-radius:50%;
display:flex;
align-items:center;
justify-content:center;
color:white;
font-size:24px;
user-select:none
}
</style>
</head>
<body>

<div id="ui">
Niveau: <span id="lvl">1</span><br>
W/S accélérer<br>
A/D tourner
</div>

<div id="mobile">
<div>
<div class="btn" id="left">◀</div>
<div class="btn" id="right">▶</div>
</div>
<div>
<div class="btn" id="gas">▲</div>
<div class="btn" id="brake">▼</div>
</div>
</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.158/build/three.min.js"></script>
<script>

/* DETECT MOBILE */
let isMobile=/Android|iPhone|iPad|iPod/i.test(navigator.userAgent)
if(isMobile){
document.getElementById("mobile").style.display="flex"
}

/* SCENE */
let scene=new THREE.Scene()

/* SKY */
let sky=new THREE.Mesh(
new THREE.SphereGeometry(200,32,32),
new THREE.MeshBasicMaterial({
color:0x87ceeb,
side:THREE.BackSide
})
)
scene.add(sky)

/* CAMERA */
let camera=new THREE.PerspectiveCamera(70,innerWidth/innerHeight,.1,1000)

/* RENDER */
let renderer=new THREE.WebGLRenderer({antialias:true})
renderer.setSize(innerWidth,innerHeight)
document.body.appendChild(renderer.domElement)

/* LIGHT */
scene.add(new THREE.AmbientLight(0x888888))
let sun=new THREE.DirectionalLight(0xffffff,1)
sun.position.set(10,20,10)
scene.add(sun)

/* GROUND */
let ground=new THREE.Mesh(
new THREE.PlaneGeometry(80,80),
new THREE.MeshStandardMaterial({color:0x444444})
)
ground.rotation.x=-Math.PI/2
scene.add(ground)

/* PARKING LINES */
function slot(x,z){
for(let i=-1;i<=1;i+=2){
let l=new THREE.Mesh(
new THREE.BoxGeometry(.1,.01,5),
new THREE.MeshStandardMaterial({color:0xffffff})
)
l.position.set(x+i*1.3,.02,z)
scene.add(l)
}
}
for(let i=-3;i<=3;i++) slot(i*4,0)

/* CAR */
let car=new THREE.Group()
function part(w,h,d,c){
return new THREE.Mesh(
new THREE.BoxGeometry(w,h,d),
new THREE.MeshStandardMaterial({color:c})
)
}
let base=part(1.8,.5,3,0xff3333)
base.position.y=.3
car.add(base)
let roof=part(1.2,.4,1.4,0xaa0000)
roof.position.set(0,.65,-.3)
car.add(roof)
scene.add(car)

/* LEVELS */
let levels=[
{park:{x:4,z:0},cars:[[0,0],[8,0]]},
{park:{x:-4,z:4},cars:[[-8,4],[0,4]]},
{park:{x:0,z:-6},cars:[[4,-6],[-4,-6]]}
]

let lvl=0,zone,obstacles=[]

function loadLevel(){
document.getElementById("lvl").textContent=lvl+1
obstacles.forEach(o=>scene.remove(o))
obstacles=[]
if(zone)scene.remove(zone)

let p=levels[lvl].park
zone=new THREE.Mesh(
new THREE.BoxGeometry(2.5,.05,5),
new THREE.MeshStandardMaterial({color:0x00ff00})
)
zone.position.set(p.x,.03,p.z)
scene.add(zone)

levels[lvl].cars.forEach(c=>{
let o=part(1.8,.5,3,0x5555ff)
o.position.set(c[0],.25,c[1])
scene.add(o)
obstacles.push(o)
})

car.position.set(0,0,10)
car.rotation.y=Math.PI
}
loadLevel()

/* PHYSICS */
let speed=0,steer=0
let max=.18,acc=.003,fric=.985

let keys={}

/* KEYBOARD */
onkeydown=e=>keys[e.key]=true
onkeyup=e=>keys[e.key]=false

/* TOUCH */
function bind(btn,key){
let el=document.getElementById(btn)
el.ontouchstart=()=>keys[key]=true
el.ontouchend=()=>keys[key]=false
}

if(isMobile){
bind("gas","w")
bind("brake","s")
bind("left","a")
bind("right","d")
}

/* DRIVE */
function drive(){
if(keys["w"]) speed+=acc
if(keys["s"]) speed-=acc*1.3

speed*=fric
speed=Math.max(Math.min(speed,max),-max/2)

if(keys["a"]) steer+=.0012
if(keys["d"]) steer-=.0012
steer*=.9

car.rotation.y+=steer*speed*35
car.position.x-=Math.sin(car.rotation.y)*speed
car.position.z-=Math.cos(car.rotation.y)*speed
}

/* COLLISION */
function hit(a,b){
return Math.abs(a.x-b.x)<1.5 &&
Math.abs(a.z-b.z)<2.5
}

/* CHECK */
function check(){
if(hit(car.position,zone.position)){
lvl++
if(lvl>=levels.length){
alert("Tous les niveaux terminés.")
lvl=0
}
loadLevel()
}

obstacles.forEach(o=>{
if(hit(car.position,o.position)){
alert("Crash.")
loadLevel()
}
})
}

/* CAMERA */
function cam(){
let d=7
camera.position.x=car.position.x+Math.sin(car.rotation.y)*d
camera.position.z=car.position.z+Math.cos(car.rotation.y)*d
camera.position.y=3
camera.lookAt(car.position)
}

/* LOOP */
function animate(){
requestAnimationFrame(animate)
drive()
cam()
check()
renderer.render(scene,camera)
}
animate()

onresize=()=>{
camera.aspect=innerWidth/innerHeight
camera.updateProjectionMatrix()
renderer.setSize(innerWidth,innerHeight)
}
</script>
</body>
</html>
