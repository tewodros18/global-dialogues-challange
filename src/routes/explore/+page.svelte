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
    import {featuresCity} from './locations/locations.js'
    import {shortSocialSummaries} from './locations/data.js';
    //@ts-ignore
    import {Text} from 'troika-three-text';
	import { CSS2DObject, CSS2DRenderer } from 'three/examples/jsm/renderers/CSS2DRenderer.js';

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


        // --- CSS2D Renderer ---
        const css2DRenderer = new CSS2DRenderer();
        css2DRenderer.setSize(window.innerWidth, window.innerHeight);
        css2DRenderer.domElement.style.position = 'absolute';
        css2DRenderer.domElement.style.top = '0px';
        css2DRenderer.domElement.style.pointerEvents = 'none';
        document.body.appendChild(css2DRenderer.domElement);

        // create an AudioListener and add it to the camera
        const listener = new THREE.AudioListener();
        camera.add(listener);

        // create a global audio source
        const sound = new THREE.Audio(listener);

        // load a sound and set it as the Audio object's buffer
        const audioLoader = new THREE.AudioLoader();
        
        if (!sound.isPlaying){
            audioLoader.load( 'audio/silent-movie-ragtime-clumsy-elegance-295463.mp3', function( buffer ) {
            sound.setBuffer( buffer );
            sound.setLoop( true );
            sound.setVolume( 0.5 );
            sound.play();
        });
        }
        
        var alltextures = [];
        const p = document.createElement('p');
        p.textContent = '';
        p.style.width = '18%';
        p.style.fontFamily  = 'Inter';
        p.style.fontSize = '1.2rem';
        p.style.color = 'black';
        p.style.fontWeight = '800';
        //@ts-ignore
        const peepIdea = new CSS2DObject(p);
        peepIdea.position.set(-2.28, 0, -2); // Position the text in front of the camera
        dollyNode.add(peepIdea);


        Object.keys(featuresCity).forEach(element => {
            alltextures.push(textureLoader.load(featuresCity[element].texture));
        });
        

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

        texture = textureLoader.load('home.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const home = new THREE.Mesh(
            new THREE.BoxGeometry(2.9,1.6,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        scene.add(home);
        home.position.set(-0.2, -1.5, -3);

        texture = textureLoader.load('hint.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const hint = new THREE.Mesh(
            new THREE.BoxGeometry(1.3,1,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        scene.add(hint);
        hint.position.set(0.6, -1.20, -3);
        //hint.rotation.z = Math.PI / 6; // Rotate the hint to face the camera


        /**
         * Walking peeps section
         * 
        */

        //@ts-ignore
        var peepsArray: int = [];

        const peepsGroup = new THREE.Group();
        var peepZindex = -2.5;
        var runningMinHeight = -1.1;
        for(let i = 0; i < 12; i++) {
            peepsArray.push(Math.random() * 0.007 + 0.004)
            texture = textureLoader.load(`./characters/peep-standing-${Math.floor(Math.random() * 30) + 1}.png`)
            texture.colorSpace = THREE.SRGBColorSpace;
            const randomHeight = Math.random() * (-1.5 - -1.85) + (-1.85); // Random height between (max - min)
            const peepInt1 = new THREE.Mesh(
                new THREE.BoxGeometry(0.5, 1.3, 0.0001),
                new THREE.MeshBasicMaterial({ map: texture, transparent: true })
            );
            peepInt1.name = `peep`;
            peepsGroup.add(peepInt1);
            peepInt1.position.set(-4.2, randomHeight, peepZindex);
            if(randomHeight < runningMinHeight) {
                runningMinHeight = randomHeight; // Update the minimum height
                peepZindex += 0.02;
            } else {
                peepZindex -= 0.01;
            }

        };
        
        
        peepsGroup.visible= false;
        dollyNode.add(peepsGroup);
        

        /**
         * Location section
         * **/

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

        texture = textureLoader.load('fast.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const post = new THREE.Mesh(
            new THREE.BoxGeometry(1,2,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );
        post.name = 'post';

        dollyNode.add(post);
        post.position.set(1.5, -0.7, -1.99);

        texture = textureLoader.load('mute.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const muteBtn = new THREE.Mesh(
            new THREE.BoxGeometry(0.3,0.3,0.000001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );
        muteBtn.name = 'mute';

        dollyNode.add(muteBtn);
        muteBtn.position.set(2.8, 1.33, -1.99);

        texture = textureLoader.load('adv.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const poster = new THREE.Mesh(
            new THREE.BoxGeometry(1,1.4,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        poster.name = 'poster'

        dollyNode.add(poster);
        poster.position.set(-2.3, 0.7, -1.99);

        texture = textureLoader.load('headline.png')
        texture.colorSpace = THREE.SRGBColorSpace;
        const news = new THREE.Mesh(
            new THREE.BoxGeometry(1.8,3,0.0001),
            new THREE.MeshBasicMaterial({ map: texture, transparent: true })
        );

        news.name = 'news'

        dollyNode.add(news);
        news.visible = false;
        news.position.set(-2.3, 0, -1.99);
        
        Object.keys(featuresCity).forEach((key) => {
            const currCity = featuresCity[key];
            console.log(currCity)
            const textureUnique = textureLoader.load(currCity.texture)
            textureUnique.colorSpace = THREE.SRGBColorSpace;
            const cityMesh = new THREE.Mesh(
                new THREE.BoxGeometry(currCity.sizeX,currCity.sizeY,currCity.sizeZ),
                new THREE.MeshBasicMaterial({ map: textureUnique, transparent: true })
            );
            scene.add(cityMesh);
            cityMesh.position.set(currCity.dollyPosX, currCity.dollyPosY, currCity.dollyPosZ);
        });


        window.addEventListener('resize', () => {
            console.log(window.innerWidth)
            renderer.setSize(window.innerWidth, window.innerHeight);
            css2DRenderer.setSize(window.innerWidth, window.innerHeight);
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix(); 
        });

        window.addEventListener('beforeunload', () => {
            // Stop the audio when the page is unloaded
            sound.stop();
        });

        function setCursor(className: string) {
        if (renderer.domElement.className !== className) { // Avoid unnecessary DOM manipulation
                renderer.domElement.className = className;
            }
        }

        
        window.addEventListener('click', (ev) => {
            const raycaster = new THREE.Raycaster();

            const mouseNDC = new THREE.Vector2(
                (ev.clientX / window.innerWidth) *  2 - 1,
                -(ev.clientY / window.innerHeight) *  2 + 1,
            );

            raycaster.setFromCamera(mouseNDC,camera);
        

            const intersections = raycaster.intersectObjects(scene.children);

            if (intersections.length > 0) {
                if(intersections[0].object.name == 'post'){
                    peepIdea.visible = false;
                    plane_sheet.sequence.play({iterationCount: 1}).finally(()=>{
                        p.innerHTML = "";
                        news.visible = false;
                        peepIdea.visible = true;
                        peepsGroup.visible = true;
                        dollyNode.position.x = dollyNode.position.x + 20;
                        if(dollyNode.position.x > 280){
                            dollyNode.position.x = 20;
                        }
                        plane_sheet.sequence.play({iterationCount: 1, direction: 'reverse', rate: 8})
                    });
                    cloud_bg_sheet.sequence.play({iterationCount: Infinity});
                    console.log("post");
                }
                if(intersections[0].object.name == 'mute'){
                    if(sound.isPlaying){
                        sound.stop();
                    } else {
                        sound.play();
                    }
                }
                if(intersections[2].object.name == 'peep' || intersections[1].object.name == 'peep'){
                    const randomItem = shortSocialSummaries[Math.floor(Math.random() * shortSocialSummaries.length)];
                    news.visible = true;
                    p.innerText = "";
                    p.innerText = `${randomItem.title} \n\n${randomItem.tweet}`;
                    console.log("peep")
                }
                else{
                    console.log(intersections)
                }
            }


        })

        
        function animate() {
            for (let i = 0; i < peepsGroup.children.length; i++ && peepsGroup.visible) {
                peepsGroup.children[i].position.x += peepsArray[i];
                if(peepsGroup.children[i].position.x > 4.3) {
                    peepsGroup.children[i].position.x = -4.2;
                    
                }
            }
            renderer.render(scene,camera);
            css2DRenderer.render(scene, camera);
        }
        

        studio.initialize();

        // Theatre.js setup

        const project = getProject("CIP Explorer");

        // sheet to animate the globe page
        const plane_sheet = project.sheet("plane");
        const cloud_bg_sheet = project.sheet("cloud_bg");


        // plane_sheet.sequence.attachAudio({ source: './audio/Plane_drive.wav' }).then(() => {
        //     console.log('Audio loaded!')
        // })

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

