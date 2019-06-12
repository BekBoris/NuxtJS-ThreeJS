
<template>
  <div class="mainContainer">
    <div ref="container" id="container"></div>
    <div class="inputselectfield">
    <div>
      <p>Position</p>
      <input type="number" name="position" v-model="location_model.position.x" />
      <input type="number" name="position" v-model="location_model.position.y" />
      <input type="number" name="position" v-model="location_model.position.z" />
    </div>
    <div>
      <p>Position</p>
      <input type="number" name="rotation" v-model="location_model.rotation.x" />
      <input type="number" name="rotation" v-model="location_model.rotation.y" />
      <input type="number" name="rotation" v-model="location_model.rotation.z" />
    </div>
    <div>
      <p>Position</p>
      <input type="number" name="scale" v-model="location_model.scale.x" />
      <input type="number" name="scale" v-model="location_model.scale.y" />
      <input type="number" name="scale" v-model="location_model.scale.z" />
    </div>

      <div>
        <p>Position Rotation Scaled Selector</p>
        <select v-model="selectedtool">
          <option v-for="option in options" v-bind:value="option.value">{{option.text}}</option>
        </select>
      </div>

    </div>
  </div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { TransformControls } from 'three/examples/jsm/controls/TransformControls.js';


export default {
  name: 'THREETest',
  data: function() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      gltf: null,
      orbit: null,
      control: null,
      selectedtool: "translate",
      options: [ {text: "position", value: "translate"},
                {text: "rotation", value: "rotate"},
                {text: "scale", value: "scale"}
              ],

      location_model: { position: {x: 0, y: 0, z: 0},
                rotation: {x: 0, y: 0, z: 0},
                scale: { x: 3, y: 3, z: 3 },
          },
    };
  },
  mounted: function() {
    this.unwatch = $nuxt.$watch(()=>this.location_model, ()=>{this.redraw()}, {deep: true})

    let container = this.$refs.container;
    // SET AND ADD CAMERA
    this.camera = new THREE.PerspectiveCamera(75, window.innerWidth/window.innerHeight, 0.1, 1000);
    this.camera.position.z = 1;
    // ADD SCENE
    this.scene = new THREE.Scene();
    // ADD LIGHT TO SCENE
    this.light = new THREE.AmbientLight(0xffffff, 5.3);
    this.scene.add(this.light);
    // RENDER IN PAGE
    this.renderer = new THREE.WebGLRenderer({antialias: true});
    this.renderer.setClearColor('#dbdbdb');
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.renderer.gammaFactor = 1.5;
    this.renderer.gammaOutput = true;
    container.appendChild(this.renderer.domElement);
    // SET GRID HELPER
    const size = 10;
    const divisions = 25;
    this.gridHelper = new THREE.GridHelper( size, divisions );
    this.scene.add( this.gridHelper );
    // SET ORBIT CONTROLS
    this.orbit = new OrbitControls( this.camera, this.renderer.domElement );
    this.camera.position.set( 0, 2, 2 );
    this.orbit.update();
    this.orbit.addEventListener("change", () => this.renderer.render(this.scene, this.camera));
    // SET MODEL CONTROL
    this.control = new TransformControls( this.camera, this.renderer.domElement );
    this.control.setSize(1);
    this.control.addEventListener( 'change', () => { this.renderer.render(this.scene, this.camera); });


    // EVENT LISTNER TO ENABLE ORBIT CAMERA AND CHANGE VALUE OF INPUTS
    this.control.addEventListener( 'dragging-changed', ( event ) => {
      const gltf_position = this.gltf.position;
      const gltf_rotation = this.gltf.rotation;
      const gltf_scale = this.gltf.scale;
      this.location_model.position = {x: gltf_position.x, y: gltf_position.y, z: gltf_position.z};
      this.location_model.rotation = {x: gltf_rotation.x, y: gltf_rotation.y, z: gltf_rotation.z};
      this.location_model.scale = {x: gltf_scale.x, y: gltf_scale.y, z: gltf_scale.z};
      this.orbit.enabled = ! event.value
    });
    // ADD GLTFLOADER
    this.loader = new GLTFLoader();

    this.loader.load("./meat.glb", gltf => {
      this.gltf = gltf.scene;
      this.scene.add(gltf.scene);

      // ADD CONTROLS TO MODEL
      this.control.attach( gltf.scene );
      this.scene.add( this.control );

      const gltf_scale = this.location_model.scale;
      gltf.scene.scale.set(gltf_scale.x, gltf_scale.y, gltf_scale.z)
      this.orbit.update();

      this.renderer.render(this.scene, this.camera);
    }, undefined,
    error => {
      console.log(error);
    });
 },

 methods: {
   redraw: function() {
     const position = this.location_model.position;
     const rotation = this.location_model.rotation;
     const scale = this.location_model.scale;
     this.gltf.position.set(position.x, position.y, position.z);
     this.gltf.rotation.set(rotation.x, rotation.y, rotation.z);
     this.gltf.scale.set(scale.x, scale.y, scale.z);
     this.renderer.render(this.scene, this.camera);

   },

 },

  watch: {
    selectedtool: function(e) {
      this.selectedtool = e;
      this.control.setMode (e);

    },
    location_model: {
      deep: true,
      handler: function () {

      }
    },
  },
}
</script>

<style>
#container {
  height: 100%;
}
*{
  margin: 0px;
  padding: 0px;
  box-sizing: border-box;
}
.mainContainer {
  width: 100vw;
  height: 100vh;
  overflow-x: hidden;
  position: relative;
}
.inputselectfield {
  position: absolute;
  left: 10px;
  top: 10px;
}
</style>
