---
layout:     post
title:      A beginners guide to TGSI 
date:       2016-07-04 15:31:19
excerpt_separator: <!--more-->
categories: Programming 
tags: [mesa, linux, graphics]
comments:   true

---
TGSI stands for Tungsten Graphics Shader Infrastructure. The development of TGSI was started by tungsten graphics and hence the name. So what the 
heck is TGSI? Let's find out together.
<!--more-->

When we write a code and compile it, the compiler converts it into a intermediate representation which is suitable for optimisation before it is 
translated to machine code. TGSI is a intermediate representation to write shaders and it is compiled to create machine code according to the underlying 
hardware. When you code in TGSI you get the feeling of coding in assembly language. When I was working on VDPAU I had to write shaders
in TGSI and the only docs that I could get my hands on was [this](http://gallium.readthedocs.io/en/latest/tgsi.html). Though it was not complete and only had
information about opcodes I was able to find my way by looking at the code available in mesa. So enough with the chit chat let's look at
some at some code.

```c
create_frag_shader(struct pipe context *pipe)
{
struct ureg_program *shader;
struct ureg_src i_vtex, sampler;
struct ureg_dst texel;
struct ureg_dst fragment;

shader = ureg_create(PIPE_SHADER_FRAGMENT);
if (!shader)
   return false;

i_vtex = ureg_DECL_fs_input(shader, TGSI_SEMANTIC_GENERIC,
                            VS_O_VTEX, TGSI_INTERPOLATE_LINEAR);
sampler = ureg_DECL_sampler(shader, 0);
texel = ureg_DECL_temporary(shader);
fragment = ureg_DECL_output(shader, TGSI_SEMANTIC_COLOR, 0);

ureg_TEX(shader, texel, TGSI_TEXTURE_2D, i_vtex, sampler);

/*
 * Do operations on the texel.
 */

ureg_MOV(shader, o_fragment, ureg_src(texel));

ureg_release_temporary(shader, texel);
ureg_END(shader);
   
return ureg_create_shader_and_destroy(shader, pipe);
}
```
This is the simplest fragment shader you can come up with, it is like the hello world program of shaders. Let's go over it and try to understand 
what it is doing.

All the TGSI shaders have a ureg_program which is kind of program that has to be excuted for each pixel so each of our commands are indexed in ureg_program and every instruction requires ureg_program as argument.
All TGSI instructions, known as opcodes, operate on arbitrary-precision floating-point four-component vectors. We use X, Y, Z and W to represent the components of the vector. 
ureg_src and ureg_dst represent 4 component vectors, ureg_dst is the destination vector and ureg_src is the source vector as is obivous with their names. We can get a ureg_src from a ureg_dst using `ureg_src()`.

So let's start with the code, we first declare all the ureg_* which we will need during the program. The next step is to initialize the ureg program.
After that we call ureg_DECL_fs_input which gives us the coordinate of the pixel on which we are executing. The next thing is the sampler, the 
sampler has the information about the texture of the pixel. Now we declare a temporary vector and the output vector in which we have to put the 
color of the pixel we are operating on.

The `ureg_TEX()` function then puts the texture of the given coordinate retrieved from the sampler_view to the dst vector. The color information 
is stored as RGBA (X:R, Y:G, Z:B, W:A) in ureg_dst. We can now perform operations on the texel like setting a particular component to zero and even 
complex algorithms like lanczos scaling. In above example we  do not edit the color of pixel and simply put it the o_fragment vector. 

All the opcodes in TGSI have one dst vector and from 0 to 3 source vectors. Let's look at some of the opcodes. You can find a comprehensive list of
opcodes [here](http://gallium.readthedocs.io/en/latest/tgsi.html#instruction-set). All these instructions are excuted component wise.

* `ureg_MOV(shader, dst, src);`: dst = src
* `ureg_ADD(shader, dst, src0, src1)`: dst = src0 + src1
* `ureg_SUB(shader, dst, src0, src1)`: dst = src0 - src1
* `ureg_MUL(shader, dst, src0, src1)`: dst = src0 * src1
* `ureg_DIV(shader, dst, src0, src1)`: dst = src0 / src1
* `ureg_MAD(shader, dst, src0, src1, src2)`: dst = (src0 * src1) + src2
* `ureg_FRC(shader, dst, src)`: dst = fraction(src)
* `ureg_FLR(shader, dst, src)`: dst = floor(src)


There are some functions other than the opcodes which are very useful like

* `ureg_imm*f` 

   The '*' can be 1, 2, 3 and 4. This function is used to create a ureg_src with given values. e.g.

```c
   ureg_src src = ureg_imm1f(shader, 1.0f);
   ureg_src src = ureg_imm2f(shader, 0.5f, 0.4f);
   ureg_src src = ureg_imm3f(shader, 0.6f, 0.3f, 0.4f);
```

* `ureg_writemask`

   It allows us to perform operations on a particular components of a vector. e.g. 

```c
   ureg_MOV(shader, ureg_writemask(dst, TGSI_WRITEMASK_X),
            ureg_imm1f(shader, 1.0f));
```
* `ureg_scalar`
   This allows us to create a vector which contains a particular component of some other vector in all of its components. e.g.

```c
   ureg_src src = ureg_scalar(src0, TGSI_SWIZZLE_X);
```
   This output of this instruction is 

```
   src.x = src0.x
   src.y = src0.x
   src.z = src0.x
   src.w = src0.x
```
I hope that now you are capable of writing a shader in TGSI. We will look how we can write more complex shaders using TGSI in the next post until 
then

Have a Good Day!!
