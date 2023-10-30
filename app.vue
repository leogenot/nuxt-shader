<script setup>
//shader from: https://www.shadertoy.com/view/fllBzH
const vertex = `
attribute vec2 inPos;

void main() 
{
    gl_Position = vec4(inPos, 0.0, 1.0);
}`
const fragment = `
precision mediump float;

uniform vec2 iResolution;
uniform vec2 iMouse;
uniform float iTime;

vec3 lightDirection = normalize(vec3(0.5, 0.5, 1.0));
float nAir = 1.003;
float nWater = 1.333;
vec3 up = vec3(0.0, 0.0, 1.0);

vec4 water(float value) {
    vec3 col = vec3(0.8, 0.8, 1.0) * (0.1 * value);

    return vec4(col, 0.3);
}


void mainImage( out vec4 fragColor, in vec2 fragCoord )
{
    vec2 uv = fragCoord/iResolution.xy;
    vec2 bb = (fragCoord - (iResolution.xy/2.0)) / iResolution.y;
    
    float lambda1 = 7.0;
    
    float temp = lambda1 * (pow(bb.x, 2.0) + pow(bb.y, 2.0)) - 1.5 * iTime;
    
    float f = cos(temp) + 5.0;
    vec3 gx = vec3(1.0, 0.0, -sin(temp) * 2.0 * lambda1 * bb.x);
    vec3 gy = vec3(0.0, 1.0, -sin(temp) * 2.0 * lambda1 * bb.y);
    
    vec3 nVec = normalize(cross(gx, gy));
    if (dot(nVec, up) < 0.0) {
        nVec = -1.0 * nVec;
    }
 
    
    float diffuseCoeff = max(dot(nVec, lightDirection), 0.0);
    
    
    float alpha = acos(dot(up, nVec));
    float beta = asin(nAir * sin(alpha) / nWater);
    float gamma = alpha - beta;
    float dd = f * tan(gamma);
    float m = f * tan(alpha);
    vec2 nHorizontal = -nVec.xy;
    vec2 coords = bb - dd * nHorizontal / m;

    vec4 waterColor = water(diffuseCoeff);
    
    float ss = 0.0;
    if (f > 5.0) {
        ss = 1.0;
    }
    
    
    vec3 other = vec3(diffuseCoeff);

    fragColor = waterColor;
}

void main() 
{
    mainImage( gl_FragColor, gl_FragCoord.xy );
}
    `
</script>

<template>
  <div>
    <shader :vertex="vertex" :fragment="fragment" />
  </div>
</template>
