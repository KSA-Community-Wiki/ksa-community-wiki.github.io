---
title: FAQ
layout: default
---
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="index.css">
    <title> KSA Community FAQ <title>
</head>
<body>
    <div class="gray-box">
        <h1>KSA Community Wiki</h1>
        <p>Here is the FAQ for new KSA people (source: <a href="url">discord.gg/kittenspaceagency</a></p>
    </div>
    <br>
    <div class="gray-box">
        <details>
            <summary>How are Floating-Point Precision issues handled?</summary>
              <p> Most coordinates in games use Vector3 which is a spatial unit made up of three “floats”, which is a 32bit number format. This is useful for most rendering and spatial purposes, a floating-point variable can represent a wider range of numbers than a fixed-point
variable of the same bit width at the cost of precision.

So essentially, a floating-point number will have a point at which noticeable precision begins being lost. If we define 1 unit as 1 meter, this often becomes noticeable at around 8 kilometres from the origin. You will see lots of “shaking” and other problems, but with rendering but also physics.

Solutions

There are lots of possible approaches. For KSA the main aim is to keep the core architecture as simple as possible. The simulation is powered as much as possible by “doubles”, which are a 64-bit floating-point precision number. Rendering is then done with the camera always at zero, pushing any floating point issues far out to the edges of the camera where they are not perceptible. This approach has been working very well with the KEPLER simulation layer.

Combined with this on the physics level by having different contexts and handling the simulation of those contexts independently, we can avoid having to deal with everything in one “scene”. The key benefit of this context handling is performance but an additional benefit is being able to avoid precision issues with physics handling.<p>
        </details>
  </div>
      <div class="gray-box">
        <details>
            <summary>How many parts will craft support?</summary>
                
<p>
 This is something we will review as time goes on. One of the key reasons we use our BRUTAL Framework for this project is precisely because we want to be able to draw many things with many parts. With projects like AotR we have been able to draw and simulate so many parts that the limitation we applied was driven primarily by simplifying referencing - rather than performance or design issues themselves. What this means is that the limitation came artificially from the ID (a number) that we used to reference the parts. If we use, say, a USHORT (Unsigned Short number, 16bit), it uses 2 bytes and gives a number between 0 and 65535. This is not only about how much memory (or data in MP referencing) is used for that instance, but how we structure the various structs and buffers both on rendering and simulation. My defacto response with these things is to use USHORT (so, up to 65535 parts per vehicle) unless a good reason exists to extend to UINT (Unsigned Integer, 32bit) using 4 bytes. 

Rendering Parts in Batches

In BRUTAL a lot of our rendering is done using instanced meshes. So we don't have a "Renderer Component" like in unity, instead - each "thing" that needs to be drawn can batch together with all other like things. BRUTAL allows this new "instance" to be done directly to the GPU, which is even more efficient than commands in unity like Graphics.DrawMeshInstanced. In fact, we can send such information once to the GPU (either in a batch or each instance) and then ask the GPU to keep doing it until we stop. This means there is no conversation between the CPU <-> GPU which can give enormous benefits in both performance and memory churn. It is worth mentioning, this is not straightforward. There is no convience for us that engines like Unity/Unreal give - this means that all the buffers need to be configured - yet again our framework is named BRUTAL for a reason. But we trade that convience for scale, both in frame by frame performance but (perhaps more importantly) drastically reducing memory churn. 

Simulating Parts

A wider topic will cover our various "layers" and groupings in our simulation. I'll introduce a few concepts here, being Pieces, Part, Vessel. A "part" can be made or many pieces, these pieces can use common meshes that we can then batch together. A game that does this very well is Cities Skylines - where building are actually collections of other meshes. This allows us to maximize the use of batched rendering. There is then a common library of "meshes" that you can use when making parts - or you can just ignore them all and add in a custom mesh as well (very useful for modders with specialist parts).

This requires a much more detailed topic, but "sub part" is a key aspect of the current design. This very much is inspired by mods like Unviersal Storage, together with the technical implementation in AotR and games like Cities Skylines around batch rendering.

This is all a long winded way of saying that in an engine like unity/unreal - a "part" is actually quite a high-cost thing in the games scene. In BRUTAL and KSA, it is simply a C# class that likely has some pointers back to a core template. This drastically reduces the memory cost, the memory churn, and allows us to fine-tune the simulation and rendering approaches.<p>
        </details>
  </div>
    </div>
      <div class="gray-box">
        <details>
            <summary>Will Multiplayer be supported??</summary>
                <p>Yes. But with some key caveats.
Although the exact form is subject to change, the current approach sees us following the "shared timeline" approach. This would function similar to how paradox games do, where any player can change the speed, or the host can, and then that speed is applied to all players.

The multiplayer approach is actually a part of BRUTAL that is already battle-tested with our game Stationeers. When we switched to using our version (RocketNet) or RakNet, we saw many orders of magnitude improvement in multiplayer scale and performance. This is because instead of sending network messages, we fill (and then unpack) a byte array.

Our studio in general we believe is well placed to make Multiplayers, as it is part of many of the games we have made, and much of our studio has a long history making multiplayer games. 

Such an approach means our proposed multiplayer has limited use cases. It would function similar to games such as Stormworks. While you could run a separate space agency, and your own craft, you would need to agree with the people you are playing when to speed up and when to slow down. So the concession we want to make here has strong impact on multiplayer options. This concession helps a great deal with reducing overall complexity, with both how we synchronize things as well as referencing. We can maintain an absolute state in MP, instead of having to record when and what happened, then reconcile them together. Additionally, beyond the technical issues with "time packets", there are UX/UI issues that we just aren't happy to undertake. Perhaps that is the kind of MP that modders might be able to undertake, where they can hold bigger issues for more niche users. This also ties in with our desire for the base game to have more traditional KSP orbits/scales. While modders can do whatever, at a more KSP scale when operating around kerbin-like planets - time warp changes aren't a huge issue compared to the need for this when using RSS for example, where it takes some time even to reach orbit! 
One thing that does help with this, though, is that we don't have the same context of locality being required for a vehicle to "do something". Which means active stationkeeping and simulation comes "for free" for vehicles. This means that vehicles can do various things at all time, taking the concepts that mods like Kerbalism started to implement on KSP but expanding that out to the datastructures and simulation "layers" themselves from the ground up. We don't have "unity prefabs" or a "physics SDK" to worry about - so the simulation can be segmented up however we want.<p>  
        </details>
  </div>
    </div>
      <div class="gray-box">
        <details>
            <summary>What is the Art Style?</summary>
<p>This is still somewhat in flux, but somewhere between existing KSP and KSP2. Consistency, performance, and ease of development/modding are the only important aspects for the studio when it comes to the art style.
We are mostly using PBR for the shader approach, so that allows us to make quite nice looking assets.

Some of our team members come from KSP development, and other similar games (such as our own Stationeers), and other games such as ArmA/OFP. We strongly believe in there being technical alignment between the art at the product, which we demonstrated recently in many of our assets for Stationeers - ensuring they were modeled with thought as to their actual real-life counterparts.</p>  
        </details>
  </div>
    </div>
      <div class="gray-box">
        <details>
            <summary>Will the Game support Modding?</summary>
             <pre>Yes. It already does.
The very core game data itself is a mod. Which means that essentially everything we do, can be done as a mod. This includes:
- C# injection
- Changing data, such as solar systems and planets
- Customizing shaders 

Really pretty much everything. Modding is considered
essential to every aspect of the game.
This means it factors into our designs not just around how data is loaded, but how 
data is structured within the code.
Early builds will allow us to stress-test our decisions 
early, with modders able to highlight issues with how we structure things. This is important
to minimize core data structure changes during Alpha and Beta (and beyond), as such 
changes are enormously frustrating for users and 
modders at best - and destructive to the community at worst.<pre>
        </details>
  </div>
        <div class="gray-box">
        <details>
            <summary>What Scale will the Solar System have?</summary>
 <p>The core game data is essentially a mod, so anything we do with the game is open for modders to change. This means our core focus is on providing a base solar system, with ease of use for modders to add their own.

At this stage our current thinking is basically do do somewhere between current KSP and x2-2.5 current KSP size for both the bodies and their orbits. In other words, we are aiming to replicate the same feeling, commitment, and challenge of existing KSP.

We feel like base KSP is a great compromise between many factors when it comes to scale, and so we are not trying to reinvent that - instead focused on solid datastructures and ease of development for modders to fill any gaps.</p>  
        </details>
  </div>
    </div>
          <div class="gray-box">
        <details>
            <summary>Will you do N-Body Orbital Simulation?</summary>
 <p>The core focus initially is to provide patchec conics, almost identical to how KSP does it. However, it is possible that if the studio has the right talent (and a team member has the desire) for N-Body to be added as an option. Regardless, the game is being built so a modder could develop a C# mod and add this. Care is being taken to ensure the game is being structured so that if we can't add N-Body physics, someone else could add it.</p>  
        </details>
  </div>
    </div>
          <div class="gray-box">
        <details>
            <summary>What Game Engine do you use?</summary>
<p>We have developed in-house technology we call the "BRUTAL Framework". Instead of an engine, it is more like the XNA Framework developed by Microsoft. BRUTAL allows us to access graphics (and other) API's like Vulkan directly. There is a massive focus on scale, which means a heavy focus on what is called an "interop" layer. This is the layer between which C# (the base language used in our projects using BRUTAL), and C++ which our plugins and APIs like Vulkan run on.

The purpose of using our own framework is that many of the games our studio makes need to scale, and we want to have complete agency over fixing the bugs and problems that are encountered. While both Unity and Unreal are perfectly good tools for many games, our studio has grown intensely frustrated with both of them for developing the types of games we want. They are also both very expensive to utilize.

BRUTAL is named very deliberately. It is not easy to use, and it does no hand holding. It simply exposes the functionality, with nearly it's entire focus providing an extremely efficient interop layer between the two. This results in incredible performance, at the expense of ease of use.

So it is important to clarify, BRUTAL is not a silver bullet. It is simply a tool developed for a very specific purpose - to build games that really scale.</p>  
        </details>
  </div>
    </div>
    <body>
    <div class="gray-box">
        <h1>Can't find specific answers?</h1>
        <p>Join the Offical Discord from the KSA Team. Link: <a href="url">discord.gg/kittenspaceagency</a></p>
    </div>
