---
trigger: always_on
description: 3D graphics development with Three.js, React Three Fiber, and WebGL. Scene setup, performance optimization, 3D models, animations, and best practices.
---


# 3D Graphics Development

Complete guide for building 3D experiences in the browser using Three.js, React Three Fiber, and WebGL.

## CRITICAL: Container Dimensions

**NEVER forget container heights for 3D canvases.**

```jsx
// WRONG - Will cause rendering issues
<div>
  <Canvas>
    <Scene />
  </Canvas>
</div>

// CORRECT - Fixed height container REQUIRED
<div style={{ height: '500px', width: '100%' }}>
  <Canvas>
    <Scene />
  </Canvas>
</div>
```

```javascript
// Three.js - WRONG
renderer.setSize(window.innerWidth, window.innerHeight);

// Three.js - CORRECT with container
const container = document.getElementById('canvas-container');
renderer.setSize(container.clientWidth, container.clientHeight);
renderer.domElement.style.width = '100%';
renderer.domElement.style.height = '400px'; // REQUIRED
```

---

## Three.js Essentials

### Scene Setup

```javascript
import * as THREE from 'three';

// Create scene
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a1a);

// Create camera
const camera = new THREE.PerspectiveCamera(
  75, // FOV
  window.innerWidth / window.innerHeight, // Aspect ratio
  0.1, // Near plane
  1000 // Far plane
);
camera.position.set(0, 0, 5);

// Create renderer
const renderer = new THREE.WebGLRenderer({ 
  antialias: true, // Smooth edges
  alpha: true, // Transparent background
  powerPreference: 'high-performance'
});
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); // Performance
renderer.outputColorSpace = THREE.SRGBColorSpace;
renderer.toneMapping = THREE.ACESFilmicToneMapping;

// Add to DOM
document.body.appendChild(renderer.domElement);

// Animation loop
function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}
animate();

// Handle resize
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
```

### Basic Geometries

```javascript
// Box
const boxGeometry = new THREE.BoxGeometry(1, 1, 1);
const boxMaterial = new THREE.MeshStandardMaterial({ color: 0x0077ff });
const box = new THREE.Mesh(boxGeometry, boxMaterial);

// Sphere
const sphereGeometry = new THREE.SphereGeometry(1, 32, 32);
const sphere = new THREE.Mesh(sphereGeometry, material);

// Plane
const planeGeometry = new THREE.PlaneGeometry(10, 10);
const plane = new THREE.Mesh(planeGeometry, material);

// Custom geometry
const geometry = new THREE.BufferGeometry();
const vertices = new Float32Array([
  0, 0, 0,
  1, 0, 0,
  0, 1, 0
]);
geometry.setAttribute('position', new THREE.BufferAttribute(vertices, 3));
```

### Materials

```javascript
// Basic materials
const basicMat = new THREE.MeshBasicMaterial({ color: 0xff0000 });
const standardMat = new THREE.MeshStandardMaterial({
  color: 0x0077ff,
  metalness: 0.5,
  roughness: 0.5,
});
const physicalMat = new THREE.MeshPhysicalMaterial({
  color: 0xffffff,
  metalness: 0,
  roughness: 0,
  transmission: 1, // Glass-like
  thickness: 0.5,
});

// With textures
const textureLoader = new THREE.TextureLoader();
const texture = textureLoader.load('/textures/wood.jpg');
texture.wrapS = THREE.RepeatWrapping;
texture.wrapT = THREE.RepeatWrapping;
texture.repeat.set(2, 2);

const texturedMat = new THREE.MeshStandardMaterial({
  map: texture,
  normalMap: normalTexture,
  roughnessMap: roughnessTexture,
});
```

### Lighting

```javascript
// Ambient light - base illumination
const ambient = new THREE.AmbientLight(0xffffff, 0.5);

// Directional light - sun-like
const directional = new THREE.DirectionalLight(0xffffff, 1);
directional.position.set(5, 5, 5);
directional.castShadow = true;
directional.shadow.mapSize.width = 2048;
directional.shadow.mapSize.height = 2048;

// Point light - lightbulb
const point = new THREE.PointLight(0xff0000, 1, 100);
point.position.set(0, 0, 0);

// Spot light - focused beam
const spot = new THREE.SpotLight(0xffffff, 1);
spot.position.set(0, 10, 0);
spot.angle = Math.PI / 6;
spot.penumbra = 0.5;

// Hemisphere light - sky/ground
const hemisphere = new THREE.HemisphereLight(0xffffff, 0x444444, 0.5);

scene.add(ambient, directional, point, spot, hemisphere);
```

---

## React Three Fiber

### Canvas Setup

```jsx
import { Canvas } from '@react-three/fiber';
import { OrbitControls, PerspectiveCamera } from '@react-three/drei';

function Scene() {
  return (
    <div style={{ height: '500px', width: '100%' }}>
      <Canvas
        shadows
        dpr={[1, 2]} // Device pixel ratio
        gl={{ 
          antialias: true,
          alpha: true,
          powerPreference: 'high-performance'
        }}
      >
        <PerspectiveCamera makeDefault position={[0, 0, 5]} />
        <OrbitControls enableDamping dampingFactor={0.05} />
        
        <ambientLight intensity={0.5} />
        <directionalLight position={[5, 5, 5]} castShadow />
        
        <MeshComponent />
      </Canvas>
    </div>
  );
}
```

### Mesh Components

```jsx
import { useRef, useFrame } from 'react';
import { Mesh } from 'three';

function AnimatedBox() {
  const meshRef = useRef<Mesh>(null);
  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/advance-minimax-m2-cursor-rules](https://github.com/madebyaris/advance-minimax-m2-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
