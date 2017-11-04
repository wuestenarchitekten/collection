# LED Cube Render
The basic Idea is to render each slice of the cube from 3 directions. 
In the given example with a pixel resolution of 10x10x10 you end up with 30 renders.
Each render uses a seperate camera that is individually positioned and comes with a very small Near and Far plane.

All the renders from one axis are fed into a GLSL TOP with some simple code:

```
// Example Pixel Shader

// uniform float exampleUniform;

vec2 map(vec2 value, vec2 inMin, vec2 inMax, vec2 outMin, vec2 outMax)
{
	return outMin + (outMax - outMin) * (value - inMin) / (inMax - inMin);
}

out vec4 fragColor;
void main()
{
    vec4 pos = texture(sTD2DInputs[0], vUV.st);
    
    vec2 pixelSize = uTD2DInfos[1].res.xy;
    pos.zy = map(pos.zy, vec2(0.0),vec2(1.0),vec2(0.0),vec2(1.0)-pixelSize)+pixelSize/2.0;
    
    vec4 color = texture(sTD2DInputs[int(pos.x)+1],pos.zy);
    
    float value = dot(color.rgb,vec3(1.0));
    color.a = min(ceil(value),1.0);
    
    fragColor = TDOutputSwizzle(color);
}

```
The code shown here is the lookup into the x-Axis renderers.
The first input to the GLSL TOP is all the LED positions as rgb color values.
So the red value of any given pixel describes the x position of the LED and analogous the green and blue values describe the z and y position of the LED light.
Since I'm rendering in this example down the x-Axis, each of the 10 renders is one slice and for each LED described in the first input, the red value will be an index to which slice to sample from while the green and blue values are the uv coordinates in that slice.

Doing this for all 3 Axis and adding up the outputs should give a good approximation of which LEDs need to be turned on.
