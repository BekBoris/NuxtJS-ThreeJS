<template>
  <div class="mainContainer">
    <div ref="container" id="container"></div>

    <div>
      <video id="video" type="video/ogv" autoplay style="display:none">
        <source src="../static/Sintel.ogv" type="video/ogg" />
      </video>
    </div>

    <div class="inputselectfield">
      <p class="selectedGLTF">{{ this.gltf_clicked_name }}</p>

      <div>
        <input type="file" ref="file" @change="fileReader" />
      </div>
    </div>

    <div class="controls">
      <button type="button" name="translate" @click="controlSelect">
        Translate
      </button>
      <button type="button" name="rotate" @click="controlSelect">Rotate</button>
      <button type="button" name="scale" @click="controlSelect">Scale</button>
    </div>
  </div>
</template>

<script>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { TransformControls } from "three/examples/jsm/controls/TransformControls.js";

class Asset {
  constructor(name, meshes) {
    this.name = name;
    this.meshes = meshes;
  }
}

export default {
  name: "THREETest",
  data: function() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      gltf: null,
      orbit_controls: null,
      transform_controls: null,

      assets: [],
      gltf_clicked_name: ""
    };
  },
  mounted: function() {
    let container = this.$refs.container;

    // SET AND ADD CAMERA
    this.camera = new THREE.PerspectiveCamera(
      75,
      window.innerWidth / window.innerHeight,
      0.1,
      1000
    );
    this.camera.position.set(0, 2, 2);

    // ADD SCENE
    this.scene = new THREE.Scene();

    // ADD LIGHT TO SCENE
    this.light = new THREE.AmbientLight(0xffffff, 5.3);
    this.scene.add(this.light);

    // RENDER IN PAGE
    this.renderer = new THREE.WebGLRenderer({ antialias: true });
    this.renderer.setClearColor("#dbdbdb");
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.renderer.gammaFactor = 1.5;
    this.renderer.gammaOutput = true;
    container.appendChild(this.renderer.domElement);

    // SET GRID HELPER
    const size = 10;
    const divisions = 25;
    this.gridHelper = new THREE.GridHelper(size, divisions);
    this.scene.add(this.gridHelper);

    //SET PLANEGEOMETRY
    const video = document.getElementById("video");
    // video.load(); // must call after setting/changing source
    video.onloadeddata = function() {
      console.log("test");
      video.play();
    };
    const texture = new THREE.VideoTexture(video);
    texture.minFilter = THREE.LinearFilter;
    texture.magFilter = THREE.LinearFilter;
    texture.format = THREE.RGBFormat;
    texture.needsUpdate = true;

    const geometry = new THREE.PlaneGeometry(5, 2);
    const material = new THREE.MeshBasicMaterial({ map: texture });

    const plane = new THREE.Mesh(geometry, material);
    plane.material.side = THREE.DoubleSide;
    plane.position.y = 1;
    this.scene.add(plane);

    // SET ORBIT CONTROLS
    this.orbit_controls = new OrbitControls(
      this.camera,
      this.renderer.domElement
    );
    this.orbit_controls.addEventListener("change", () =>
      this.renderer.render(this.scene, this.camera)
    );

    // SET MODEL CONTROL
    this.transform_controls = new TransformControls(
      this.camera,
      this.renderer.domElement
    );
    this.scene.add(this.transform_controls);

    this.transform_controls.setSize(1);
    this.transform_controls.addEventListener("change", () => {
      this.renderer.render(this.scene, this.camera);
    });

    // EVENT LISTNER TO ENABLE ORBIT CAMERA AND CHANGE VALUE OF INPUTS
    this.transform_controls.addEventListener("dragging-changed", event => {
      this.orbit_controls.enabled = !event.value;
    });

    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();

    this.$refs.file.onclick = event => event.stopPropagation();

    const onMouseClick = event => {
      event.preventDefault();
      // calculate mouse position in normalized device coordinates
      // (-1 to +1) for both components
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      // update the picking ray with the camera and mouse position
      raycaster.setFromCamera(mouse, this.camera);

      this.assets.push(new Asset("Screen", plane));
      const meshes = this.assets
        .map(asset => {
          return asset.meshes;
        })
        .flat();

      // plane.raycast(raycaster, intersects);
      const intersects = raycaster.intersectObjects(meshes, true);
      if (intersects[0] !== undefined) {
        this.transform_controls.attach(intersects[0].object);
        intersects[0].object.traverseAncestors(anc => {
          if (anc.userData.asset !== undefined) {
            const model_name = anc.userData.asset.name;
            this.gltf_clicked_name = model_name;
          }
        });
      }
    };

    window.addEventListener("click", onMouseClick, false);
    this.animate();
    // this.renderer.render(this.scene, this.camera);
  },

  methods: {
    loadGLTF: function(gltf_array_buffer) {
      // ADD GLTFLOADER
      const loader = new GLTFLoader();
      loader.parse(
        gltf_array_buffer,
        "",
        gltf => {
          const name = prompt("Please Enter 3d Model Name");
          gltf.scenes.map(scene => {
            this.scene.add(scene);
            const asset = new Asset(name, gltf.scene.children);
            this.assets.push(asset);
            scene.children.forEach(mesh => {
              mesh.userData = { ...mesh.userData, asset };
              // mesh.userData = {
              //   asset: asset
              // };
            });
            // ADD CONTROLS TO MODEL

            this.transform_controls.attach(scene);
          });

          gltf.scene.scale.set(5, 5, 5);

          this.renderer.render(this.scene, this.camera);
        },
        error => {
          console.log(error);
        }
      );
    },

    fileReader: function(event) {
      const file = this.$refs.file.files[0];
      const readers = new FileReader();

      readers.addEventListener("loadend", e => {
        this.loadGLTF(e.target.result);
      });

      event.target.value = null;
      readers.readAsArrayBuffer(file);
    },

    controlSelect: function(event) {
      this.transform_controls.setMode(event.target.name);
    },

    animate: function() {
      requestAnimationFrame(this.animate);
      this.renderer.render(this.scene, this.camera);
    }
  }
};
</script>

<style>
#container {
  height: 100%;
}
* {
  margin: 0px;
  padding: 0px;
  box-sizing: border-box;
}
html,
body {
  height: 100%;
}
canvas {
  width: 100% !important;
  height: 100%;
}
.mainContainer {
  width: 100%;

  height: 100%;
  position: relative;
}
.inputselectfield {
  position: absolute;
  left: 10px;
  top: 10px;
}
.selectedGLTF {
  background-color: white;
  padding: 5px 0;
  margin-bottom: 10px;
  height: 30px;
  border-radius: 5px;
  text-align: center;
}
.controls {
  position: absolute;
  bottom: 50px;
  left: 50%;
  transform: translate(-50%);
}
</style>
