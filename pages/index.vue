<template>
  <div class="mainContainer">
    <div ref="container" id="container">

    </div>

    <div class="inputselectfield">

      <p class="selectedGLTF"> {{ this.gltf_clicked_name }}</p>

      <div>
        <input type="file" ref="file"  @change="fileReader" />
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
      orbit_controls: null,
      transform_controls: null,
      loader_gltfs: null,
      gltf_array_buffer: null,
      raycaster: null,
      mouse: null,
      intersects: null,
      load_gltf_models: [],
      load_gltf_model_names: [],
      gltf_clicked_name: "",
    };
  },
  mounted: function() {

    let container = this.$refs.container;

    // SET AND ADD CAMERA
    this.camera = new THREE.PerspectiveCamera( 75, window.innerWidth/window.innerHeight, 0.1, 1000 );
    this.camera.position.set( 0, 2, 2 );

    // ADD SCENE
    this.scene = new THREE.Scene();

    // ADD LIGHT TO SCENE
    this.light = new THREE.AmbientLight( 0xffffff, 5.3 );
    this.scene.add( this.light );

    // RENDER IN PAGE
    this.renderer = new THREE.WebGLRenderer( { antialias: true } );
    this.renderer.setClearColor( '#dbdbdb' );
    this.renderer.setSize( window.innerWidth, window.innerHeight );
    this.renderer.gammaFactor = 1.5;
    this.renderer.gammaOutput = true;
    container.appendChild( this.renderer.domElement );

    // SET GRID HELPER
    const size = 10;
    const divisions = 25;
    this.gridHelper = new THREE.GridHelper( size, divisions );
    this.scene.add( this.gridHelper );

    // SET ORBIT CONTROLS
    this.orbit_controls = new OrbitControls( this.camera, this.renderer.domElement );
    this.orbit_controls.addEventListener( "change", () => this.renderer.render( this.scene, this.camera ) );

    // SET MODEL CONTROL
    this.transform_controls = new TransformControls( this.camera, this.renderer.domElement );
    this.transform_controls.setSize(1);
    this.transform_controls.addEventListener( 'change', () => { this.renderer.render(this.scene, this.camera); } );

    // EVENT LISTNER TO ENABLE ORBIT CAMERA AND CHANGE VALUE OF INPUTS
    this.transform_controls.addEventListener( 'dragging-changed', ( event ) => {
      this.orbit_controls.enabled = ! event.value
    } );

    this.raycaster = new THREE.Raycaster();
    this.mouse = new THREE.Vector2();

    this.$refs.file.onclick = event => event.stopPropagation();

    const onMouseClick = ( event ) => {
      event.preventDefault();
      // calculate mouse position in normalized device coordinates
      // (-1 to +1) for both components
      this.mouse.x = ( event.clientX / window.innerWidth ) * 2 - 1;
      this.mouse.y = - ( event.clientY / window.innerHeight ) * 2 + 1;
      // update the picking ray with the camera and mouse position
      this.raycaster.setFromCamera( this.mouse, this.camera );

      this.intersects = this.raycaster.intersectObjects( this.load_gltf_models, true );
      if ( this.intersects.length !== 0 ) {

        this.intersects[0].object.traverseAncestors( a => {
          if ( a.name.length > 0 ) this.load_gltf_model_names.map(name => {
            if ( name === a.name ) return this.gltf_clicked_name = a.name;
           } );
        } );
      };
    };

    window.addEventListener( 'click', onMouseClick, false);

    this.renderer.render(this.scene, this.camera);
  },

 methods: {
   loadGLTF: function() {
     // ADD GLTFLOADER
     this.loader_gltfs = new GLTFLoader();

       if ( this.gltf_array_buffer !== null ) {

         this.loader_gltfs.parse(this.gltf_array_buffer, '',

          gltf => {
              this.scene.add( gltf.scene );

              gltf.scene.name = prompt( "Please Enter 3d Model Name" );
              this. load_gltf_model_names.push( gltf.scene.name );
              this.load_gltf_models.push( gltf.scene );


              // ADD CONTROLS TO MODEL
              this.transform_controls.attach( gltf.scene );
              this.scene.add( this.transform_controls );

              gltf.scene.scale.set( 5, 5, 5 );

              this.renderer.render( this.scene, this.camera );

       }, error => {
            console.log( error )
       });
     }
   },

   fileReader:   function() {
     const file = this.$refs.file.files[0]
     const readers = new FileReader();
     const gltf_reader = new Promise( ( resolve, reject ) => {
         readers.onload =  ( e ) => {
           this.gltf_array_buffer = e.target.result;
           resolve()
         };
       }
     );
    readers.readAsArrayBuffer( file ); //async await
    gltf_reader.then( () => this.loadGLTF() );

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
.selectedGLTF {
  background-color: white;
  padding: 5px 0;
  margin-bottom: 10px;
  height: 30px;
  border-radius: 5px;
  text-align: center;
}


</style>
