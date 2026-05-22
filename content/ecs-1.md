+++
title = 'Entity Component Systems: Part 1'
date = 2026-05-14T06:39:02-04:00
draft = true
tags = ['game development', 'C++']
+++

My only knowledge of entity component systems consists of:

1. This video about building games in Bevy: [](https://www.youtube.com/watch?v=7Wrc6KU2M0s)
2. A brief bit of a Casey Muratori talk about how object oriented programming is bad using gaming as an example. Though I can't seem to find the clip.

From what I've gathered, it's a way of organizing the data in your game that's a lot more efficient than object oriented programming, or modelling your game objects as inheritance trees. This is possible
by defining "systems" that work on "entities" and "components". It's a lot like SQL, actually. If we think of entities as IDs, and components as tables that have a foreign key into the entities, then the system
is just a function from one table to its transformation. It maps each table (component set) to its next state in the game.

I want to try building an ECS system, but I also don't learn anything when reading tutorials, so I'm going to try building one myself. 

Here are some notes from the Bevy video so I'm not flying completely blind:

```
- Components are Rust structs that have the component trait on them.
  - Since we're using C++, a close analogy is probably C++ structs.
- Systems are Rust functions that can perform some desired action.
- Entities are unique identifiers. When we spawn components, or multiple components together, an ID gets associated with them.

To build using ECS, you need to think about three things:
1. How do I want my components to be structured?
2. What components should I combine to create my entities?
3. How should I name them so my systems can easily find what they need?
```

Here's what I'm thinking so far in terms of architecture:

- Every game starts with a game loop, that runs once per frame, and is passed the elapsed time since the last frame
- That game loop should run every system that exists in the game, which means that our gameloop class should have a way of adding and removing systems
- Systems might not need to run every frame, but in case it doesn't, we just do-nothing and return the prior state of the components.
- More interestingly, a system may identify entities according to their components, (we need some kind of filter op / fast search of components), modify those components, and spawn/despawn new entities as necessary.

For instance, let's try to build a n-body sim. There should be particles, displayed on screen in 2d, and these particles should have some forces between them (say gravity). You should be able to spawn a particle with a 
velocity by clicking and dragging, and speed up / slow down / pause the simulation. To build this, I feel like we'd need a Particle{} component (consisting of `mass`, `position`, `velocity`), and a Color{} component,
so we know what color to render it in. Perhaps, since we need to know the position when drawing but not when integrating, we should separate out `position` from `mass` and `velocity`; but position and velocity are very 
similar, so perhaps we have a Position{}, Velocity, and Mass{} component, all separate. I'd like to structure my components such that it's possible to get exactly the projection I need just by selecting a subset of 
components; there should be no unused fields in each system. Perhaps that'd require everything to be too granular; however, let's look at the video for some guidance.

You're probably wondering why I haven't discussed how rendering would work. We will shortly, I think it's related to resources, which we haven't mentioned yet.

```
- "You can think of your game as a database where the ids are the keys and components are the columns." Okay. I misremembered the analogy above.

```

Beyone that, it's relatively straightforward. That's the point. And I think systems control things like rendering, but that might actually
be a job for resources, which we haven't discussed yet.

