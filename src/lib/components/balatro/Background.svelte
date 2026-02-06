<script lang="ts">
    import { T, useTask } from "@threlte/core";
    import { Vector2, Color } from "three";

    interface Props {
        spinSpeed?: number;
        spinAmount?: number;
        contrast?: number;
        spinRotation?: number;
        pixelFilter?: number;
        uvScale?: number;
    }

    let {
        spinSpeed = 0.75,
        spinAmount = 1.5,
        contrast = 2.0,
        spinRotation = 0.5,
        pixelFilter = 3500.0,
        uvScale = 125.0
    }: Props = $props();

    let time = 0;

    const uniforms = {
        uTime: { value: 0 },
        uColor1: { value: new Color("#ff0066") }, // Balatro Pink/Red
        uColor2: { value: new Color("#00aaff") }, // Balatro Blue
        uColor3: { value: new Color("#ffaa00") }, // Balatro Gold/Orange
        uSpinRotation: { value: spinRotation },
        uSpinSpeed: { value: spinSpeed },
        uSpinAmount: { value: spinAmount },
        uContrast: { value: contrast },
        uPolarCoordinates: { value: false },
        uPolarCenter: { value: new Vector2(0.5, 0.5) },
        uPolarZoom: { value: 1.0 },
        uPolarRepeat: { value: 1.0 },
        uPixelFilter: { value: pixelFilter },
        uUvScale: { value: uvScale }
    };

    // Update uniforms when props change
    $effect(() => {
        uniforms.uSpinSpeed.value = spinSpeed;
        uniforms.uSpinAmount.value = spinAmount;
        uniforms.uContrast.value = contrast;
        uniforms.uSpinRotation.value = spinRotation;
        uniforms.uPixelFilter.value = pixelFilter;
        uniforms.uUvScale.value = uvScale;
    });

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
    uniform vec3 uColor3;
    uniform float uSpinRotation;
    uniform float uSpinSpeed;
    uniform float uSpinAmount;
    uniform float uContrast;
    uniform bool uPolarCoordinates;
    uniform vec2 uPolarCenter;
    uniform float uPolarZoom;
    uniform float uPolarRepeat;
    uniform float uPixelFilter;
    uniform float uUvScale;

    varying vec2 vUv;

    const float SPIN_EASE = 1.0;
    const float PI = 3.14159265359;

    vec4 effect(vec2 screenSize, vec2 screenCoords) {
      // Convert to UV coords (0-1) and floor for pixel effect
      float pixel_size = length(screenSize.xy) / uPixelFilter;
      vec2 uv = (floor(screenCoords.xy * (1.0 / pixel_size)) * pixel_size - 0.5 * screenSize.xy) / length(screenSize.xy);
      float uv_len = length(uv);

      // Adding in a center swirl, changes with time
      float speed = (uSpinRotation * SPIN_EASE * 0.2) + 302.2;
      float new_pixel_angle = (atan(uv.y, uv.x)) + speed - SPIN_EASE * 20.0 * (uSpinAmount * uv_len + (1.0 - uSpinAmount));
      vec2 mid = (screenSize.xy / length(screenSize.xy)) / 2.0;
      uv = vec2((uv_len * cos(new_pixel_angle) + mid.x), (uv_len * sin(new_pixel_angle) + mid.y)) - mid;

      // Now add the paint effect to the swirled UV
      uv *= uUvScale;
      speed = uTime * uSpinSpeed;
      vec2 uv2 = vec2(uv.x + uv.y);

      for(int i = 0; i < 5; i++) {
        uv2 += sin(max(uv.x, uv.y)) + uv;
        uv += 0.5 * vec2(cos(5.1123314 + 0.353 * uv2.y + speed * 0.131121), sin(uv2.x - 0.113 * speed));
        uv -= 1.0 * cos(uv.x + uv.y) - 1.0 * sin(uv.x * 0.711 - uv.y);
      }

      // Make the paint amount range from 0 - 2
      float contrast_mod = (0.25 * uContrast + 0.5 * uSpinAmount + 1.2);
      float paint_res = min(2.0, max(0.0, length(uv) * 0.035 * contrast_mod));
      float c1p = max(0.0, 1.0 - contrast_mod * abs(1.0 - paint_res));
      float c2p = max(0.0, 1.0 - contrast_mod * abs(paint_res));
      float c3p = 1.0 - min(1.0, c1p + c2p);

      vec4 ret_col = (0.3 / uContrast) * vec4(uColor1, 1.0) + (1.0 - 0.3 / uContrast) * (vec4(uColor1, 1.0) * c1p + vec4(uColor2, 1.0) * c2p + vec4(uColor3 * c3p, c3p));
      return ret_col;
    }

    vec2 polar_coords(vec2 uv, vec2 center, float zoom, float repeat) {
      vec2 dir = uv - center;
      float radius = length(dir) * 2.0;
      float angle = atan(dir.y, dir.x) * 1.0 / (PI * 2.0);
      return mod(vec2(radius * zoom, angle * repeat), 1.0);
    }

    void main() {
      vec2 uv = vUv;

      if (uPolarCoordinates) {
        uv = polar_coords(vUv, uPolarCenter, uPolarZoom, uPolarRepeat);
      }

      vec2 screenSize = vec2(1.0, 1.0);
      gl_FragColor = effect(screenSize, uv);
    }
  `;
</script>

<T.Mesh scale={100} position={[0, 0, -5]}>
    <T.PlaneGeometry />
    <T.ShaderMaterial {vertexShader} {fragmentShader} {uniforms} />
</T.Mesh>
