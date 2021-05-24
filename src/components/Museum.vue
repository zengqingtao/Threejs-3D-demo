<template>
	<div id="container">
        <el-dialog 
            :visible.sync="dialogVisible"
        >
            <img :src="imgObject[clickMeshName]" />
        </el-dialog>
    </div>
</template>

<script>
import * as THREE from "three"
import { DoubleSide } from 'three';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
// import { FlyControls } from 'three/examples/jsm/controls/FlyControls';
// import {
// 	GLTFLoader
// } from 'three/examples/jsm/loaders/GLTFLoader'
export default {
	data(){
		return{
			scene:null,
			renderer:null,
			camera:null,
			mixer:null,
			clock:null,
			motion : {
				height: 1.8,
				speed: 0.2,
				turnSpeed: Math.PI * 0.02
			},
			delta:0.008, // seconds.
			moveDistance:'', // 20 pixels per second 
			rotateAngle:'', // pi/2 radians (90 degrees) pen second
			
			obj3:null,
			obj5:null,
            keyMap:{},

			// sphereObject:null,
			crossFadeControls:[],
			currentBaseAction:'idle',
			allActions:[],
			baseActions : {
				idle: {
					weight: 1
				},
				walk: {
					weight: 0
				},
				run: {
					weight: 0
				}
			},
			additiveactions : {
				sneak_pose: {
					weight: 0
				},
				sad_pose: {
					weight: 0
				},
				agree: {
					weight: 0
				},
				headshake: {
					weight: 0
				}
			},
			numAnimations:null,
            raycaster:null,
            mouse:null,
            clickObject:[],
            imgObject:{
                "shufa1":require("./img/shufa1.jpg"),
                "shufa2":require("./img/shufa2.jpg"),
                "shufa3":require("./img/shufa3.jpg"),
                "shufa4":require("./img/shufa4.jpg"),
                "shufa5":require("./img/shufa5.jpg"),
            },
            clickMeshName:'',
            dialogVisible:false
		}
	},

	mounted(){
		this.moveDistance = 20 * this.delta
		this.rotateAngle = Math.PI / 2 * this.delta
		this.init()
        
        window.addEventListener( 'mousedown',this.onMouseDown, false );
        window.addEventListener( 'mousemove',this.onMouseMove, false );
	},

	methods:{

        onMouseDown( event ) {

            // calculate mouse position in normalized device coordinates
            // (-1 to +1) for both components
            this.mouse = new THREE.Vector2();
            this.mouse.x = ( event.clientX / window.innerWidth ) * 2 - 1;
            this.mouse.y = - ( event.clientY / window.innerHeight ) * 2 + 1;

            this.raycaster = new THREE.Raycaster();
            this.raycaster.setFromCamera( this.mouse, this.camera );

            // calculate objects intersecting the picking ray
            const intersects = this.raycaster.intersectObjects( this.clickObject);//this.scene.children
            // console.log(this.scene.children)
            console.log(intersects)

            // for ( let i = 0; i < intersects.length; i ++ ) {

            //     intersects[ i ].object.material.color.set( 0xffff00 );
            // }
            if(intersects.length){
                this.clickMeshName = intersects[0].object.name
                this.dialogVisible = true
            }

            // this.renderer.render(this.scene, this.camera);

        },

        onMouseMove(event){
            // calculate mouse position in normalized device coordinates
            // (-1 to +1) for both components
            this.mouse = new THREE.Vector2();
            this.mouse.x = ( event.clientX / window.innerWidth ) * 2 - 1;
            this.mouse.y = - ( event.clientY / window.innerHeight ) * 2 + 1;

            this.raycaster = new THREE.Raycaster();
            this.raycaster.setFromCamera( this.mouse, this.camera );

            // calculate objects intersecting the picking ray
            const intersects = this.raycaster.intersectObjects( this.clickObject);//this.scene.children
            // console.log(this.scene.children)
            console.log(intersects)

            if(intersects.length){
                document.body.style.cursor = 'pointer'
            }else{
                document.body.style.cursor = 'default'
            }
        },

        init() {

            let container = document.getElementById('container');
            this.clock = new THREE.Clock();

            this.scene = new THREE.Scene();

            this.scene.translateX(3.0).translateY(-0.09).translateZ(-2.5);
            this.scene.background = new THREE.Color(0xa0a0a0);
            this.scene.fog = new THREE.Fog(0xa0a0a0, 10, 50);

            const hemilight = new THREE.HemisphereLight(0x606060, 0x2a2a35);
            hemilight.position.set(0, 1, 0);
            this.scene.add(hemilight);

            // //Texture
            const walle = new THREE.TextureLoader().load(require("./img/walle.jpg"));
            walle.wrapS = THREE.RepeatWrapping;
            walle.wrapT = THREE.RepeatWrapping;
            walle.repeat.set(4, 1.5);

            const texture2 = new THREE.TextureLoader().load(require("./img/stone-wall.jpg"));
            texture2.wrapS = THREE.RepeatWrapping;
            texture2.wrapT = THREE.RepeatWrapping;
            texture2.repeat.set(4, 1);

            const texture3 = new THREE.TextureLoader().load(require("./img/ceiling2.jpg"))
            texture3.wrapS = THREE.RepeatWrapping;
            texture3.wrapT = THREE.RepeatWrapping;
            texture3.repeat.set(32, 32);
            const texture4 = new THREE.TextureLoader().load(require("./img/floor.jpg"));
            texture4.wrapS = THREE.RepeatWrapping;
            texture4.wrapT = THREE.RepeatWrapping;
            texture4.repeat.set(32, 32);
            // const texture5 = new THREE.TextureLoader().load(require("./img/alexander.jpg"));
            const texture6 = new THREE.TextureLoader().load(require("./img/map.jpg"));
            // const texture7 = new THREE.TextureLoader().load(require("./img/art.jpg"));
            const texture8 = new THREE.TextureLoader().load(require("./img/shufa2.jpg"));
            const texture9 = new THREE.TextureLoader().load(require("./img/shufa1.jpg"));
            const texture10 = new THREE.TextureLoader().load(require("./img/shufa3.jpg"));
            const texture11 = new THREE.TextureLoader().load(require("./img/shufa5.jpg"));
            const texture12 = new THREE.TextureLoader().load(require("./img/shufa4.jpg"));
            const texture13 = new THREE.TextureLoader().load(require("./img/art.jpg"));

            // floor

            const mesh = new THREE.Mesh(new THREE.PlaneGeometry(8, 10, 20, 20), new THREE.MeshBasicMaterial({
                map: texture4,
                side: DoubleSide
            }));
            mesh.rotation.x = -Math.PI / 2;
            mesh.receiveshadow = true;
            this.scene.add(mesh);

            //ceiling
            const planeGeometry3 = new THREE.PlaneGeometry(8, 10, 20, 20);
            const planematerial3 = new THREE.MeshBasicMaterial({
                map: texture3,
                side: DoubleSide
            });
            const planeObject3 = new THREE.Mesh(planeGeometry3, planematerial3);
            this.scene.add(planeObject3);
            planeObject3.translateX(0.0).translateY(2.5).translateZ(0.0);
            planeObject3.rotation.x = -Math.PI / 2;

            //Pictures
            // const pic1 = new THREE.Mesh(new THREE.PlaneGeometry(1, 2), new THREE.MeshBasicMaterial({
            //     map: texture5,
            //     side: DoubleSide
            // }));
            // this.scene.add(pic1);

            // pic1.translateX(3.97).translateY(1.26).translateZ(2.0);
            // pic1.rotateOnAxis(new THREE.Vector3(0, 1, 8), Math.PI / 2);

            const pic2 = new THREE.Mesh(new THREE.PlaneGeometry(4, 2), new THREE.MeshBasicMaterial({
                map: texture6,
                // side: DoubleSide
            }));
            this.scene.add(pic2);
            pic2.translateX(1.6).translateY(1.26).translateZ(4.8);
            pic2.rotation.y = Math.PI / 2;
            pic2.rotateOnAxis(new THREE.Vector3(0, 1), Math.PI / 2);

            // const pic3 = new THREE.Mesh(new THREE.PlaneGeometry(4, 2), new THREE.MeshBasicMaterial({
            //     map: texture7,
            //     side: DoubleSide
            // }));
            // this.scene.add(pic3);
            // pic3.translateX(-3.95).translateY(1.26).translateZ(-0.2);
            // // pic3.rotation.z = Math.PI / 2;
            // pic3.rotateOnAxis(new THREE.Vector3(0, 1, 6), Math.PI / 2);

            const pic4 = new THREE.Mesh(new THREE.PlaneGeometry(1.8, 1.8), new THREE.MeshBasicMaterial({
                map: texture9,
                side: DoubleSide
            }));
            this.scene.add(pic4);
            pic4.translateX(-2.05).translateY(1.26).translateZ(-0.55);
            pic4.rotateOnAxis(new THREE.Vector3(0, 1, 0), Math.PI / 2);
            pic4.name="shufa1"
            this.clickObject.push(pic4)

            const pic5 = new THREE.Mesh(new THREE.PlaneGeometry(1, 2), new THREE.MeshBasicMaterial({
                map: texture8,
                side: DoubleSide
            }));
            this.scene.add(pic5);
            pic5.translateX(-3.95).translateY(1.26).translateZ(3.5);
            pic5.rotation.x = Math.PI / 2;
            pic5.rotation.y = Math.PI / 2;
            pic5.rotateOnAxis(new THREE.Vector3(0, 0, -1), Math.PI / 2);
            pic5.name="shufa2"
            this.clickObject.push(pic5)

            const pic6 = new THREE.Mesh(new THREE.PlaneGeometry(0.8, 0.8), new THREE.MeshBasicMaterial({
                map: texture10,
                side: DoubleSide
            }));
            this.scene.add(pic6);
            pic6.translateX(-4.5).translateY(1.26).translateZ(5.1);
            pic6.rotation.y = Math.PI / 2;
            pic6.rotateOnAxis(new THREE.Vector3(0, -1, 0), Math.PI / 2);
            pic6.name="shufa3"
            this.clickObject.push(pic6)

            const pic7 = new THREE.Mesh(new THREE.PlaneGeometry(1, 2), new THREE.MeshBasicMaterial({
                map: texture11,
                side: DoubleSide
            }));
            this.scene.add(pic7);
            pic7.translateX(3.97).translateY(1.3).translateZ(-1.5);
            pic7.rotateOnAxis(new THREE.Vector3(0, -1,0), Math.PI / 2);
            pic7.name = 'shufa5'
            this.clickObject.push(pic7)


            const pic8 = new THREE.Mesh(new THREE.PlaneGeometry(0.8, 0.8), new THREE.MeshBasicMaterial({
                map: texture12,
                side: DoubleSide
            }));
            this.scene.add(pic8);
            pic8.translateX(-0.99).translateY(1.7).translateZ(4);
            pic8.rotateOnAxis(new THREE.Vector3(0, 1, 0), Math.PI / 2);
            pic8.name = "shufa4"
            this.clickObject.push(pic8)

            const pic9 = new THREE.Mesh(new THREE.PlaneGeometry(3, 2), new THREE.MeshBasicMaterial({
                map: texture13,
                side: DoubleSide
            }));
            this.scene.add(pic9)
            pic9.translateX(0.0).translateY(1.26).translateZ(-3.85);
            // pic9.rotation.x = Math.PI /2;
            pic9.rotation.y = Math.PI / 2;
            pic9.rotateOnAxis(new THREE.Vector3(0, -1, 0), Math.PI / 2);

            // //Building

            //lleftwall
            const cubeGeometry2 = new THREE.BoxGeometry(2.5, 1, 10);
            const cubeMaterial2 = new THREE.MeshBasicMaterial({
                map: walle
            });
            const leftwall = new THREE.Mesh(cubeGeometry2, cubeMaterial2);
            this.scene.add(leftwall);

            // set position of the cube
            leftwall.translateX(-4.5).translateY(1.26).translateZ(0.0);
            leftwall.rotateOnAxis(new THREE.Vector3(0, 0, 1), Math.PI / 2);

            // backwall
            const cubeGeometry1 = new THREE.BoxGeometry(2.5, 1, 8);
            const cubeMaterial1 = new THREE.MeshBasicMaterial({
                map: walle
            });
            const backwall = new THREE.Mesh(cubeGeometry1, cubeMaterial1);
            this.scene.add(backwall);

            // Set position of the cube
            backwall.translateX(0.0).translateY(1.26).translateZ(-4.5);
            backwall.rotation.x = Math.PI / 2;
            backwall.rotation.y = Math.PI / 2;

            // rightwall
            const cubeGeometry = new THREE.BoxGeometry(2.5, 1, 10);
            const cubeMaterial = new THREE.MeshBasicMaterial({
                map: walle
            });
            const rightwall = new THREE.Mesh(cubeGeometry, cubeMaterial);
            this.scene.add(rightwall);

            // set position of the cube
            rightwall.translateX(4.5).translateY(1.26).translateZ(0.0);
            rightwall.rotateOnAxis(new THREE.Vector3(0, 0, 1), Math.PI / 2);

            //interior Walls
            // Wall1
            const cubegeometryw = new THREE.BoxGeometry(2.45, 1, 2);
            const cubematerialw = new THREE.MeshBasicMaterial({
                map: texture2
            });
            const Wall1 = new THREE.Mesh(cubegeometryw, cubematerialw);
            this.scene.add(Wall1);

            // set position of the cube
            Wall1.translateX(-1.5).translateY(1.26).translateZ(3.9);
            Wall1.rotateOnAxis(new THREE.Vector3(0, 0, 1), Math.PI / 2);

            // Wall3
            const cubeGeometryw3 = new THREE.BoxGeometry(2.45, 1, 2);
            const cubeMaterialw3 = new THREE.MeshBasicMaterial({
                map: texture2
            });
            const Wall3 = new THREE.Mesh(cubeGeometryw3, cubeMaterialw3);
            this.scene.add(Wall3);

            // Set position of the cube
            Wall3.translateX(3.0).translateY(1.26).translateZ(-0.2);
            Wall3.rotation.x = Math.PI / 2;
            Wall3.rotation.y = Math.PI / 2;

            // Wall4 添加墙4
            const cubeGeometryw4 = new THREE.BoxGeometry(2.45, 1, 2);
            const cubeMaterialw4 = new THREE.MeshBasicMaterial({
                map: texture2
            });
            const Wall4 = new THREE.Mesh(cubeGeometryw4, cubeMaterialw4);
            this.scene.add(Wall4);

            // Set position of the cube 把墙4立起来
            Wall4.translateX(-1.5).translateY(1.26).translateZ(-0.6);
            Wall4.rotateOnAxis(new THREE.Vector3(0, 0, 1), Math.PI / 2);

            // // frontwall 添加前面的墙（半透明）
            const cubeGeometryF = new THREE.BoxGeometry(2.5, 0.01, 8);
            const cubeMaterialF = new THREE.MeshBasicMaterial({
                color: 'rgb(255,250,250)',
                opacity: 0.5,
                transparent: true
            });
            const frontwall = new THREE.Mesh(cubeGeometryF, cubeMaterialF);
            this.scene.add(frontwall);

            // // set position of the cube 定位前面的墙，让其立起来
            frontwall.translateX(0.0).translateY(1.25).translateZ(4.95);
            frontwall.rotation.x = Math.PI / 2;
            frontwall.rotation.y = Math.PI / 2;

            //Load Models

		
            // const loader = new GLTFLoader();
            // let obj;
			// let obj2;
			// let obj4;

            // loader.load("models/models/robot/scene.gltf", function(gltf) {
            //     obj = gltf.scene;
            //     gltf.scene.scale.set(0.01, 0.008, 0.01);
            //     this.scene.add(gltf.scene);
            //     obj.translateX(-3.0).translateY(0.01).translateZ(-3.85);
            //     obj.rotation.y -= Math.PI / 2;
            //     obj.rotateOnAxis(new THREE.Vector3(0, 1, 0), Math.PI / 2);
            // });

            // loader.load("models/models/modern_door/scene.gltf", function(gltf2) {
            //     obj2 = gltf2.scene;
            //     gltf2.scene.scale.set(0.1, 0.15, 0.1);
            //     this.scene.add(gltf2.scene);
            //     obj2.translateX(-3.0).translateY(0.01).translateZ(4.95);
            //     obj2.rotateOnAxis(new THREE.Vector3(0, 1, 0), Math.PI / 2);
            // });

            // loader.load("models/models/book/scene.gltf", function(gltf3) {
            //     this.obj3 = gltf3.scene;
            //     gltf3.scene.scale.set(0.34, 0.4, 0.34);
            //     this.scene.add(gltf3.scene);
            //     this.obj3.translateX(-0.5).translateY(1.07).translateZ(3.5);
            //     this.obj3.rotateOnAxis(new THREE.Vector3(0, 1, 0), Math.PI / 2);
            //     this.obj3.rotation.y += 2.0;
            // });

            // loader.load("models/models/stool/scene.gltf", function(gltf4) {
            //     obj4 = gltf4.scene;
            //     gltf4.scene.scale.set(0.02, 0.015, 0.04);
            //     this.scene.add(gltf4.scene);
            //     obj4.translateX(-0.5).translateY(0.05).translateZ(3.5);
            //     obj4.rotateOnAxis(new THREE.Vector3(0, 1, 0), Math.PI / 2);
            // });

            // loader.load("models/models/ink_and_quil1/scene.gltf", function(gltf5) {
            //     this.obj5 = gltf5.scene;
            //     this.scene.add(gltf5.scene);
            //     this.obj5.translateX(-0.5).translateY(1.13).translateZ(3.52);
            //     this.obj5.rotatenAxis(new THREE.Vector3(0, 1, 0), Math.PI / 2);
            //     this.obj5.rotation.y += 1.1;
            // });

            // loader.load("models/xbot.glb", function(gltf) {
            //     let model = gltf.scene;
            //     this.scene.add(model);
            //     model.translateX(3.0).translateY(0.0).translateZ(-2.8);
            //     model.rotateOnAxis(new THREE.Vector3(0, -1, 0), Math.PI / 2);

            //     model.traverse(function(object) {

            //         if (object.isMesh) object.castShadow = true;

            //     });

            //     let skeleton = new THREE.SkeletonHelper(model);
            //     skeleton.visible = false;
            //     this.scene.add(skeleton);

            //     const animations = gltf.animations;
            //     this.mixer = new THREE.AnimationMixer(model);

            //     this.numAnimations = animations.length;

            //     for (let i = 0; i !== this.numAnimations; + i) {

            //         let clip = animations[i];
            //         const name = clip.name;

            //         if (this.baseActions[name]) {

            //             const action = this.mixer.clipAction(clip);
            //             this.activateAction(action);
            //             this.baseActions[name].action = action;
            //             this.allActions.push(action);
            //         } else {
            //             // Make the clip additive and remove the reference frame 
            //             THREE.AnimationUtils.makeClipAdditive(clip);
            //             if (clip.name.endswith('pose')) {

            //                 clip = THREE.AnimationUtils.subclip(clip, clip.name, 2, 3, 30);
            //             }

            //             const action = this.mixer.clipAction(clip);
            //             this.activateAction(action);
            //             this.allActions.push(action);
            //         }
            //     }

            //     this.animate();

            // });
            // ↓画线
                let material = new THREE.LineBasicMaterial( { color: 0x0000ff } );
                let geometry = new THREE.Geometry();
                    geometry.vertices.push(new THREE.Vector3( -0.2, 0, 6) );
                    geometry.vertices.push(new THREE.Vector3( 0, 0.2, 6) );
                    geometry.vertices.push(new THREE.Vector3( 0.2, 0, 6) );
                    geometry.vertices.push(new THREE.Vector3( 0.1, 0, 6) );
                    geometry.vertices.push(new THREE.Vector3( 0.1, -0.2, 6) );
                    geometry.vertices.push(new THREE.Vector3( -0.1, -0.2, 6) );
                    geometry.vertices.push(new THREE.Vector3( -0.1, -0.2, 6) );
                    geometry.vertices.push(new THREE.Vector3( -0.1, 0, 6) );
                    geometry.vertices.push(new THREE.Vector3( -0.2, 0, 6) );

                let line = new THREE.Line( geometry, material );
                this.scene.add( line );
                
            //↑
			
            this.renderer = new THREE.WebGLRenderer({
                antialias: true
            });
            this.renderer.setPixelRatio(window.devicePixelRatio);
            this.renderer.setSize(window.innerWidth, window.innerHeight);
            this.renderer.outputEncoding = THREE.sRGBEncoding;
            this.renderer.shadowMap.enabled = true;
            container.appendChild(this.renderer.domElement);

		 

            // camera
            this.camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 1, 100);
            this.camera.position.set(-0.1, 1, 6);
            // this.camera.translateX(3.0).translateY(5.0).translateZ(5.5);

            const controls = new OrbitControls(this.camera, this.renderer.domElement);
            controls.enablePan = true;
            controls.enablezoom = true;
            controls.target.set(0, 1, 0)
            controls.update();
            this.camera.lookAt(controls.target)
            //   const controls =new FlyControls(this.camera);
            //     controls.movementSpeed = 100; //设置移动的速度
            //     controls.rollSpeed = Math.PI / 6; //设置旋转速度
            //     controls.autoForward = false;
            //     controls.dragToLook = true;

            // window.addEventListener('resize', this.onWindowResize);

			this.animate()//我加到这里，渲染项目
        },

        // activateAction(action) {

        //     const clip = action.getClip();
        //     const settings = this.baseActions[clip.name] || this.additiveactions[clip.name];
        //     this.setweight(action, settings.weight);
        //     action.play();
        // },

        // modifyTimescale(speed) {

        //     this.mixer.timeScale = speed;

        // },

        // preparecrossFade(startAction, endAction, duration) {
        //     // If the current action is 'idle', execute the crossfade immediately;
        //     // else wait until the current action has finished its current loop
        //     if (this.currentBaseAction === 'idle' || !startAction || !endAction) {

        //         this.executeCrossFade(startAction, endAction, duration);

        //     } else {

        //         this.synchronizeCrossFade(startAction, endAction, duration);

        //     }

        //     // Update control colors

        //     if (endAction) {

        //         const clip = endAction.getClip();
        //         this.currentBaseAction = clip.name;

        //     } else {

        //         this.currentBaseAction = 'None';

        //     }

        //     this.crossFadeControls.forEach(function(control) {

        //         const name = control.property;
        //         if (name === this.currentBaseAction) {

        //             control.setActive();

        //         } else {

        //             control.setInactive();

        //         }

        //     });

        // },

        // synchronizecrossFade(startAction, endAction, duration) {

        //     this.mixer.addEventListener('loop', onLoopFinished);

        //     function onLoopFinished(event) {

        //         if (event.action === startAction) {

        //             this.mixer.removeEventListener('loop', onLoopFinished);
        //             this.executeCrossFade(startAction, endAction, duration);

        //         }
        //     }
        // },

        // executeCrossFade(startAction, endAction, duration) {
        //     // Not only the start action, but also the end action must get a weight of 1 before fading
        //     // (concerning the start action this is already guaranteed in this place)

        //     if (endAction) {

        //         this.setweight(endAction, 1);
        //         endAction.time = 0;

        //         if (startAction) {

        //             // Crossfade with warping

        //             startAction.crossFadeTo(endAction, duration, true);
        //         } else {

        //             //Fade in

        //             endAction.fadeIn(duration);

        //         }
        //     } else {

        //         //Fade out

        //         startAction.fadeOut(duration);

        //     }
        // },

        // // This function is needed, since animationAction.crossFadeTo() disables its start action and sets
        // // the start action's timescale to ((start animation's duration) / (end animation's duration))

        // setweight(action, weight) {

        //     action.enabled = true;
        //     action.setEffectiveTimeScale(1);
        //     action.setEffectiveweight(weight);
        // },

        // onwindowResize() {
        //     this.camera.aspect = window.innerWidth / window.innerHeight;
        //     this.camera.updateProjectionMatrix();

        //     this.renderer.setSize(window.innerWidth, window.innerHeight);
        // },

    	animate() {
            // Render loop
            requestAnimationFrame(this.animate)


            // // for (let i = 0; i !== this.numAnimations; ++i) {

            // //     const action = this.allActions[i];
            // //     const clip = action.getClip();
            // //     const settings = this.baseActions[clip.name] || this.additiveactions[clip.name];
            // //     settings.weight = action.getEffectiveweight();

            // // }


            // Key controls
            if (this.keyMap[87] || this.keyMap[119]) { // w key
                this.camera.translateZ(-this.moveDistance);
                this.keyMap[87] = false;
                this.keyMap[119] = false;
                console.log("11")
            }

            if (this.keyMap[83] || this.keyMap[115]) { // s key
                this.camera.translateZ(this.moveDistance);
                this.keyMap[83] = false;
                this.keyMap[115] = false;
            }

            if (this.keyMap[65] || this.keyMap[97]) { // A key
                // Redirect motion by 90 degrees
                this.camera.rotateOnAxis(new THREE.Vector3(0, 1, 0),this.rotateAngle * 25)
                this.keyMap[65] = false;
                this.keyMap[97] = false;
            }

            if (this.keyMap[68] || this.keyMap[100]) { // D key
                this.camera.rotateOnAxis(new THREE.Vector3(0, 1, 0), -this.rotateAngle * 25)
                this.keyMap[68] = false;
                this.keyMap[100] = false;
            }

			let tTracker=false;
            //Corridor Lights
            if (this.keyMap[80] || this.keyMap[112]) { //P
                tTracker = !tTracker;
                if (tTracker) {
                    const midlight = new THREE.PointLight(0x9ACD32, 1, 5);
                    midlight.translateX(-3.0).translateY(1.5).translateZ(0.0);
                    midlight.castshadow = true;
                    midlight.name = "midLight";

                    this.scene.add(midlight);
                } else {
                    this.scene.remove(this.scene.getobjectByName("midLight"));
                }
                this.keyMap[80] = false;
                this.keyMap[112] = false;
            }


			// let lTracker=false;

            // if (this.keyMap[81] || this.keyMap[113]) { //Q
            //     lTracker = !lTracker;
            //     if (lTracker) {
            //         const backLight = new THREE.PointLight(0x005eff, 1, 5);
            //         backLight.translateX(-3.0).translateY(1.5).translateZ(-3.0);
            //         backLight.castshadow = true;
            //         backLight.name = "backLight";
            //         this.scene.add(backLight);

            //     } else {
            //         this.scene.remove(this.scene.getobjectByName("backLight"))
            //         this.keyMap[81] = false;
            //         this.keyMap[113] = false;
            //     }

			// 	let rTracker = false;
            //     if (this.keyMap[82] || this.keyMap[114]) { //R
            //         rTracker = !rTracker;
            //         if (rTracker) {
            //             const frontLight = new THREE.PointLight(0xE066FF, 0.5, 5);
            //             frontLight.translateX(-3.0).translateY(1.0).translateZ(2.5);
            //             frontLight.castshadow = true;
            //             this.scene.add(frontLight);
            //             frontLight.name = "frontLight";
            //         } else {
            //             this.scene.remove(this.scene.getObjectByName("frontLight"));
            //         }

            //         this.keyMap[82] = false;
            //         this.keyMap[114] = false;
            //     }
            //     //lights around the Robot 
			// 	let Tracker8 = false;
            //     if (this.keyMap[56]) { //8
            //         Tracker8 = !Tracker8;
            //         if (Tracker8) {
            //             const LightR = new THREE.PointLight(0x005eff, 2, 5);
            //             LightR.translateX(2.7).translateY(0.0).translateZ(-2.8);
            //             LightR.castshadow = true;
            //             this.scene.add(LightR);
            //             LightR.name = "LightR";
            //         } else {
            //             this.scene.remove(this.scene.getobjectByName("LightR"));
            //         }
            //         this.keyMap[56] = false;
            //     }

			// 	let Tracker9 = false;
            //     if (this.keyMap[57]) { //9
            //         Tracker9 = !Tracker9;
            //         if (Tracker9) {
            //             const LightR2 = new THREE.PointLight(0xffff00, 2, 5);
            //             LightR2.translateX(2.7).translateY(0.0).translateZ(-2.8);
            //             LightR2.castshadow = true;
            //             this.scene.add(LightR2);
            //             LightR2.name = "LightR2";
            //         } else {
            //             this.scene.remove(this.scene.getobjectByName("LightR2"));
            //         }
            //         this.keyMap[57] = false;
            //     }

            //     if (this.keyMap[85] || this.keyMap[117]) {
            //         this.camera.rotateOnAxis(new THREE.Vector3(1, 0, 0), this.rotateAngle)
            //         this.keyMap[85] = false;
            //         this.keyMap[117] = false;
            //     }

            //     if (this.keyMap[84] || this.keyMap[116]) {
            //         this.camera.rotateOnAxis(new THREE.Vector3(1, 0, 0), -this.rotateAngle)
            //         this.keyMap[84] = false;
            //         this.keyMap[116] = false;
            //     }

            //     if (this.keyMap[49] || this.keyMap[50] || this.keyMap[51]) {
            //         let activity;
            //         if (this.keyMap[49]) {
            //             activity = 'idle'
            //         } else {
            //             activity = 'run'
            //         }

            //         const settings = this.baseActions[activity];
            //         const currentSettings = this.baseActions[this.currentBaseAction];
            //         const currentAction = currentSettings ? currentSettings.action : null
            //         const action = settings ? settings.action : null

            //         this.preparecrossFade(currentAction, action, 0.35)

            //         this.keyMap[49] = false
            //         this.keyMap[50] = false
            //         this.keyMap[51] = false

            //     }

            //     //interact with the models on the stool 
            //     if (this.keyMap[89] || this.keyMap[121]) { // y key
            //         this.obj5.rotation.y += 0.1 * 1.2;
            //         this.obj3.rotation.y += 0.1 * 1.2;
            //         this.keyMap[89] = false;
            //         this.keyMap[121] = false;
            //     }

            //     if (this.keyMap[90] || this.keyMap[122]) { // z key
            //         this.obj3.rotation.y -= 0.1 * 2;
            //         this.keyMap[90] = false;
            //         this.keyMap[122] = false;
            //     }

			// 	let xTracker = false;

			// 	    if (this.keyMap[88] || this.keyMap[120]) { //x
			// 	        xTracker = !xTracker;
			// 	        if (xTracker) {
			// 	            const dLight = new THREE.SpotLight(0xFFF00, 2, 2);
			// 	            dLight.translateX(0.0).translater(0).translateZ(4.0);
			// 	            dLight.castshadow = true;
			// 	            dLight.name = "dLight";

			// 	            this.scene.add(dLight);
			// 	        } else {
			// 	            this.scene.remove(this.scene.getobjectByName("dLight"));
			// 	        }
			// 	        this.keyMap[88] = false;
			// 	        this.keyMap[120] = false;
			// 	    }
                let that = this
                const keyPress = function(event){
                    that.keyMap[event.keyCode] = true;
                    
                }

                window.addEventListener('keypress', keyPress);

                // Get the time elapsed since the last frame, used for mixer update

                // const mixerUpdateDelta = this.clock.getDelta();

                // update the animation mixer, the stats panel, and render this frame 

                // this.mixer.update(mixerUpdateDelta);

            // }
            // update the picking ray with the camera and mouse position
                this.renderer.render(this.scene, this.camera);
        }

	}
}
</script>


<style scoped>
    img{
        width: 80%;
    }
</style>