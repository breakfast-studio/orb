<template>
    <Lunchbox :cameraPosition="[0.5, 0, 2.8]" :cameraLook="[0, 0, 0]">
        <mesh v-for="(v, i) in verts" :scale="1 + getScale(i) * 0.02" :key="i">
            <bufferGeometry @added="geoAdded($event, i)" />
            <meshBasicMaterial
                :color="`hsl(0, 0%, ${
                    Math.floor((getScale(i) * 0.5 + 0.5) * 25) + 15
                }%)`"
                :side="THREE.DoubleSide"
            />

            <mesh @added="meshAdded" :scale="1.001">
                <bufferGeometry @added="geoAdded($event, i)" />
                <meshBasicMaterial color="black" wireframe />
            </mesh>
        </mesh>

        <mesh :scale="0.95">
            <icosahedronGeometry :args="[1, 3]" />
            <meshBasicMaterial color="aqua" />
        </mesh>
    </Lunchbox>
</template>

<script setup lang="ts">
import { onBeforeRender, setCustomRender } from 'lunchboxjs'
import { ref } from 'vue'
import * as THREE from 'three'

// time
const now = ref(Date.now())
onBeforeRender(() => (now.value = Date.now()))

// vertices (any THREE geometry should work here)
const geo = new THREE.IcosahedronGeometry(1, 3)
const verts = splitToChunks(Array.from(geo.attributes.position.array), 9)

// buffer geo
const geoAdded = (
    { instance }: { instance: THREE.BufferGeometry },
    idx: number
) => {
    instance.setAttribute(
        'position',
        new THREE.BufferAttribute(new Float32Array(verts[idx]), 3)
    )
}

// scale function (used for color and position)
const getScale = (idx: number) => {
    return Math.sin(
        now.value * 0.002 + idx * 0.5 * Math.sin(now.value * 0.0001)
    )
}

// misc
function splitToChunks<T>(array: Array<T>, chunkLength: number) {
    const copy = [...array]
    const output: T[][] = []
    for (let i = 0; i < Math.ceil(array.length / chunkLength); i++) {
        output.push(copy.splice(0, chunkLength))
    }
    return output
}

// postprocessing
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer'
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass'
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass'

let composer: EffectComposer | null = null

setCustomRender((opts) => {
    const canvas = opts.renderer?.domElement

    // ignore if no canvas
    if (!canvas || !opts.scene || !opts.camera) return

    // setup effect composer if needed
    if (!composer) {
        composer = new EffectComposer(opts.renderer as THREE.WebGLRenderer)

        const renderPass = new RenderPass(opts.scene, opts.camera)
        composer.addPass(renderPass)

        const bloomPass = new UnrealBloomPass(
            new THREE.Vector2(canvas.width, canvas.height),
            0.55,
            0.1,
            0.1
        )
        composer.addPass(bloomPass)
    }

    composer.render()
})
</script>