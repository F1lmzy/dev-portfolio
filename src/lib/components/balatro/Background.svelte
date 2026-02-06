<script lang="ts">
  import { T, useTask } from '@threlte/core';
  import { Vector2, Color } from 'three';

  let time = 0;
  
  const uniforms = {
    uTime: { value: 0 },
    uResolution: { value: new Vector2(1, 1) },
    uColor1: { value: new Color('#ff4d4d') }, // Bright Red (Balatro Red)
    uColor2: { value: new Color('#0044ff') }  // Bright Blue for contrast
  };

  useTask((delta) => {
    time += delta;
    uniforms.uTime.value = time;
  });

  const vertexShader = `
    varying vec2 vUv;
    void main() {
      vUv = uv;
      gl_Position = projectionMatrix * modelViewMatrix * vec4(position, 1.0);
    }
  `;

  const fragmentShader = `
    uniform float uTime;
    uniform vec3 uColor1;
    uniform vec3 uColor2;
    varying vec2 vUv;

    // Simple pseudo-random noise
    float hash(vec2 p) { return fract(1e4 * sin(17.0 * p.x + p.y * 0.1) * (0.1 + abs(sin(p.y * 13.0 + p.x)))); }

    float noise(vec2 x) {
      vec2 i = floor(x);
      vec2 f = fract(x);
      float a = hash(i);
      float b = hash(i + vec2(1.0, 0.0));
      float c = hash(i + vec2(0.0, 1.0));
      float d = hash(i + vec2(1.0, 1.0));
      vec2 u = f * f * (3.0 - 2.0 * f);
      return mix(a, b, u.x) + (c - a) * u.y * (1.0 - u.x) + (d - b) * u.x * u.y;
    }

    float fbm(vec2 x) {
      float v = 0.0;
      float a = 0.5;
      vec2 shift = vec2(100);
      mat2 rot = mat2(cos(0.5), sin(0.5), -sin(0.5), cos(0.50));
      for (int i = 0; i < 5; ++i) {
        v += a * noise(x);
        x = rot * x * 2.0 + shift;
        a *= 0.5;
      }
      return v;
    }

    void main() {
      vec2 p = vUv * 2.0 - 1.0;
      float t = uTime * 0.2;
      
      // Swirl distortion
      float r = length(p);
      float angle = atan(p.y, p.x);
      float f = fbm(vec2(r * 3.0 - t, angle * 2.0 + t));
      
      vec3 color = mix(uColor2, uColor1, f);
      
      // Vignette/Darken edges
      color *= 1.0 - smoothstep(0.5, 1.5, r);

      gl_FragColor = vec4(color, 1.0);
    }
  `;
</script>

<T.Mesh scale={100} position={[0, 0, -5]}>
  <T.PlaneGeometry />
  <T.ShaderMaterial
    {vertexShader}
    {fragmentShader}
    {uniforms}
  />
</T.Mesh>
