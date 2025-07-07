<script lang="ts">
    import { browser } from '$app/environment'; 
	import * as THREE from "three"
    import { Reflector} from 'three/examples/jsm/objects/Reflector.js';
    import * as TWEEN from '@tweenjs/tween.js';
    import { onMount } from 'svelte';
    import * as theatrecore from '@theatre/core';
    import studio from '@theatre/studio'
    const { Tween, Easing, update } = TWEEN;
    const { getProject, types} = theatrecore;

    if(browser){
        
        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setAnimationLoop(animate);
        document.body.appendChild(renderer.domElement);

        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);


        // Three js objects

        const geometry = new THREE.BoxGeometry(1, 1, 1);
        const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
        const cube = new THREE.Mesh(geometry, material);
        scene.add(cube);


        window.addEventListener('resize', () => {
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix(); 
        });

        camera.position.z = 5;

        function animate() {
            renderer.render(scene,camera);
        }


        studio.initialize();

        // Theatre.js setup

        const project = getProject("CIP Explorer");

        const globe_sheet = project.sheet("Globe");

        const boxObj = globe_sheet.object("Box", {
            rotation: types.compound({
                x: types.number(cube.rotation.x, { range: [-2, 2] }),
                y: types.number(cube.rotation.y, { range: [-2, 2] }),
                z: types.number(cube.rotation.z, { range: [-2, 2] }),
            }),
        });
        
        boxObj.onValuesChange((values) => {
            const { x, y, z } = values.rotation
            cube.rotation.set(x * Math.PI, y * Math.PI, z * Math.PI)
        })

        

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