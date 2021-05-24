<template>
	<div id="container"></div>
</template>
<script>
import * as THREE from "three";
import Stats from "stats.js";//指示器
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { DoubleSide } from 'three';
import img from "../assets/images/0.jpg";
import rt from "../assets/images/skybox/rt.png"
import bk from "../assets/images/skybox/bk.png"
import dn from "../assets/images/skybox/dn.png"
import ft from "../assets/images/skybox/ft.png"
import up from "../assets/images/skybox/up.png"
import lf from "../assets/images/skybox/lf.png"
export default {
	data(){
		return{
			scene:null,
			camera:null,
			renderer:null,
			// controls:null,
			imgUrl:img,
			stats:null,
			rt_img:rt,
			bk_img:bk,
			dn_img:dn,
			ft_img:ft,
			up_img:up,
			lf_img:lf
		}
	},

	mounted(){
		this.init()
	},

	methods:{
		init(){
			this.scene = new THREE.Scene();
			this.camera = new THREE.PerspectiveCamera( 75, window.innerWidth/window.innerHeight, 0.1, 1000 );

			this.renderer = new THREE.WebGLRenderer({antialias:true});//{antialias:true}让立方体平滑
			this.renderer.setSize( window.innerWidth, window.innerHeight );
			document.body.appendChild( this.renderer.domElement );
			window.addEventListener('resize',()=>this.onWindowResize())
			
			const textureLoader = new THREE.TextureLoader();

			const boxTexture = textureLoader.load(this.imgUrl)


			const geometry = new THREE.BoxGeometry( 1, 1, 1 );//几何图形
			const material = new THREE.MeshBasicMaterial( { map:boxTexture,side:DoubleSide} );//材质
			const cube = new THREE.Mesh( geometry, material );//立方体：组合几何图形和材质
			cube.name = 'box'


			const skyBoxGeometry = new THREE.BoxGeometry( 200, 200, 200 );
			const skyMeshBasicMaterial = [
				new THREE.MeshBasicMaterial( { map:textureLoader.load(this.rt_img),side:DoubleSide} ),
				new THREE.MeshBasicMaterial( { map:textureLoader.load(this.lf_img),side:DoubleSide} ),
				new THREE.MeshBasicMaterial( { map:textureLoader.load(this.up_img),side:DoubleSide} ),
				new THREE.MeshBasicMaterial( { map:textureLoader.load(this.dn_img),side:DoubleSide} ),
				new THREE.MeshBasicMaterial( { map:textureLoader.load(this.bk_img),side:DoubleSide} ),
				new THREE.MeshBasicMaterial( { map:textureLoader.load(this.ft_img),side:DoubleSide} )
			];//材质
			const skyCube = new THREE.Mesh( skyBoxGeometry, skyMeshBasicMaterial );//立方体：组合几何图形和材质
			// skyCube.name = "skybox"

			this.controls = new OrbitControls(this.camera, this.renderer.domElement);

			this.scene.add( cube );//添加立方体到场景
			this.scene.add( skyCube );//添加立方体到场景

			this.stats = new Stats();
			this.stats.showPanel(0);
			window.document.body.appendChild(this.stats.dom)

			this.camera.position.set(0,0,5);//设置相机的位置
			this.render()
		},

		onWindowResize(){
			this.renderer.setSize( window.innerWidth, window.innerHeight );
			this.camera.aspect = window.innerWidth/window.innerHeight
			this.camera.updateProjectionMatrix()
		},

		render() {
			this.stats.begin()
			requestAnimationFrame( this.render );
			// const box = this.scene.getObjectByName('box')
			// const skybox = this.scene.getObjectByName('skybox')
			// skybox.rotation.x += 0.001;

			
			// box.rotation.x += 0.01;
			// box.rotation.y += 0.01;
			this.controls.update()

			this.renderer.render( this.scene, this.camera );//渲染场景和相机
			this.stats.end()
		}
	}
}
</script>
