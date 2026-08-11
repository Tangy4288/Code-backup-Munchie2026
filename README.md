<!DOCTYPE html>
<html lang="en">
<head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Aladin&display=swap" rel="stylesheet">
  <title>Dimen</title>
  <style>
    body {
      z-index: 1;
      margin: 0;
      overflow: hidden;
      background-color: black;
      
    }

#textBox {
  margin-top: 4px;
  width: 75%;
  height: 25%;
  position: fixed;
  opacity: 0;
  position: fixed;
  left: 50%;
  transform: translateX(-50%);
  font-size: 78px;
  font-family: "Aladin", system-ui;
  font-weight: 400;
  font-style: normal;
  text-shadow: 2px 2px 2px black;
  color: white;
  background-color:black;
  border-radius: 14px;
  z-index: 3;
  transition: opacity 1s ease;
}  

#textBoxOut {
  width: 76%;
  height: 26%;
  position: fixed;
  opacity: 0;
  position: fixed;
  left: 50%;
  transform: translateX(-50%);
  font-size: 78px;
  font-family: "Aladin", system-ui;
  font-weight: 400;
  font-style: normal;
  text-shadow: 2px 2px 2px black;
  color: white;
  background-color:white;
  border-radius: 14px;
  z-index: 2;
  transition: opacity 1s ease;
}  

#ShopKeeperName {
  transition: opacity 1s ease;
}

#startBtn {
    z-index: 11;
}


#gameTitle,
#controls {
  position: fixed;
  z-index: 10;
}
  </style>
</head>
<body>




<h1 id="gameTitle" style="
  position: fixed;
  left: 50%;
  transform: translateX(-50%);
  font-size: 78px;
  font-family: Aladin, system-ui;
  font-weight: 400;
  font-style: normal;
  text-shadow: 2px 2px 2px black;
  color: white;
  z-index: 9998;
  ">Dimen</h1>

<h1 id="ShopKeeperName" style="
  position: fixed;
  margin-bottom:6px;
  left: 25%;
  transform: translateX(-50%);
  font-size: 35px;
  font-family: Aladin, system-ui;
  font-weight: 400;
  font-style: normal;
  text-shadow: 2px 2px 2px black;
  color: white;
  z-index: 9998;
  opacity: 0;
  ">Samje</h1>

  <br>
<p id="ShopDialogue1" style="
  position: fixed;
  padding-top: 20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 25px;
  font-family: Aladin, system-ui;
  font-weight: 400;
  font-style: normal;
  text-shadow: 2px 2px 2px black;
  color: white;
  z-index: 9998;
  opacity: 0;
  ">Welcome to Dimen. I am the shopkeeper. Have this to get you started.</p>
  
  <p id="ShopDialogue2" style="
  position: fixed;
  padding-top: 20px;
  left: 50%;
  transform: translateX(-50%);
  font-size: 25px;
  font-family: Aladin, system-ui;
  font-weight: 400;
  font-style: normal;
  text-shadow: 2px 2px 2px black;
  color: white;
  z-index: 9998;
  opacity: 0;
  ">Have this to get you started.</p>
  
<button id="startBtn" style="
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  font-family: Aladin, system-ui;
  font-weight: 400;
  font-style: normal;

  font-size: 24px;
  padding: 10px 20px;
  z-index: 9999;
">Start Game</button>




<div id="textBox"></div>
<div id="textBoxOut"></div>
  
<ul id="controls" style="
  list-style-type: none;
  position: fixed;
  font-family: Aladin, system-ui;
  font-weight: 400;
  font-style: normal;
  color: white;
  font-size: 24px;">
  <li>X = crouch</li>
  <li>E = interact</li>
  <li>WASD = move</li>
</ul>
<script type="importmap">
  {
    "imports": {
      "three": "https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.module.js",
      "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.152.2/examples/jsm/",
      "cannon-es": "https://cdn.jsdelivr.net/npm/cannon-es@0.20.0/dist/cannon-es.js"
    }
  }
</script>

<script type="module">

import * as THREE from 'three';
import * as CANNON from 'cannon-es';
import { PointerLockControls } from 'three/addons/controls/PointerLockControls.js';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import {OrderedDitherPass } from 'https://cdn.jsdelivr.net/gh/samwhitford/threejs-ordered-dithering-effect@main/OrderedDitherPass.js';
import { DotScreenPass } from 'three/addons/postprocessing/DotScreenPass.js';

const orderedDitherEffect = new OrderedDitherPass(8, 2);



let click = false;

// Scene setup
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x000000);
const velocity = new THREE.Vector3();
let speed = 5.0; // units per second
const clock = new THREE.Clock();

const idelAmp = 0.1;
const idelSpeed = 0.5;
const initY = 1;

const cubeOutlines = [];
const wallOutlines = [];

let gameStarted = false;

let topBlocked = false;

let idleH = 0;

let CW = 5;

let Shop = false;
let ShopDialogue1 = false;
let ShopDialogue2 = false;

let selectedWaterBody = null;

let item = "none";

const waterBodies = [];
const characters = [];
const cubes = [];
const cubeBodies = [];

//Fsihing Rod
const RodGeometry = new THREE.CylinderGeometry(0.1,0.1,4,8,1);
const RodMaterial = new THREE.MeshBasicMaterial({ color: 0x000000, depthTest: false });
const FishingRod = new THREE.Mesh(RodGeometry, RodMaterial);
FishingRod.renderOrder = 1000;
const RodoutlineMaterial = new THREE.MeshBasicMaterial({
    color: 0xFFFFFF,
    side: THREE.BackSide,
    
  });

  const RodoutlineMesh = new THREE.Mesh(FishingRod.geometry.clone(), RodoutlineMaterial);
  RodoutlineMesh.scale.multiplyScalar(1.05); // Slightly bigger than original
  RodoutlineMesh.material.depthTest = false;
  RodoutlineMesh.renderOrder = 999;
  FishingRod.add(RodoutlineMesh); // Add as child to sync position/quaternion automatically

//Fish
// CapsuleGeometry(radius, length, capSegments, radialSegments)
const FishGeometry = new THREE.SphereGeometry(0.5,12, 12);
const FishMaterial = new THREE.MeshBasicMaterial({ color: 0x000000, depthTest: false });
const Fish = new THREE.Mesh(FishGeometry, FishMaterial);
Fish.renderOrder = 1000;
const FishoutlineMaterial = new THREE.MeshBasicMaterial({
    color: 0xFFFFFF,
    side: THREE.BackSide,
    
  });

  const FishoutlineMesh = new THREE.Mesh(Fish.geometry.clone(), FishoutlineMaterial);
  FishoutlineMesh.scale.multiplyScalar(1.05); // Slightly bigger than original
  FishoutlineMesh.material.depthTest = false;
  FishoutlineMesh.renderOrder = 999;
  Fish.add(FishoutlineMesh); // Add as child to sync position/quaternion automatically

let crouch = false;
let cameraCurrentOffsetY = 1.2; // Start at standing height

let NPCinteract = false;

let onGround = false;
let jumping = false;
let isGrabbing = false;
let grabbedObject = null;

let interact = false;

let Escape = false;

const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();
let selectedObject = null;
let selectedNPC = null;
const physicsWorld = new CANNON.World({
    gravity: new CANNON.Vec3(0, -9.82, 0),
});



// Physics World
const frictionMaterial = new CANNON.Material('frictionMaterial');
frictionMaterial.friction = 0.9;

const contactMaterial = new CANNON.ContactMaterial(
  frictionMaterial, // material A
  frictionMaterial, // material B
  {
    friction: 0.9,
    restitution: 0.1  // how bouncy it is
  }
);
physicsWorld.addContactMaterial(contactMaterial);

const terrainMaterial = new CANNON.Material("terrainMaterial");

const TcontactMaterial = new CANNON.ContactMaterial(
    frictionMaterial,
    terrainMaterial,
    {
        friction: 1,
        restitution: 0.0
    }
);
physicsWorld.addContactMaterial(TcontactMaterial);

// Camera setup
const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(20, 2, 0);
camera.lookAt(0, 0, 0);
scene.add(camera);

FishingRod.rotation.x = Math.PI / 1.5;
camera.add(FishingRod);

Fish.rotation.x = Math.PI / 2;
camera.add(Fish);

// Background Camera
const startCamera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
startCamera.position.set(0, 10, 0);
startCamera.lookAt(0, 0, 0);
let HeightState = 0.8;
// Player physics body
const playerShapeCurrent = new CANNON.Sphere(0.8);
let playerBody = new CANNON.Body({
    mass: 4000,
  position: new CANNON.Vec3(camera.position.x, camera.position.y, camera.position.z),
  shape: playerShapeCurrent,
  linearDamping: 0.9, // Helps stop sliding
  collisionFilterGroup: 1,
  material:frictionMaterial
});
physicsWorld.addBody(playerBody);

// Track collision contact normals to detect ground contact
playerBody.addEventListener('collide', function (event) {
  const contact = event.contact;

  const contactNormal = new CANNON.Vec3();
  if (contact.bi.id === playerBody.id) {
    contact.ni.negate(contactNormal);
  } else {
    contactNormal.copy(contact.ni);
  }

  if (contactNormal.y > 0.5) {
    onGround = true;
  }
});

// Renderer
const renderer = new THREE.WebGLRenderer({ antialias: false });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
renderer.setPixelRatio(1);
document.body.appendChild(renderer.domElement);

renderer.domElement.style.position = 'fixed';
renderer.domElement.style.top = '0';
renderer.domElement.style.left = '0';
renderer.domElement.style.zIndex = '0';

//renderer.toneMapping = THREE.ReinhardToneMapping;
//renderer.toneMappingExposure = 1.2;
//renderer.outputColorSpace = THREE.SRGBColorSpace;

const dotScreenPass = new DotScreenPass(
  new THREE.Vector2(window.innerWidth / 2, window.innerHeight / 2), // center
  0.5,  // angle in radians
  3.0   // scale (dot size)
);




const composer = new EffectComposer(renderer);



// 1. Normal scene render
const renderPass = new RenderPass(scene, camera);

composer.addPass(renderPass);

//composer.addPass(dotScreenPass);

composer.addPass(orderedDitherEffect);



// Create outline using edges

const lineMat = new THREE.LineBasicMaterial({ color: 0x000000 });












// Light

//const Rl1wgeometry = new THREE.CylinderGeometry(2,2,4,8,1);
//const Rl1wmaterial = new THREE.MeshBasicMaterial({ color: 0xddd5c4 });
//const Rl1wmesh = new THREE.Mesh(Rl1geometry, Rl1material);

const Rl1geometry = new THREE.SphereGeometry(0.5, 16, 16);
const Rl1material = new THREE.MeshBasicMaterial({ color: 0xddd5c4 });
const Rl1mesh = new THREE.Mesh(Rl1geometry, Rl1material);
Rl1mesh.position.set(3, 22, 0);
scene.add(Rl1mesh);

const roomLight1 = new THREE.SpotLight(0xDDD5C4, 0.7, 30, Math.PI / 6, 0.1, 0);

roomLight1.position.set(3, 22, 0)

roomLight1.target.position.set(3, 0, 0)

const passiveLightE = new THREE.AmbientLight(0xFFF8E7, 0.25);

const passiveLightG = new THREE.PointLight(0xCCCCCC, 0.45, 35, 1.2);

passiveLightG.castShadow = true;

scene.add(passiveLightE);
scene.add(passiveLightG);
scene.add(roomLight1);
scene.add(roomLight1.target);

// Street 
const size = 400;
const segments = 2;

const Geometry = new THREE.PlaneGeometry(size*5, size*2.5, segments, segments);
Geometry.rotateX(-Math.PI / 2);

Geometry.computeVertexNormals();

const material = new THREE.MeshStandardMaterial({ color: 0x9a9a9a });
const Street = new THREE.Mesh(Geometry, material);
Street.receiveShadow = true;

scene.add(Street);


// Street physics
const StreetBody = new CANNON.Body({
    type: CANNON.Body.STATIC, collisionFilterGroup: 1, collisionFilterMask: 2 | 1
});
StreetBody.addShape(new CANNON.Plane());
StreetBody.quaternion.setFromEuler(-Math.PI / 2, 0, 0);
physicsWorld.addBody(StreetBody);

// Window Resize
window.addEventListener('resize', () => {
  renderer.setSize(window.innerWidth, window.innerHeight);
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
});

// Controls
const controls = new PointerLockControls(camera, document.body);
scene.add(controls.getObject());

document.addEventListener('click', () => {
  controls.lock(); // request pointer lock
  click = true;
});

let TotalFish = 0;

let Fishinteract = false;

let NoMove = false;

let Skip = false;

let SHY = 1000;
const move = { forward: false, backward: false, left: false, right: false };
let inventory = false;
let makeCube = false;
const radialSeg = 20;
const capSeg = 10;
const heightSeg = 1;
//Shop Keeper
const SHgeometry = new THREE.CapsuleGeometry(0.8, 1.5, radialSeg, capSeg, heightSeg);
  const SHmaterial = new THREE.MeshStandardMaterial({ color: 0xdddddd });
  const SHmesh = new THREE.Mesh(SHgeometry, SHmaterial);
  SHmesh.position.set(4.5,SHY,0);
  SHmesh.userData.id = "shopKeeper"
  scene.add(SHmesh);
  characters.push(SHmesh);


  const SHoutlineMaterial = new THREE.MeshBasicMaterial({
    color: 0x000000,
    side: THREE.BackSide,
  });

  const SHoutlineMesh = new THREE.Mesh(SHmesh.geometry.clone(), SHoutlineMaterial);
  SHoutlineMesh.scale.multiplyScalar(1.05); // Slightly bigger than original
  SHmesh.add(SHoutlineMesh); // Add as child to sync position/quaternion automatically
  SHoutlineMesh.userData.id = "shopKeeper"

  const SHHgeometry = new THREE.SphereGeometry(0.5, 16, 16);
  const SHHmaterial = new THREE.MeshStandardMaterial({ color: 0xdddddd });
  const SHHmesh = new THREE.Mesh(SHHgeometry, SHHmaterial);
  SHHmesh.position.set(4.5,SHY+2,0)
  SHHmesh.userData.id = "shopKeeper"
  scene.add(SHHmesh);
  characters.push(SHHmesh); // optional if head should be selectable

  const SHHoutlineMaterial = new THREE.MeshBasicMaterial({
    color: 0x000000,
    side: THREE.BackSide,
  });

  const SHHoutlineMesh = new THREE.Mesh(SHHmesh.geometry.clone(), SHHoutlineMaterial);
  SHHoutlineMesh.scale.multiplyScalar(1.05); // Slightly bigger than original
  SHHmesh.add(SHHoutlineMesh); // Add as child to sync position/quaternion automatically
  SHHoutlineMesh.userData.id = "shopKeeper"

document.addEventListener('keydown', (e) => {
    if (e.code === 'Escape') {
      Escape = true;
    }
    if (e.code === 'KeyE') {
        
        if (!isGrabbing && selectedObject) {
            const distance = camera.position.distanceTo(selectedObject.position);
            if (distance < 5) {
                isGrabbing = true;
                grabbedObject = selectedObject;
                
            }
        } else if (isGrabbing) {
            isGrabbing = false;
            grabbedObject = null;
        }
        
        if (NPCinteract && selectedNPC) {
          NPCinteract = false;
        }

        if (Fishinteract && selectedWaterBody) {
          Fishinteract = false;
        }
          
        if (!NPCinteract && selectedNPC) {
          NPCinteract = true;
        }

        if (!Fishinteract && selectedWaterBody) {
          Fishinteract = true;
        }
        
    }
    
    if (e.code === 'KeyC' && FishHeld) {
        item = "fish";
    } 
    if (e.code === 'KeyF') {
        item = "fishingRod";
    } 
    if (e.code === 'KeyR') {
        item = "none";
    }
    if (e.code === 'KeyW') move.forward = true;
    if (e.code === 'KeyS') move.backward = true;
    if (e.code === 'KeyA') move.left = true;
    if (e.code === 'KeyD') move.right = true;
    //if (e.code === 'KeyC') makeCube = true;
    if (e.code === 'KeyX' && !crouch) {
      crouch = true;
    } else if (e.code === 'KeyX' && crouch) {
      crouch = false;
    }
    
    if (e.code === 'Space') {
        Skip = true;
}

});
document.addEventListener('keyup', (e) => {
    if (e.code === 'KeyW') move.forward = false;
    if (e.code === 'KeyS') move.backward = false;
    if (e.code === 'KeyA') move.left = false;
    if (e.code === 'KeyD') move.right = false;
    //if (e.code === 'KeyC') makeCube = false;
    if (e.code === 'Escape') Escape = false;
});
document.getElementById('startBtn').addEventListener('click', () => {
  gameStarted = true;
  document.getElementById('startBtn').style.display = 'none';
  document.getElementById('gameTitle').style.display = 'none';
  document.getElementById('controls').style.display = 'none';
  controls.lock();
});

const outlineMaterial = new THREE.MeshBasicMaterial({
    color: 0x000000,
    side: THREE.BackSide,
  });



function onMouseMotion(event) {
    pointer.x = ((event.clientX / window.innerWidth) * 2 - 1);
    pointer.y = -((event.clientY / window.innerHeight) * 2 + 1);
};
document.addEventListener('mousemove', onMouseMotion);

function createCubeOutlineMesh(originalMesh, scale=1.05) {
  const outlineMaterial = new THREE.MeshBasicMaterial({
    color: 0x000000,
    side: THREE.BackSide,
  });
  const outlineMesh = new THREE.Mesh(originalMesh.geometry.clone(), outlineMaterial);
  outlineMesh.scale.multiplyScalar(1.05); // Slightly bigger than original
  originalMesh.add(outlineMesh); // Add as child to sync position/quaternion automatically
}

function createWallOutlineMesh(originalMesh) {
  const outlineMaterial = new THREE.MeshBasicMaterial({
    color: 0x000000,
    side: THREE.BackSide,
  });
  const outlineMesh = new THREE.Mesh(originalMesh.geometry.clone(), outlineMaterial);
  outlineMesh.castShadow = false;
  outlineMesh.receiveShadow = false;
  outlineMesh.scale.multiplyScalar(1.035); // Slightly bigger than original
  originalMesh.add(outlineMesh); // Add as child to sync position/quaternion automatically
}

function createCube(x, y, z) {
  const size = 1;
  const geometry = new THREE.BoxGeometry(size, size, size);
  const material = new THREE.MeshStandardMaterial({ color: 0xdddddd });
  const mesh = new THREE.Mesh(geometry, material);
  mesh.castShadow = true;
  mesh.receiveShadow = true;
  scene.add(mesh);
  cubes.push(mesh);
  
  
  createCubeOutlineMesh(mesh);
  
  
  const body = new CANNON.Body({ 
    mass: 500,
    material: frictionMaterial,
    collisionFilterGroup: 2, collisionFilterMask: 3 | 2 | 1
  });
  body.addShape(new CANNON.Box(new CANNON.Vec3(size / 1.9, size / 1.9, size / 1.9)));
  body.position.set(x, y+2, z);
  physicsWorld.addBody(body);
  cubeBodies.push(body);
}
  
function createWall(x, y, z, width = 6, height = 6, depth = 2, rotationx = 0, rotationy = 0, rotationz = 0, color = 0x888888, collisionFilterGroup = 1) {
  // Three.js mesh
  const geometry = new THREE.BoxGeometry(width, height, depth);
  const material = new THREE.MeshBasicMaterial({ color: color });
  const mesh = new THREE.Mesh(geometry, material);
  mesh.position.set(x, y, z);
  mesh.rotation.y = THREE.MathUtils.degToRad(rotationy);
  mesh.rotation.x = THREE.MathUtils.degToRad(rotationx);
  mesh.rotation.z = THREE.MathUtils.degToRad(rotationz);
  scene.add(mesh);
  
  createWallOutlineMesh(mesh);
  

  
  // Cannon.js body
  const shape = new CANNON.Box(new CANNON.Vec3(width / 2, height / 2, depth / 2));
  const body = new CANNON.Body({
    mass: 0, // Static wall
    position: new CANNON.Vec3(mesh.position.x, mesh.position.y, mesh.position.z),
    shape: shape,
    collisionFilterGroup: collisionFilterGroup, 
  });
  
  physicsWorld.addBody(body);

  return { mesh, body }; // ? this is correctly inside the function
}

function createDWall(x, y, z, width = 6, height = 6, depth = 2) {
  const geometry = new THREE.BoxGeometry(width, height, depth);
  const material = new THREE.MeshLambertMaterial({ color: 0x666666 });
  const mesh = new THREE.Mesh(geometry, material);
  mesh.position.set(x, y, z);
 scene.add(mesh);
 
  
  createWallOutlineMesh(mesh);

  
  // Cannon.js body
  const shape = new CANNON.Box(new CANNON.Vec3(width / 2, height / 2, depth / 2));
  const body = new CANNON.Body({
    mass: 0, // Static wall
    position: new CANNON.Vec3(x, y, z),
    shape: shape,
    collisionFilterGroup: 1, 
  });
  physicsWorld.addBody(body);

  return { mesh, body }; // ? this is correctly inside the function
}

const groundRayLength = 1.1; // slightly bigger than player radius


function checkGrounded() {
  const rayFrom = new CANNON.Vec3(
    playerBody.position.x,
    playerBody.position.y,
    playerBody.position.z
  );

  const rayTo = new CANNON.Vec3(
    playerBody.position.x,
    playerBody.position.y - groundRayLength,
    playerBody.position.z
  );

  const rayResult = new CANNON.RaycastResult();

  onGround = false;

  physicsWorld.raycastClosest(
    rayFrom,
    rayTo,
    {
      skipBackfaces: true,
      collisionFilterMask: -1
    },
    rayResult
  );

  if (rayResult.hasHit) {
    // Optional: check surface normal so walls don't count
    if (rayResult.hitNormalWorld.y > 0.5) {
      onGround = true;
    }
  }
}

function createWaterBody(x,z) {
  //button 
  const geometry = new THREE.CylinderGeometry(2,2,4,8,1);
  geometry.rotateY(-Math.PI / 2);
  const material = new THREE.MeshLambertMaterial({ color: 0xaaaaaa });
  const circle = new THREE.Mesh(geometry, material);
  circle.position.set(x,-1.9,z)
  scene.add(circle);
  waterBodies.push(circle);
  // Parameters: radiusTop, radiusBottom, height, numSegments
  const WcylinderShape = new CANNON.Cylinder(2,2, 4,8, 1); // Create a body and add the shape 
  const WcylinderBody = new CANNON.Body({ mass: 10, collisionFilterGroup: 3, collisionFilterMask: 1 | 2, material: frictionMaterial, linearDamping: 0.5  }); 
  WcylinderBody.addShape(WcylinderShape); // Position it in the world 
  WcylinderBody.position.set(x, -1.8, z); // Add to physics world 
  

  
  
  const WcoutlineMesh = new THREE.Mesh(circle.geometry.clone(), outlineMaterial);
  WcoutlineMesh.scale.multiplyScalar(1.021); // Slightly bigger than original
  circle.add(WcoutlineMesh); // Add as child to sync position/quaternion automatically
  
  WcylinderBody.position.copy(circle.position);
  
  physicsWorld.addBody(WcylinderBody);
  
  //button base
  
  const bgeometry = new THREE.CylinderGeometry(1.5,1.5,4,8,1);
  bgeometry.rotateY(-Math.PI / 2);
  const bmaterial = new THREE.MeshLambertMaterial({ color: 0x777777 });
  const bcircle = new THREE.Mesh(bgeometry, bmaterial);
  bcircle.position.set(x,-1.8,z)
  scene.add(bcircle);
  waterBodies.push(bcircle);
  // Parameters: radiusTop, radiusBottom, height, numSegments
  const bWcylinderShape = new CANNON.Cylinder(2,2, 4,8, 1); // Create a body and add the shape 
  const bWcylinderBody = new CANNON.Body({ mass: 0, collisionFilterGroup: 2}); 
  bWcylinderBody.addShape(WcylinderShape); // Position it in the world 
  bWcylinderBody.position.set(x, -1.7, z); // Add to physics world 
  
  const bWoutlineMesh = new THREE.Mesh(bcircle.geometry.clone(), outlineMaterial);
  bWoutlineMesh.scale.multiplyScalar(1.028); // Slightly bigger than original
  bcircle.add(bWoutlineMesh); // Add as child to sync position/quaternion automatically
  
  physicsWorld.addBody(bWcylinderBody);
}


function createBuildBase(x, y, z, Lwall = false, Rwall = false, Bwall = false, counter = false) {

createWall(x, 0, -1*z, 0.5, 15, 0.5,0,0,0, 0x666666); // Closest leg

createWall(x, 0, z, 0.5, 15, 0.5,0,0,0, 0x666666); // Right Leg

createWall(-1*x, 0, z, 0.5, 15, 0.5,0,0,0, 0x666666); // Further right leg

createWall(-1*x, 0, -1*z, 0.5, 15, 0.5,0,0,0, 0x666666); // Farthest left leg

if (Bwall) {
    createWall(-1*x-0.5, 7, -1*z+z, 0.5, 1.5, 2*z+1.5);
    
    createWall(-1*x-0.5, 5, -1*z+z, 0.5, 1.5, 2*z+1.5);
    
    createWall(-1*x-0.5, 3, -1*z+z, 0.5, 1.5, 2*z+1.5);
    
    createWall(-1*x-0.5, 1, -1*z+z, 0.5, 1.5, 2*z+1.5);
}

if (Rwall) {
    createWall(-1*x+x, 7, -1*z-0.5, 2*x+1.5, 1.5, 0.5);
    
    createWall(-1*x+x, 5, -1*z-0.5, 2*x+1.5, 1.5, 0.5);
    
    createWall(-1*x+x, 3, -1*z-0.5, 2*x+1.5, 1.5, 0.5);
    
    createWall(-1*x+x, 1, -1*z-0.5, 2*x+1.5, 1.5, 0.5);
}

if (Lwall) {
    createWall(-1*x+x, 7, -1*z+2*z+0.5, 2*x+1.5, 1.5, 0.5);
    
    createWall(-1*x+x, 5, -1*z+2*z+0.5, 2*x+1.5, 1.5, 0.5);
    
    createWall(-1*x+x, 3, -1*z+2*z+0.5, 2*x+1.5, 1.5, 0.5);
    
    createWall(-1*x+x, 1, -1*z+2*z+0.5, 2*x+1.5, 1.5, 0.5);
}


if (counter) {
    createWall(-1*x+2*x, 0.65, -1*z+z, 1, 1.5, 2*z+1.5, 0,0,-14);
}

}

//createBuildBase(7,0,10, true, true, true, true);

createWaterBody(20, 10)

createCube(3, 20, 0);
createCube(3, 25, 0);

//createCube(4, 20, 0);
//createCube(4, 25, 0);

//createCube(5, 20, 0);
//createCube(5, 25, 0);

//createCube(6, 20, 0);
//createCube(6, 25, 0);

const fixedTimeStep = 1 / 60;
const maxSubSteps = 3;


const moveDirection = new THREE.Vector3();

let RodIdle;
let NoItems = true;
let FishHeld = false;


function animate() {
    requestAnimationFrame(animate);
  if (!gameStarted) {
      return;
    }
    
  let buttonActivated = false;
  
  velocity.set(0, 0, 0);
  const delta = clock.getDelta();
  physicsWorld.step(fixedTimeStep, delta, maxSubSteps);

  passiveLightG.position.set(playerBody.position.x, playerBody.position.y+1.6, playerBody.position.z);
  
  checkGrounded();
  
  const elapsedTime = clock.getElapsedTime();
  
  //WcylinderBody.position.copy(circle.position);
  
  // Sync cube meshes with physics bodies
  for (let i = 0; i < cubes.length; i++) {
    cubes[i].position.copy(cubeBodies[i].position);
    cubes[i].quaternion.copy(cubeBodies[i].quaternion);
  }
  
 // circle.position.copy(WcylinderBody.position);
  
  //SHY = Math.sin(elapsedTime)*0.05;
  // Movement logic
  RodIdle = Math.sin(elapsedTime*1)*0.05;
  SHmesh.position.set(4.5,SHY,0)
SHHmesh.position.set(4.5,SHY+2,0)
  camera.getWorldDirection(moveDirection);

  // Flatten vertical look direction for movement
  moveDirection.y = 0;
  moveDirection.normalize();

  const inputVelocity = new CANNON.Vec3(0, 0, 0);
  
  if (item === "fishingRod") {
    FishingRod.position.set(0.5, RodIdle, -2);
    Fish.position.set(0.5, 100, -2);
  } 
  if (item === "none" || NoItems) {
      FishingRod.position.set(0.5, 100, -2);
      Fish.position.set(0.5, 100, -2);
  }
  
  if (item === "fish") {
      FishingRod.rotation.x = Math.PI / 1.5;
      Fish.position.set(0.5, RodIdle, -2);
      FishingRod.position.set(0.5, 100, -2);
  }
  
  
  let IdleCameraOffsetY;
  
  if (!NoMove) {
  if (move.forward) {
    inputVelocity.x += moveDirection.x * speed;
    inputVelocity.z += moveDirection.z * speed;
    
  }
  
  if (move.backward) {
    inputVelocity.x -= moveDirection.x * speed;
    inputVelocity.z -= moveDirection.z * speed;
  }
  if (move.left) {
    const left = new THREE.Vector3().crossVectors(camera.up, moveDirection).normalize();
    inputVelocity.x += left.x * speed;
    inputVelocity.z += left.z * speed;
  }
  if (move.right) {
    const right = new THREE.Vector3().crossVectors(moveDirection, camera.up).normalize();
    inputVelocity.x += right.x * speed;
    inputVelocity.z += right.z * speed;
  }
  
  }


  if (makeCube) {
    const direction = new THREE.Vector3();
    camera.getWorldDirection(direction); // Get where the player is looking
    direction.normalize();

    const distance = 3; // How far in front of player to place the cube
    const spawnPosition = new THREE.Vector3().copy(camera.position).add(direction.multiplyScalar(distance));

    createCube(spawnPosition.x, spawnPosition.y, spawnPosition.z);
    makeCube = false;
} 

  
  
  playerBody.velocity.x = inputVelocity.x;
  playerBody.velocity.z = inputVelocity.z;

  // Grabbing logic
  if (isGrabbing && grabbedObject) {
    const index = cubes.indexOf(grabbedObject);
    if (index !== -1) {
      const grabbedBody = cubeBodies[index];
      const grabDistance = 2;
      const direction = new THREE.Vector3();
      camera.getWorldDirection(direction);
      const targetPos = camera.position.clone().add(direction.multiplyScalar(grabDistance));

      const current = grabbedBody.position;
      const target = new CANNON.Vec3(targetPos.x, targetPos.y, targetPos.z);

      const force = target.vsub(current).scale(10);
      grabbedBody.velocity.set(force.x, force.y, force.z);
      grabbedBody.angularVelocity.set(0, 0, 0);
      
      const cube = cubes[index];
      cube.quaternion.copy(camera.quaternion);
      grabbedBody.quaternion.copy(camera.quaternion); // For physics rotation too
      
    }
  }


  // Raycasting for selection
  const centerX = window.innerWidth / 2;
  const centerY = window.innerHeight / 2;
  const Tcenter = new THREE.Vector2(
    (centerX / window.innerWidth) * 2 - 1,
    -(centerY / window.innerHeight) * 2 + 1
  );
  const objects = cubes // use concat method for other items
  raycaster.setFromCamera(Tcenter, camera);
  const intersects = raycaster.intersectObjects(objects);
  const NPC = raycaster.intersectObjects(characters);
  const Fishing = raycaster.intersectObjects(waterBodies);



  if (intersects.length > 0) {
    selectedObject = intersects[0].object;
  } else {
    selectedObject = null;
  }
  
  if (NPC.length > 0) {
    selectedNPC = NPC[0].object;
  } else {
    selectedNPC = null;
  }

  if (Fishing.length > 0) {
    selectedWaterBody = Fishing[0].object;
  } else {
    selectedWaterBody = null;
  }
if (selectedWaterBody && Fishinteract && item === "fishingRod") { 
    if (!FishHeld) {
    
    FishingRod.rotation.x = Math.PI / 1.3; 
    crouch = true; 
    NoMove = true;  
     setTimeout(() => { 
         item = "fish";
         FishHeld = true;
         Fishinteract = false;
         NoMove = false;
         crouch = false;
         }, 7000);
    }
}
  if (selectedObject) {
    const pos = selectedObject.position.clone();
    pos.y += 1;

    pos.project(camera);
    const x = (pos.x * 0.5 + 0.5) * window.innerWidth;
    const y = (-pos.y * 0.5 + 0.5) * window.innerHeight;

  }

  

  if (selectedNPC && NPCinteract) {

    if (NPC[0].object.userData.id === "shopKeeper") {
        ShopDialogue1 = true;
        
      } 
    } 
  if (!selectedNPC) {
      ShopDialogue1 = false;
      NPCinteract = false;
  }
   if (ShopDialogue1) {
    document.getElementById('textBox').style.opacity = '1';
    document.getElementById('textBoxOut').style.opacity = '1';
    document.getElementById('ShopKeeperName').style.opacity = '1';
    document.getElementById('ShopDialogue1').style.opacity = '1';
    item = "fishingRod";
    NoItems = false;
   }
   if (!ShopDialogue1) {
    document.getElementById('textBox').style.opacity = '0';
    document.getElementById('textBoxOut').style.opacity = '0';
    document.getElementById('ShopKeeperName').style.opacity = '0';
    document.getElementById('ShopDialogue1').style.opacity = '0';
   }


  // Camera follows player body with offset
  let crouchOffset;
  let targetCameraOffsetY;
  

  if (crouch) {
    targetCameraOffsetY = 0.5;
    IdleCameraOffsetY = 0;
  } 
  if (!crouch) {
    targetCameraOffsetY = 1.2;
    IdleCameraOffsetY = Math.sin(elapsedTime*0.75)*0.05;
  }
  
  cameraCurrentOffsetY = THREE.MathUtils.lerp(cameraCurrentOffsetY, targetCameraOffsetY, 0.1);
  camera.position.set(
    playerBody.position.x,
    playerBody.position.y + cameraCurrentOffsetY + IdleCameraOffsetY,
    playerBody.position.z
  );
  
 
  
  
  composer.render(scene, camera);
};

animate();

</script>
</body>
</html>
