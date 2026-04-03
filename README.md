# CS4300-site

[Cube](https://www.youtube.com/watch?v=VgvrWujcPxk)

# Description:
This was relatively small. Partially due to me:
- Not having a lot of time
- Being a bit new to WGSL
- Pull my memory of 3D graphics from the last time I coded that. Which was around a year ago.
Part of the design for this came from the youtuber `tsoding` who did GPU-less 3D rendering. Notably, webgpu does, in fact, allow the usage of a GPU, but the techniques that I used were similar to GPU-less. In the vein of drawing lines to a framebuffer in the points of a mesh. All from entirely within one fragment shader. GPU-less style on the GPU.
It didn’t help that several times upon trying to write this the tab was accidentally (or on some foolish occasions, intentionally) closed. Aaaaand then I remembered that the editor didn’t save the text. On the plus side, I got more familiar with WGSL after repeatedly writing it several times. By the way, if you would like me to make an IDE-esque thing (which could do things like auto-saving), I would genuinely be happy to do so. I’ve done things like that in the past, and doing that with WGSL would seem pretty doable. 

# Feedback from classmate:
They thought it was neat. Though moreso because of it being kind of wack visa vi it being GPU-less-esque rather than finding the actual animation itself neat (since a rotating cube isn’t like… super new to them). Like, it being 3D in only a fragment shader and with no vertex shader.

# Code
```wgsl
fn sdSegment(p: vec2f, a: vec2f, b: vec2f) -> f32 {
    let pa = p-a;
    let ba = b-a;
    let h = clamp(dot(pa,ba)/dot(ba,ba), 0.0, 1.0);
    return length(pa - ba*h);
}

fn rotation(roll: f32, pitch: f32, yaw: f32) -> mat3x3f {
    let sir = sin(roll);
    let cor = cos(roll);
    let sip = sin(pitch);
    let cop = cos(pitch);
    let siy = sin(yaw);
    let coy = cos(yaw);

    return mat3x3f(cop*cor, siy*sip*cor - coy*sir, coy*sip*cor + siy*sir,
                   cop*sir, siy*sip*sir + coy*cor, coy*sip*sir - siy*cor,
                   -sip, siy*cop, coy*cop);
}

@fragment 
fn fs( @builtin(position) pos : vec4f ) -> @location(0) vec4f {
    // get normalized texture coordinates (aka uv) in range 0-1
    let screenSize = textureDimensions(backBuffer);
    let aspect = f32(screenSize.y)/f32(screenSize.x);
    let npos = uv( pos.xy )*vec2f(1, aspect);

    let meshSize = 12;
    let meshLines = array(
        vec3f(1,1,1), vec3f(1,1,-1), vec3f(1,-1,1), vec3f(1,-1,-1), vec3f(1,1,1), vec3f(1,-1,1), vec3f(1,1,-1), vec3f(1,-1,-1),
        vec3f(-1,1,1), vec3f(-1,1,-1), vec3f(-1,-1,1), vec3f(-1,-1,-1), vec3f(-1,1,1), vec3f(-1,-1,1), vec3f(-1,1,-1), vec3f(-1,-1,-1),
        vec3f(1,1,1), vec3f(-1,1,1), vec3f(1,1,-1), vec3f(-1,1,-1), vec3f(1,-1,1), vec3f(-1,-1,1), vec3f(1,-1,-1), vec3f(-1,-1,-1));

    var rotation = rotation(seconds()*0.663, seconds()*3.475, seconds()*1.5874);

    var out = black;
    for(var i=0; i < meshSize; i++) {
        let cameraZ: f32 = -(sin(seconds()) + 1)*(10./5.)-1;
        let lineStart = meshLines[i*2] * rotation;
        let lineEnd = meshLines[i*2+1] * rotation;
        let transformedStart = vec2(lineStart.x/(lineStart.z-cameraZ), lineStart.y/(lineStart.z-cameraZ));
        let transformedEnd = vec2(lineEnd.x/(lineEnd.z-cameraZ), lineEnd.y/(lineEnd.z-cameraZ));
        let dist = sdSegment(npos, transformedStart, transformedEnd);
        let line = f32(meshLines[i*2].z-cameraZ > 0 && meshLines[i*2+1].z-cameraZ > 0 && dist < 0.001);
        out = vec3f(max(out.r, line * 0.2), max(out.g, line), max(out.b, line * 0.2));
    }

    return vec4f( out.x, out.y, out.z, 1. );
}
```

# Assignment 3 - WebGPU Intro:

Repo Link:
https://github.com/ALTrilling/motion-extraction-webgpu
The README.md there has the explanation.

Website Link:
https://altrilling.github.io/motion-extraction-webgpu/
