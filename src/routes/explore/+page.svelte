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
        var texture = textureLoader.load('globe.png')
        texture.colorSpace = THREE.SRGBColorSpace; // Ensure the texture is in sRGB color space
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


        texture = textureLoader.load('camera.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const camUI = new THREE.Mesh(
            new THREE.BoxGeometry(5, 3.5, 0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );// a UI element to represent the caera view
        camUI.position.z = -2; 
        camUI.visible = false;
        dollyNode.add(camUI);

        texture = textureLoader.load('./plane/plane2.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const plane = new THREE.Mesh(
            new THREE.BoxGeometry(2.2,2.2,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );
    
        plane.rotation.z = Math.PI / 2; // Rotate the plane to face the camera
        plane.position.z = -1.8; 
        plane.position.x = 6; //6
        plane.position.y = -3;// -3Position the plane in front of the camera
        
        texture = textureLoader.load('./plane/cloud.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const cloud1 = new THREE.Mesh(
            new THREE.BoxGeometry(1.2, 1.2, 0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        cloud1.position.z = -1.9; 
        cloud1.position.x = 6; //6
        cloud1.position.y = -3;// -3Position the plane in front of the camera
        cloud1.rotation.z = -Math.PI / 2; // Rotate the plane to face the camera

        texture = textureLoader.load('./plane/cloud1.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const cloud2 = new THREE.Mesh(
            new THREE.BoxGeometry(2, 0.8, 0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        texture = textureLoader.load('./plane/sky.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const sky = new THREE.Mesh(
            new THREE.BoxGeometry(10, 10, 0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        texture = textureLoader.load('./plane/stamp1.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const stamp = new THREE.Mesh(
            new THREE.BoxGeometry(1, 0.8, 0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        texture = textureLoader.load('./plane/stamp2.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const stamp1 = new THREE.Mesh(
            new THREE.BoxGeometry(1, 0.8, 0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

    
        dollyNode.add(plane);
        dollyNode.add(cloud1);
        dollyNode.add(cloud2);
        dollyNode.add(sky);
        dollyNode.add(stamp);
        dollyNode.add(stamp1);


        /**
         * Walking peeps section
         * 
        */
        const peepsArray = [Math.random() * 0.009 + 0.004,Math.random() * 0.009 + 0.004,Math.random() * 0.009 + 0.004];
        const peepsGroup = new THREE.Group();
        var peepZindex = -2.5;
        var runningMinHeight = -1.1;
        peepsArray.forEach(element => {
            texture = textureLoader.load('./characters/peep-16.png')
            texture.colorSpace = THREE.SRGBColorSpace;
            const randomHeight = Math.random() * (-1.1 - -1.5) + (-1.5); // Random height between (max - min)
            const peepInt1 = new THREE.Mesh(
                new THREE.BoxGeometry(0.9, 1.3, 0.0001),
                new THREE.MeshBasicMaterial({ map: texture, transparent: true })
            );
            peepsGroup.add(peepInt1);
            peepInt1.position.set(-4.2, randomHeight, peepZindex);
            if(randomHeight < runningMinHeight) {
                runningMinHeight = randomHeight; // Update the minimum height
                peepZindex += 0.02;
            } else {
                peepZindex -= 0.01;
            }

        });
        
        
        //peepsGroup.visible= false;
        dollyNode.add(peepsGroup);
        


        /**
         * Globe page section
         */
        
        const globeBaseNode = new THREE.Object3D();
        scene.add(globeBaseNode);

        texture = textureLoader.load('globe.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const globe = new THREE.Mesh(
            new THREE.BoxGeometry(3, 3, 0.01),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true})
        )

        globe.position.z = -8; 
        globeBaseNode.add(globe);

        texture = textureLoader.load('./characters/peep-standing-16.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const peep = new THREE.Mesh(
            new THREE.BoxGeometry(0.25,0.5,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        scene.add(peep);
        peep.position.set(0.55, 0.35, -2.5);

        const peep2 = new THREE.Mesh(
            new THREE.BoxGeometry(0.25,0.5,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        scene.add(peep2);
        peep2.position.set(-0.29, -0.25, -2.5);



        const peep3 = new THREE.Mesh(
            new THREE.BoxGeometry(0.25,0.5,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        scene.add(peep3);
        peep3.position.set(-0.65, 0.35, -2.5);

        /**
         * Location section
         * **/

        dollyNode.position.x = 40;
        camUI.visible = false;

        texture = textureLoader.load('./plane/sky.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const locSky = new THREE.Mesh(
            new THREE.BoxGeometry(10,10,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        dollyNode.add(locSky);
        locSky.position.set(0, 0, -3.1);

        texture = textureLoader.load('./clouds/gcloud1.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const locCloud1 = new THREE.Mesh(
            new THREE.BoxGeometry(1,0.5,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );
        dollyNode.add(locCloud1);
        locCloud1.position.set(2, 1.5, -2.9);

        texture = textureLoader.load('./clouds/gcloud3.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const locCloud2 = new THREE.Mesh(
            new THREE.BoxGeometry(3,0.8,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        dollyNode.add(locCloud2);
        locCloud2.position.set(-3.1, 1, -3.1);    


        texture = textureLoader.load('wall.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const wall = new THREE.Mesh(
            new THREE.BoxGeometry(8,4,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        dollyNode.add(wall);
        wall.position.set(0.5, 0, -2);

        texture = textureLoader.load('./location/brazil.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const locForeground = new THREE.Mesh(
            new THREE.BoxGeometry(5,4,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        scene.add(locForeground);
        locForeground.position.set(20, -0.3, -3);

    

        updateCameraPosition(globeBaseNode);
        function updateCameraPosition(Node: THREE.Object3D) {
            camera.position.set(Node.position.x, Node.position.y, Node.position.z);
            //camera.lookAt(globeBaseNode.position);
        }

        window.addEventListener('resize', () => {
            console.log(window.innerWidth)
            renderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix(); 
        });

    
        function animate() {
            for (let i = 0; i < peepsGroup.children.length; i++ && peepsGroup.visible) {
                peepsGroup.children[i].position.x += peepsArray[i];
                if(peepsGroup.children[i].position.x > 4.3) {
                    peepsGroup.children[i].position.x = -4.2;
                    
                }
            }
            renderer.render(scene,camera);
        }



        

        studio.initialize();

        // Theatre.js setup

        const project = getProject("CIP Explorer");

        // sheet to animate the globe page
        const globe_sheet = project.sheet("Globe");
        const plane_sheet = project.sheet("plane");
        const cloud_bg_sheet = project.sheet("cloud_bg");


        globe_sheet.sequence.attachAudio({ source: './audio/intro.wav' }).then(() => {
            console.log('Audio loaded!')
        })

        plane_sheet.sequence.attachAudio({ source: './audio/Plane_drive.wav' }).then(() => {
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
            peep2.scale.set(+(values.scale.x.toFixed(1)), +(values.scale.y.toFixed(1)), +(values.scale.z.toFixed(1)));
            peep3.visible = values.visibility;
            peep3.scale.set(+(values.scale.x.toFixed(1)), +(values.scale.y.toFixed(1)), +(values.scale.z.toFixed(1)));
        });

        const planeObj = plane_sheet.object("plane", {
            position: types.compound({
                x: types.number(plane.position.x, { range: [-10, 6] }),
                y: types.number(plane.position.y, { range: [-10, 3] }),
                z: types.number(plane.position.z, { range: [-10, 10] }),
            }),
        })

        planeObj.onValuesChange((values) => {
            plane.position.set(values.position.x, values.position.y, values.position.z);
        });

        const cloud1Obj = plane_sheet.object("cloud 1", {
            position: types.compound({
                x: types.number(cloud1.position.x, { range: [-10, 6] }),
                y: types.number(cloud1.position.y, { range: [-10, 3] }),
                z: types.number(cloud1.position.z, { range: [-10, 10] }),
            }),
        })

        cloud1Obj.onValuesChange((values) => {
            cloud1.position.set(values.position.x, values.position.y, values.position.z);
        });

        const cloud2Obj = plane_sheet.object("cloud 2", {
            position: types.compound({
                x: types.number(cloud1.position.x, { range: [-10, 6] }),
                y: types.number(cloud1.position.y, { range: [-10, 3] }),
                z: types.number(cloud1.position.z, { range: [-10, 10] }),
            }),
        })

        cloud2Obj.onValuesChange((values) => {
            cloud2.position.set(values.position.x, values.position.y, values.position.z);
        });

        const skyObj = plane_sheet.object("sky", {
            position: types.compound({
                x: types.number(cloud1.position.x, { range: [-10, 6] }),
                y: types.number(cloud1.position.y, { range: [-10, 3] }),
                z: types.number(cloud1.position.z, { range: [-10, 10] }),
            }),
        })

        skyObj.onValuesChange((values) => {
            sky.position.set(values.position.x, values.position.y, values.position.z);
        });

        const stampObj = plane_sheet.object("stamp", {
            position: types.compound({
                x: types.number(cloud1.position.x, { range: [-10, 6] }),
                y: types.number(cloud1.position.y, { range: [-10, 3] }),
                z: types.number(cloud1.position.z, { range: [-10, 10] }),
            }),
            rotation: types.compound({
                x: types.number(stamp.rotation.x, { range: [-2, 2] }),
                y: types.number(stamp.rotation.y, { range: [-2, 2] }),
                z: types.number(stamp.rotation.z, { range: [-2, 2] }),
            }),
        })

        stampObj.onValuesChange((values) => {
            stamp.position.set(values.position.x, values.position.y, values.position.z);
            stamp.rotation.set(values.rotation.x * Math.PI, values.rotation.y * Math.PI, values.rotation.z * Math.PI);
        });

        const stamp1Obj = plane_sheet.object("stamp1", {
            position: types.compound({
                x: types.number(cloud1.position.x, { range: [-10, 6] }),
                y: types.number(cloud1.position.y, { range: [-10, 3] }),
                z: types.number(cloud1.position.z, { range: [-10, 10] }),
            }),
            rotation: types.compound({
                x: types.number(stamp.rotation.x, { range: [-2, 2] }),
                y: types.number(stamp.rotation.y, { range: [-2, 2] }),
                z: types.number(stamp.rotation.z, { range: [-2, 2] }),
            }),
        })

        stamp1Obj.onValuesChange((values) => {
            stamp1.position.set(values.position.x, values.position.y, values.position.z);
            stamp1.rotation.set(values.rotation.x * Math.PI, values.rotation.y * Math.PI, values.rotation.z * Math.PI);
        });

        const dollyCloud1 = cloud_bg_sheet.object("cloudbg1", {
            position: types.compound({
                x: types.number(locCloud1.position.x, { range: [-10, 6] }),
                y: types.number(locCloud1.position.y, { range: [-10, 3] }),
                z: types.number(locCloud1.position.z, { range: [-10, 10] }),
            }),
        })

        dollyCloud1.onValuesChange((values) => {
            locCloud1.position.set(values.position.x, values.position.y, values.position.z);
        });

        const dollyCloud2 = cloud_bg_sheet.object("cloudbg2", {
            position: types.compound({
                x: types.number(locCloud2.position.x, { range: [-10, 6] }),
                y: types.number(locCloud2.position.y, { range: [-10, 3] }),
                z: types.number(locCloud2.position.z, { range: [-10, 10] }),
            }),
        })

        dollyCloud2.onValuesChange((values) => {
            locCloud2.position.set(values.position.x, values.position.y, values.position.z);
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