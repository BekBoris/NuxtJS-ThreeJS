<template>
  <div class="mainContainer">
    <div ref="container" id="container">

    </div>

    <div class="inputselectfield">

      <div>
        <input type="file" ref="file"  @change="fileReader" />
      </div>

      <div>
        <button type="button" ref="button"  @click="loaderGLTF" >Add to SCENE</button>
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
      loader: null,
      array_buffer: null,
    };
  },
  mounted: function() {

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
      this.orbit.enabled = ! event.value
    });

     this.renderer.render(this.scene, this.camera);
 },

 methods: {
   loaderGLTF: function() {
     // ADD GLTFLOADER
     this.loader = new GLTFLoader();

       if (this.array_buffer !== null ) {

         this.loader.parse(this.array_buffer, '',

          gltf => {
              this.scene.add(gltf.scene);

              // ADD CONTROLS TO MODEL
              this.control.attach( gltf.scene );
              this.scene.add( this.control );

              gltf.scene.scale.set(5, 5, 5);

              this.renderer.render(this.scene, this.camera);

       }, error => {
            console.log(error)
       });
     }
   },

   fileReader: function() {
     const file = this.$refs.file.files[0]
     const reader = new FileReader();

     reader.onload = (e) => {
       this.array_buffer = e.target.result;
    };

    reader.readAsArrayBuffer(file);
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
button {
  padding: 3px;
  margin: 10px 0 0 0;
}
</style>
