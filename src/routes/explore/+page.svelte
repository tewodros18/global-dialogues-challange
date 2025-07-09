<script lang="ts">
    import { browser } from '$app/environment'; 
	import * as THREE from "three"
    import { Reflector} from 'three/examples/jsm/objects/Reflector.js';
    import * as TWEEN from '@tweenjs/tween.js';
    import { onMount } from 'svelte';
    import * as theatrecore from '@theatre/core';
    import studio from '@theatre/studio'
	import { scale } from 'svelte/transition';
    const { Tween, Easing, update } = TWEEN;
    const { getProject, types} = theatrecore;

    if(browser){
        const textureLoader = new THREE.TextureLoader();
        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setAnimationLoop(animate);
        document.body.appendChild(renderer.domElement);

        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        // Three js objects
        const dollyNode = new THREE.Object3D();//object to hold the camera and some UI elements near the camera
        dollyNode.add(camera);
        scene.add(dollyNode);


        const camUI = new THREE.Mesh(
            new THREE.BoxGeometry(5, 3.5, 0.0001),
            new THREE.MeshBasicMaterial({ map: textureLoader.load('camera.png'), transparent: true })
        );// a UI element to represent the caera view
        camUI.position.z = -2; 
        camUI.visible = false;
        dollyNode.add(camUI);


        const plane = new THREE.Mesh(
            new THREE.BoxGeometry(1.8,1.2,0.0001),
            new THREE.MeshBasicMaterial({ map: textureLoader.load('plane.png'), transparent: true })
        );
        plane.position.z = -2; 
        plane.position.x = 6; //6
        plane.position.y = -3;// -3Position the plane in front of the camera
        

        const smoke = new THREE.Mesh(
            new THREE.BoxGeometry(2,1,0.0001),
            new THREE.MeshBasicMaterial({ map: textureLoader.load('smoke.png'), transparent: true })
        );

        smoke.position.z = -0.1;
        smoke.position.y = -0.6;
        smoke.position.x = 0.6;// Position the plane in front of the camera

        plane.add(smoke);
        dollyNode.add(plane);

        const globeBaseNode = new THREE.Object3D();
        scene.add(globeBaseNode);

        const texture = textureLoader.load('globe.png')
        const globe = new THREE.Mesh(
            new THREE.BoxGeometry(3, 3, 0.01),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true})
        )

        globe.position.z = -8; 
        globeBaseNode.add(globe);

        const globePeepGrp = new THREE.Object3D(); 
        scene.add(globePeepGrp);

        const peep = new THREE.Mesh(
            new THREE.BoxGeometry(0.25,0.5,0.0001),
            new THREE.MeshBasicMaterial({ map: textureLoader.load('./characters/peep-standing-16.png'), transparent: true })
        );

        globePeepGrp.add(peep);
        peep.position.set(0.55, 0.35, -2);

        const peep2 = new THREE.Mesh(
            new THREE.BoxGeometry(0.25,0.5,0.0001),
            new THREE.MeshBasicMaterial({ map: textureLoader.load('./characters/peep-standing-16.png'), transparent: true })
        );

        globePeepGrp.add(peep2);
        peep2.position.set(-0.29, -0.25, -2);



        const peep3 = new THREE.Mesh(
            new THREE.BoxGeometry(0.25,0.5,0.0001),
            new THREE.MeshBasicMaterial({ map: textureLoader.load('./characters/peep-standing-16.png'), transparent: true })
        );

        globePeepGrp.add(peep3);
        peep3.position.set(-0.65, 0.35, -2);

        globePeepGrp.visible = true;




        updateCameraPosition(globeBaseNode);
        function updateCameraPosition(Node: THREE.Object3D) {
            camera.position.set(Node.position.x, Node.position.y, Node.position.z);
            //camera.lookAt(globeBaseNode.position);
        }

        window.addEventListener('resize', () => {
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix(); 
        });

        

        function animate() {
            //plane.position.x -= 0.02;
            //plane.position.y += 0.01;
            //smoke.position.x += 0.0002;
            //smoke.position.y += 0.0001;
            //smoke.scale.x += 0.001;
            if(plane.position.y > 2.5){
                plane.position.y = -3;
                plane.position.x = 6;
                smoke.position.y = -0.5;
                smoke.position.x = 0.6;
                smoke.scale.x = 1;
            }
            renderer.render(scene,camera);
        }


        studio.initialize();

        // Theatre.js setup

        const project = getProject("CIP Explorer");

        // sheet to animate the globe page
        const globe_sheet = project.sheet("Globe");
        globe_sheet.sequence.attachAudio({ source: './audio/intro.wav' }).then(() => {
            console.log('Audio loaded!')
        })


        const globeObj = globe_sheet.object("Globe", {
            position: types.compound({
                x: types.number(globe.position.x, { range: [-10, 10] }),
                y: types.number(globe.position.y, { range: [-10, 10] }),
                z: types.number(globe.position.z, { range: [-10, 10] }),
            }),
            rotation: types.compound({
                x: types.number(globe.rotation.x, { range: [-2, 2] }),
                y: types.number(globe.rotation.y, { range: [-2, 2] }),
                z: types.number(globe.rotation.z, { range: [-2, 2] }),
            }),
        })

        globeObj.onValuesChange((values) => {
            const { x, y, z } = values.position;
            globe.position.set(x, y, z);
            globe.rotation.set(values.rotation.x * Math.PI, values.rotation.y * Math.PI, values.rotation.z * Math.PI);
        });

        const camUIObj = globe_sheet.object("CamUI", {
            visibility: types.boolean(false),
        });

        camUIObj.onValuesChange((values) => {
            camUI.visible = values.visibility;
        });

        const introPeepObj = globe_sheet.object("IntroPeep", {
            visibility: types.boolean(false),
            scale: types.compound({
                x: types.number(peep.scale.x, { range: [-2, 2] }),
                y: types.number(peep.scale.y, { range: [-2, 2] }),
                z: types.number(peep.scale.z, { range: [-2, 2] }),
            }),
        })

        introPeepObj.onValuesChange((values) => {
            peep.visible = values.visibility;
            peep.scale.set(+(values.scale.x.toFixed(1)), +(values.scale.y.toFixed(1)), +(values.scale.z.toFixed(1)));
            peep2.visible = values.visibility;
            peep2.scale.set(values.scale.x, values.scale.y, values.scale.z);
            peep3.visible = values.visibility;
            peep3.scale.set(values.scale.x, values.scale.y, values.scale.z);
        });


    }

</script>

<svelte:head>
	<title>CIP Explorer</title>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" >
    <link href="https://fonts.googleapis.com/css2?family=Inter:opsz,wght@14..32,100&display=swap" rel="stylesheet">
</svelte:head>

<style>
    :global(body) {
        margin: 0;
        padding: 0;
        height: 100%; /* Ensure html and body take full viewport height */
        width: 100%;  /* Ensure html and body take full viewport width */
        overflow: hidden; /* Prevent any scrollbars on the main page */
    }
</style>