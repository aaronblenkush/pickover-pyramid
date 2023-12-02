<script>
import { T, useTask } from '@threlte/core';
import {
    Grid,
    MeshLineGeometry,
    MeshLineMaterial,
    OrbitControls,
} from '@threlte/extras';
import { CatmullRomCurve3, Vector3 } from 'three';
import { onMount } from 'svelte';

function makePoints(points,coords=[]) {
    let vertices = [];
    while(coords.length > 0) {
        vertices.push(Reflect.construct(Vector3,coords.slice(0,3)));
        coords = coords.slice(3);
    }
    const curve = new CatmullRomCurve3(vertices);
    return curve.getPoints(points);
}

let lines = [
    [60,-2, 0,-2,-3, 0, 0,-2, 0, 2,], // CL
    [2, -2, 0, 2,-2, 0,-2,],          // L
    [2,  0, 4, 0,-2, 0, 2,],          // SW
    [2, -2, 0,-2, 0, 4, 0,],          // NW
    [60, 2, 0,-2, 0, 0,-3,-2, 0,-2,], // CT
    [2, -2, 0,-2, 2, 0,-2,],          // T
    [2,  2, 0, 2,-2, 0,-2,],          // \
    [2, -2, 0, 2, 2, 0, 2,],          // B
    [60, 2, 0, 2, 0, 0, 3,-2, 0, 2,], // CB
    [2,  2, 0,-2, 2, 0, 2,],          // R
    [2,  0, 4, 0, 2, 0,-2,],          // NE
    [2,  2, 0, 2, 0, 4, 0,],          // SE
    [60, 2, 0,-2, 3, 0, 0, 2, 0, 2,], // CR
    [ 2,-2, 0, 2, 2, 0,-2,],          // /
];

let renderedLines = [];
let dashRatio = [];
let camera;
let draw = true;
let ctl;
let rotate = 15;

function advance() {
    if (renderedLines.length == lines.length) {
        draw = false;
        if (ctl) clearInterval(ctl);
    }
    let i = renderedLines.length+1;
    renderedLines = lines.slice(0,i);
    dashRatio[i-1] = 1;
}

function delay(ms) {
    return new Promise(resolve => setTimeout(resolve,ms));
}

onMount(async () => {
    await delay(1000);
    ctl = setInterval(advance,500);
    return () => {
        clearInterval(ctl);
    }
});

useTask((delta) => {
    if (draw) {
        let i = renderedLines.length - 1;
        for (let i=0;i<renderedLines.length;i++) {
            if (dashRatio[i] > 0) {
                dashRatio[i] = Math.max(0,dashRatio[i] - delta * 3);
            }
        }
    } else {
        if (rotate > 0) {
            rotate = Math.max(0,rotate - delta * 10);
            camera.position.set(0,15,rotate);
        }
    }
});

</script>

{#each renderedLines as [sections,...line],i}
{@const points = makePoints(sections,line)}
<T.Mesh position.y={0}>
    <MeshLineGeometry {points} />
    <MeshLineMaterial
        color="#0000ff"
        width={0.1}
        dashArray={1}
        dashRatio={dashRatio[i]}
        dashOffset=0
        depthTest={false}
        transparent
    />
</T.Mesh>
{/each}

<T.PerspectiveCamera
    makeDefault
    fov={30}
    on:create={({ ref }) => {
        camera = ref;
        ref.position.set(0, 15, 15);
    }}
>
    <OrbitControls
        autoRotate={false}
        autoRotateSpeed={0}
        enableDamping
        target.y={2}
    />
</T.PerspectiveCamera>

<T.DirectionalLight
    position={[0, 10, 10]}
    castShadow
/>

<Grid
    gridSize={[20, 20]}
    cellColor={"#46536b"}
    sectionThickness={0}
/>
