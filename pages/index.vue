
<template>
  <div class="mainContainer">
    <div ref="container" id="container"></div>

    <div>
        <p>Position</p>
        <input ref="text" type="number" name="position" v-model="position.x" @change="inputChange" />
        <input type="number" name="position" v-model="position.y" @change="inputChange" />
        <input type="number" name="position" v-model="position.z" @change="inputChange" />
    </div>

    <div>
        <p>Rotation</p>
        <input type="number" name="rotation" v-model="rotation.x" @change="inputChange" />
        <input type="number" name="rotation" v-model="rotation.y" @change="inputChange" />
        <input type="number" name="rotation" v-model="rotation.z" @change="inputChange" />
    </div>

    <div>
        <p>Scale</p>
        <input type="number" name="scale" v-model="scale.x" @change="inputChange" />
        <input type="number" name="scale" v-model="scale.y" @change="inputChange" />
        <input type="number" name="scale" v-model="scale.z" @change="inputChange" />
    </div>

      <div>
      <p>Position Rotation Scaled Selector</p>
      <select  @change="positionRotationScale" v-model="selectTool">
        <option>position</option>
        <option>rotation</option>
        <option>scale</option>
      </select>
    </div>
  </div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from '~~/node_modules/three/examples/jsm/loaders/GLTFLoader.js';
import { OrbitControls } from '~~/node_modules/three/examples/jsm/controls/OrbitControls.js';
import  TransformControls  from '../assets/TransformControls.js';


export default {
  name: 'THREETest',
  data() {
    return {
      camera: null,
      scene: null,
      renderer: null,
      gltf: null,
      orbit: null,
      control: null,

      selectTool: "position",
      position: { x: 0, y: 0, z: 0 },
      rotation: { x: 0, y: 0, z: 0 },
      scale: { x: 3, y: 3, z: 3 },
    }
  },
  mounted() {
    let container = this.$refs.container;
    // SET AND ADD CAMERA
    this.camera = new THREE.PerspectiveCamera(75, container.clientWidth/container.clientHeight, 0.1, 1000);
    this.camera.position.z = 1;
    // ADD SCENE
    this.scene = new THREE.Scene();
    // ADD LIGHT TO SCENE
    this.light = new THREE.AmbientLight(0xffffff, 5.3);
    this.scene.add(this.light);
    // RENDER IN PAGE
    this.renderer = new THREE.WebGLRenderer({antialias: true});
    this.renderer.setClearColor('#dbdbdb');
    this.renderer.setSize(container.clientWidth, container.clientHeight);
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
      const gltf = this.gltf[this.selectTool];
      this[this.selectTool] = {x: gltf.x, y: gltf.y, z: gltf.z}
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

      gltf.scene.scale.set(3, 3, 3)
      this.orbit.update();

      this.renderer.render(this.scene, this.camera);
    }, undefined,
    error => {
      console.log(error);
    });
},

  methods: {
    // CONTROLS SET MODE CHANGE
    positionRotationScale: function(e) {
      console.log(this.$ref)
      switch (e.target.options.selectedIndex) {

        case 0:
          this.control.setMode( "translate" );
          break;

        case 1:
          this.control.setMode( "rotate" );
          break;

        case 2:
          this.control.setMode( "scale" );
          break;

    }
  },

    // INPUT CHAGE
    inputChange: function(e) {
    console.log(this.$ref)
      const selectOption = e.target.name
      const setNumber = this[selectOption];
      this.gltf[selectOption].set(setNumber.x, setNumber.y, setNumber.z)
      this.renderer.render(this.scene, this.camera);
    }
  },
}
</script>

<style>
#container {
  height: 500px;
  width: 500px;
}
*{
  margin: 0px;
  padding: 0px;
}
.mainContainer {
  width: 500px;
  margin: auto;
}
</style>
