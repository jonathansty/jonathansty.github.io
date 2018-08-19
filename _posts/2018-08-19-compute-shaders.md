---
layout: post
title:  "Journey into rust #2: Compute Shaders" 
categories: rust opengl graphics compute shader
date:   2018-08-19 20:30:00 +0000
---
I realized after my first post of this series that it's not just a journey into rust but also OpenGL. 
I've used other Graphics API's before but never actually got my hands dirty into OpenGL. Someone on the [rust user forums](https://users.rust-lang.org/) (they are awesome, go check it out!) suggested using compute shaders instead. At the time I had never used compute shaders for a project so I decided to take some time to refactor the program to use a compute shader. This post is a follow up on that remark and will explore the possibilities of using a rust together with OpenGL to run compute shaders.

Below is a small video of the end result with using compute shaders. There's colors and the cells have a lifetime!
<video width="600px" controls>
    <source src="/assets/rust_pretty_colors.mp4" type="video/mp4" >
</video>

## What is a compute shader?
Remember that graphics pipeline I briefly mentioned in the [previous post]({% post_url 2018-08-07-rust-conway-game-of-life %})? A **compute shader** is a shader **stage** of that pipeline. This stage can be used for computing any information you want really, it can do rendering but the main use for it would be to compute data for other tasks that later get used by the rendering. It's separate from any of the other stages and needs to be explicitly run by using a gl function call.

More information can be found at [compute shader](https://www.khronos.org/opengl/wiki/Compute_Shader).

The reason a compute shader can be useful is that it's independent of any drawing commands. With the fragment shader approach we are limited to using a Image/Texture to sample and render to. A compute shader takes arbitrary inputs and can also have arbitrary outputs. 

A basic compute shader would look like this: 
{% highlight glsl %}
#version 440 
layout(local_size_x = 1 ,local_size_y = 1) in;
layout(rgba8, binding = 0) uniform image2D img_output;
void main() {
    vec4 pixel        = vec4(0.0,0.0,0.0,1.0);
    ivec2 pixel_coord = ivec2(gl_GlobalInvocationID.xy);
    imageStore(img_output, pixel_coord, pixel);
} 
{% endhighlight %}

This is just an example and won't do anything meaningful except for writing out a black texture.

Notice the second line ```layout(local_size_x = 1 ,local_size_y = 1) in;```. This is the part where I would explain the abstract "space" compute shaders run in but instead I'm going to be lazy and link to a  wiki article for you to read.
- [Compute Space](https://www.khronos.org/opengl/wiki/Compute_Shader#Compute_space)

## Sending/Receiving data from the Compute Shader
Sending/Receiving data to and from a compute shader is quite similar as any other shader. You can still use uniforms to send data to the shader. The biggest difference is that compute shaders can write data to buffers and textures directly.

#  Using textures
The first thing I tried was to implement the same logic but in a compute shader. For this I had to change a couple of things. Instead of returning a color value the compute shaders has to directly **write** to a ```image2D```. The ```image2D``` can be defined as a uniform like this:

{% highlight rust %}
layout(rgba8, binding = 0) uniform image2D img_output;
uniform sampler2D u_previous; // Previous state it's texture we can sample from
{% endhighlight %}

Remark the ```layout(rgba8, binding = 0)``` here. These are called **layout qualifiers** and affect where the storage of a certain variable comes from. More info about this can be found [here](https://www.khronos.org/opengl/wiki/Layout_Qualifier_(GLSL)). 
All it boils down to for us is:
- ```rgba8```: The [image format]([binding point](https://www.khronos.org/opengl/wiki/Layout_Qualifier_(GLSL)#Binding_points)). The format the resource will be converted into for read and write operations.
- ```binding = 0 ```: This [binding point](https://www.khronos.org/opengl/wiki/Layout_Qualifier_(GLSL)#Binding_points) . This sets the uniform location to 0. This is important when trying to bind resources to a shader program. 

Now to read from a texture we will use ```texelFetch``` and ```imageStore```. I won't be using these in the final shader as we won't be using textures anymore but I thought it was worth mentioning these as they can be important when you are using textures.

A simplified version of the compute shader would look like this now:
{% highlight glsl %}
#version 440 
layout(local_size_x = 1 ,local_size_y = 1) in;
layout(rgba8, binding = 0) uniform image2D img_output;
uniform sampler2D u_texture; // Previous state it's texture we can sample from
void main() {
    vec4 pixel        = vec4(0.0,0.0,0.0,1.0);
    ivec2 pixel_coord = ivec2(gl_GlobalInvocationID.xy);
    float curr = texelFetch(u_texture,pixel_coord,0).r;
    bool alive = curr > 0;
    int count = 0;
    for(int i = -1; i <= 1; ++i)
    {
        for(int j = -1; j <= 1; ++j)
        {
            if(i == 0 && j == 0)
             continue;
            float tex = texelFetch(u_texture,pixel_coord + ivec2(i,j),0).r;
            if(tex > 0)
                ++count;
        }
    }
    float new_cell = curr;
    if(count < 2)                                   new_cell = 0.0f;
    else if(alive && (count == 2 || count == 3))    new_cell = 1.0f;
    else if(alive && count > 3)                     new_cell = 0.0f;
    else if(!alive && count == 3)                   new_cell = 1.0f;
    pixel =  vec4(new_cell,new_cell,new_cell,1.0f);
    imageStore(img_output, pixel_coord, pixel);
} 
{% endhighlight %}

# Using Shader Storage Buffer Objects (SSBOs)
Using textures is a bit boring, we want to be part of the cool kid SSBO gang. 

A Shader Storage buffer Object (SSBO) is just like a Uniform Buffer Object (UBO) except for the fact that you can write to SSBOs.
Another great link to a wiki article: 
- [Shader Storage Buffer Object](https://www.khronos.org/opengl/wiki/Shader_Storage_Buffer_Object). 

I have abstracted this away into something called a ```StructuredBuffer<T>```. ```T``` is the type of the structure we will be using.

{% highlight rust %}
#[derive(Default)]
pub struct StructuredBuffer<T>
        where T: Default + Clone {
    phantom: std::marker::PhantomData<T>,

    id : GLuint,
    buffer_size : usize
}
{% endhighlight %}

When creating this I ran into a couple of oddities. I wanted a buffer that was strongly typed but when you don't use your **T** the rust compiler complains. To get around this it suggests using ```std::marker::PhantomData<T>``` which works but seems weird and annoying to me coming from C++. As this structured is not supposed to be access from the CPU I won't be keeping track of any CPU data. Once the data is submitted to the GPU we don't care anymore as all the calculating happens on the GPU.  

In the game of life we want to fill our structured buffer with a predefined state. This is quite easy in OpenGL and works as following:
{% highlight rust %}
pub fn from(data : Vec<T>) -> Self {
    // Creates the buffer
    let mut id = 0;
    let buffer_size = std::mem::size_of::<T>() * data.len();

    unsafe{
        gl::GenBuffers(1,&mut id); 
        gl::BindBuffer(gl::SHADER_STORAGE_BUFFER, id);
        gl::BufferData(gl::SHADER_STORAGE_BUFFER, buffer_size as isize, data.as_ptr() as *const c_void, gl::DYNAMIC_COPY);
        gl::BindBuffer(gl::SHADER_STORAGE_BUFFER, 0);
    }

    StructuredBuffer{
        phantom: std::marker::PhantomData,
        id,
        buffer_size,
        elements: data.len()
    }
}
{% endhighlight %}

## Assembling the blocks
Now that we've got our final building blocks we can put this all together into our actual program. First thing I changed was our shader programs, instead of having 2 programs that both had a vertex and fragment shader I've now got 1 program that only does compute shader things and 1 that takes in the output of this compute shader to then proceed and render pretty colours to the screen.

# Setting up our CPU data
Our cell data is described like this in rust. There's a similar definition in GLSL (see below) that matches the same elements.
{% highlight rust %}
struct CellData{
    alive : bool,
    lifetime : f32,
    creation : f32,
};
{% endhighlight %}
I added the ```lifetime``` and ```creation``` parameters so we can use them to nicely visualize the simulation over time. Eventually I think I will try to implement a rudimentary form of GPU fluid simulation in 2D of course.

The ```generate_field``` method returns a ```Vec<CellData>``` object which we then use to create a ```StructureBuffer<CellData>```. The current state will be initialized to 0 because we will be overwriting the first state anyways.

{% highlight rust %}
let image_data = Application::generate_field(&field_size);
let prev_sb = StructuredBuffer::from(image_data);
let curr_sb = StructuredBuffer::new((field_size.x * field_size.y) as usize);
{% endhighlight %}


For every cycle we need to ```dispatch``` a compute call to the GPU.

*Hint: Not sure if your shader gets the right inputs? Check renderdoc in the "Compute shader" tab. It allows you to view to contents of each SSBO that the shader receives!*

{% highlight rust %}
self.gl_ctx.bind_pipeline(&self.compute_program);

self.compute_program.set_uniform("u_field_size", Uniform::Vec2(self.field_size.x as f32,self.field_size.y as f32));
self.compute_program.set_uniform("u_dt", Uniform::Float(update_time as f32));
self.compute_program.set_uniform("u_time", Uniform::Float(self.get_time() as f32));

self.compute_program.bind_storage_buffer(self.curr_sb.get_id(),0);
self.compute_program.bind_storage_buffer(self.prev_sb.get_id(),1);

// Dispatches a arbitrary number of groups, our local work group size is 8x8x1 so we divide our
// field by 8. 
self.gl_ctx.dispatch_compute(
    self.field_size.x as u32 / 8,
    self.field_size.y as u32 / 8,
    1,
);

self.gl_ctx.memory_barrier(MemoryBarrier::ShaderStorage);

std::mem::swap(&mut self.curr_sb, &mut self.prev_sb);
{% endhighlight %}
Notice how for the ```dispatch_compute``` call divide our field by 8. This is because the space in our compute shader (local_size_x and local_size_y) will process in groups that are sized 8x8x1.

See [OpenGL compute  shader dispatching](https://www.khronos.org/opengl/wiki/Compute_Shader#Dispatch) for more info.

Also see [Compute shader limitations](https://www.khronos.org/opengl/wiki/Compute_Shader#Limitations). In this sample program I do not do any checking of the limits as to keep the code simpler.

# The Shader 
Now that we've set up our CPU data we need to read this data in our compute shader. 

Below code snippet is the first part of my shader that defines:
- a copy of our data structures (needed because our shader doesn't know about rust structures). 
- Other uniform inputs such as ```u_time```,```u_dt``` and ```u_field_size```
    - These are used to determine the cell it's state and rendering in later stages.
- an ```OutputData``` and ```InputData``` interface block which as 1 variable array of ```CellData```'s
    - This is where the magic happens.
    - *Remark: Both input and output data blocks have a "shared" memory layout. This makes sure that all our variables are marked "active" and not ignored*

{% highlight glsl %}
struct CellData{
    bool alive;
    float lifetime;
    float creation;
};

uniform vec2 u_field_size;
uniform float u_dt;
uniform float u_time;

layout(shared, binding = 0) writeonly buffer OutputData
{
    CellData next[];
};

layout(shared, binding = 1) readonly buffer InputData
{
    CellData input[];
};
{% endhighlight %}

The actual compute happens in the main function of ```shader.compute```. It's not very different as before, instead of reading from textures and storing the values we now read directly from variable arrays.
{% highlight glsl %}
void main() {
    vec4 pixel        = vec4(0.0,0.0,0.0,1.0);
    ivec2 pixel_coord = ivec2(gl_GlobalInvocationID.xy);

    CellData curr = input[GetArrayId(pixel_coord)];

    bool alive = curr.alive;

    int count = 0;
    // SNIP --> Count all neighbours

    CellData new_cell;
    new_cell.alive = false;
    new_cell.lifetime = curr.lifetime;
    new_cell.creation = curr.creation;

    // SNIP -> Check new alive state

    new_cell.lifetime = curr.lifetime - u_dt;

    // If our cell becomes alive we set some variables. "lifetime" is a bad name seeing as we decrement it over time.
    if(new_cell.alive && !curr.alive)
    {
        new_cell.lifetime = 1.0;
        new_cell.creation = u_time;
    }

    if(new_cell.lifetime < 0.0)
        new_cell.alive = false;


    next[GetArrayId(pixel_coord)] = new_cell;
}
{% endhighlight %}

I also had to adjust my actual fragment shader to render out using the new data. I won't be posting that code here but you can see this on the [github repo](https://github.com/jonathansty/Rust-Game-of-life)!

The final result looks like this:
<video width="660px" controls style="clear:left">
    <source src="/assets/rust_pretty_colors.mp4" type="video/mp4" >
</video>

# References
- [khronos wiki](https://www.khronos.org/opengl/wiki/)
- [anton gerdelan](http://antongerdelan.net/opengl/compute.html)
- [wili's blog](http://wili.cc/blog/opengl-cs.html)