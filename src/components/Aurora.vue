<template>
    <canvas ref="canvas" class="aurora-canvas"></canvas>
</template>

<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref, watch } from "vue";
import { Color, Geometry, Mesh, Program, Renderer, Vec2 } from "ogl";

const props = withDefaults(
    defineProps<{
        colorStops: string[];
        amplitude?: number;
        blend?: number;
        speed?: number;
        intensity?: number;
    }>(),
    {
        amplitude: 2,
        blend: 0.1,
        speed: 2,
        intensity: 1,
    },
);

const canvas = ref<HTMLCanvasElement | null>(null);
let renderer: Renderer | null = null;
let program: Program | null = null;
let mesh: Mesh | null = null;
let rafId = 0;
let resizeObserver: ResizeObserver | null = null;

const vertex = `
attribute vec2 position;
attribute vec2 uv;
varying vec2 vUv;

void main() {
  vUv = uv;
  gl_Position = vec4(position, 0.0, 1.0);
}
`;

const fragment = `
precision highp float;

uniform float uTime;
uniform vec3 uColorA;
uniform vec3 uColorB;
uniform vec3 uColorC;
uniform float uAmplitude;
uniform float uBlend;
uniform float uSpeed;
uniform float uIntensity;
uniform vec2 uResolution;

varying vec2 vUv;

float hash(vec2 p) {
  return fract(sin(dot(p, vec2(127.1, 311.7))) * 43758.5453);
}

float noise(vec2 p) {
  vec2 i = floor(p);
  vec2 f = fract(p);
  float a = hash(i);
  float b = hash(i + vec2(1.0, 0.0));
  float c = hash(i + vec2(0.0, 1.0));
  float d = hash(i + vec2(1.0, 1.0));
  vec2 u = f * f * (3.0 - 2.0 * f);
  return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
}

void main() {
  float t = uTime * 0.4 * uSpeed;
  vec2 st = vUv * vec2(uResolution.x / uResolution.y, 1.0);
  float n = noise(st * 3.0 + vec2(0.0, t));
  float wave = sin((vUv.y + n * 0.4 + t) * 6.2831 * uAmplitude);
  float band = smoothstep(0.0, 1.0, vUv.y + wave * 0.08);
  vec3 col = mix(uColorA, uColorB, band);
  float sweep = smoothstep(0.1, 0.9, vUv.x + n * 0.25);
  col = mix(col, uColorC, sweep);
  float glow = pow(1.0 - abs(vUv.y - 0.5) * 2.0, 2.0);
  col += glow * 0.12 * uIntensity;
  gl_FragColor = vec4(col, uBlend);
}
`;

function hexToColor(hex: string) {
    const value = hex.replace("#", "");
    const size = value.length === 3 ? 1 : 2;
    const r =
        parseInt(value.slice(0, size).repeat(size === 1 ? 2 : 1), 16) / 255;
    const g =
        parseInt(value.slice(size, size * 2).repeat(size === 1 ? 2 : 1), 16) /
        255;
    const b =
        parseInt(
            value.slice(size * 2, size * 3).repeat(size === 1 ? 2 : 1),
            16,
        ) / 255;
    return new Color(r, g, b);
}

function updateColors() {
    if (!program) return;
    const stops = props.colorStops.length
        ? props.colorStops
        : ["#ffffff", "#000000", "#ffffff"];
    program.uniforms.uColorA.value = hexToColor(stops[0]);
    program.uniforms.uColorB.value = hexToColor(stops[1] ?? stops[0]);
    program.uniforms.uColorC.value = hexToColor(
        stops[2] ?? stops[1] ?? stops[0],
    );
}

function resize() {
    if (!renderer || !canvas.value) return;
    const parent = canvas.value.parentElement;
    if (!parent) return;
    const width = parent.clientWidth;
    const height = parent.clientHeight;
    renderer.setSize(width, height);
    if (program) {
        program.uniforms.uResolution.value.set(width, height);
    }
}

onMounted(() => {
    if (!canvas.value) return;

    renderer = new Renderer({
        canvas: canvas.value,
        dpr: Math.min(window.devicePixelRatio, 2),
        alpha: true,
    });

    const gl = renderer.gl;
    gl.clearColor(1, 1, 1, 1);

    const geometry = new Geometry(gl, {
        position: {
            size: 2,
            data: new Float32Array([-1, -1, 3, -1, -1, 3]),
        },
        uv: {
            size: 2,
            data: new Float32Array([0, 0, 2, 0, 0, 2]),
        },
    });

    program = new Program(gl, {
        vertex,
        fragment,
        uniforms: {
            uTime: { value: 0 },
            uColorA: { value: hexToColor(props.colorStops[0] ?? "#ffffff") },
            uColorB: { value: hexToColor(props.colorStops[1] ?? "#000000") },
            uColorC: { value: hexToColor(props.colorStops[2] ?? "#ffffff") },
            uAmplitude: { value: props.amplitude },
            uBlend: { value: props.blend },
            uSpeed: { value: props.speed },
            uIntensity: { value: props.intensity },
            uResolution: { value: new Vec2(1, 1) },
        },
    });

    mesh = new Mesh(gl, { geometry, program });

    const loop = (time: number) => {
        if (!renderer || !program || !mesh) return;
        program.uniforms.uTime.value = time * 0.001;
        renderer.render({ scene: mesh });
        rafId = requestAnimationFrame(loop);
    };

    resize();
    rafId = requestAnimationFrame(loop);

    resizeObserver = new ResizeObserver(resize);
    resizeObserver.observe(canvas.value);
});

watch(
    () => props.colorStops,
    () => updateColors(),
    { deep: true },
);

watch(
    () => props.amplitude,
    (value) => {
        if (program) program.uniforms.uAmplitude.value = value;
    },
);

watch(
    () => props.blend,
    (value) => {
        if (program) program.uniforms.uBlend.value = value;
    },
);

watch(
    () => props.speed,
    (value) => {
        if (program) program.uniforms.uSpeed.value = value;
    },
);

watch(
    () => props.intensity,
    (value) => {
        if (program) program.uniforms.uIntensity.value = value;
    },
);

onBeforeUnmount(() => {
    if (rafId) cancelAnimationFrame(rafId);
    resizeObserver?.disconnect();
    renderer?.gl.getExtension("WEBGL_lose_context")?.loseContext();
    renderer = null;
    program = null;
    mesh = null;
});
</script>

<style scoped>
.aurora-canvas {
    width: 100%;
    height: 100%;
    display: block;
}
</style>
