# Purpose of this small project

I plan to use `d3` to help with layout in 🎈Billow 🎈, and `svelte` as the app framework. Initially, the goal was to replicate [this tutorial](http://www.puzzlr.org/force-directed-graph-minimal-working-example/) in the context of a Svelte project, but now it's more…floaty and…balloony. 😬

As of this writing, you see a blue sky with a button imploring you to add a balloon. When you click it, a balloon sweeps in with text indicating its randomly-assigned group. Each group is attracted to a given height — one of: "bottom", "down", "middle", "up", or "top".

# What this accomplishes

I'm attempting to make a piece of solo software, but I'm a theater major with almost exclusively self-directed and on-the-job training. Though my knowledge of software practices runs both wide and deep, I've never had the luxury of being a student with the time and motivation to build systems from scratch _and then maintain them!_ Point being, if I'm going to make something without venture backing and a team of devs and such helping me, it's got to be lean af.

Enter [Svelte](https://svelte.dev). There's not much for me to say about it that isn't made clear right on the home page: it's easy to learn, easy to use, speedy and small. In short, Svelte is a delicious joy.

In my mind (and in sketches and Figma mocks and in a really complex Google Sheet I created as an early prototoype), 🎈Billow 🎈 is meant to invoke the feeling of playing the classic, nameless party game that begins as soon as someone tosses a balloon in the air and shouts, "Don't let it touch the floor!" Except that it's gonna be a productivity app, and the balloons take days, not seconds, to reach the ground. And the scene an idyllic field of spiky, balloon-popping grass (not shown in this prototype) topped by a wide blue sky. Which brings me to `d3`.

[d3](https://d3js.org) is a collection of tools designed by another idiosyncratic genius. The learning curve is notoriously steep, and so it helps to have a particular application to which d3's particularly well suited before diving in. In my case, I needed something that could take my SVG balloon shapes and a) clump them in the sky, b) employ collision-detection so that they don't overlap, c) allow positional overrides at will (for later, when it's time to interact with the balloons), and cetera. `d3` is all about the and cetera. And, on the surface, it's got a completely different philosophy from svelte, inasmuch as it was built before arrow functions.

But there are important similarities!

* Like `Svelte`, `d3` is designed to integrate into essentially any javascript environment.
* Like `Svelte`, there are tons of live demos to play with. This helps when learning things that just aren't quite clear from the official docs. (`d3`'s docs are great, but they often contain example-free paragraphs that don't make sense until you come back to them later, having tried to build something from scratch or, more likely, by modifying existing examples.)
* Both are powerful enough that a few minutes can yield astonishing results. And both are large enough in scale that a single pass isn't sufficient to learn all there is to learn.
  * Corallary: Both can be intimidating, but, in fact, scale well with one's learning, as long as that learning is experiential.
* They're both FOSS.

This last point would prove to be critical as I faced one more of their similarities. One which would send me exploring deep into the innards of `d3-selection` and `d3-force`:

* Both have facilities for easily manipulating the DOM.

There doesn't have to be any conflict here (as you can see from the finished product), but there certainly *can* be if you don't understand `d3`'s selection logic. Now we're past that, though, and I'm psyched to see what more I can do with these two tools working arm-in-arm!

## How to run it

### Install

```
▶ git clone https://github.com/clozach/Playing-with-d3-Force-Layouts-in-Svelte.git d3force
Cloning into 'd3force'...
remote: Enumerating objects: 220, done.
remote: Counting objects: 100% (220/220), done.
remote: Compressing objects: 100% (103/103), done.
remote: Total 220 (delta 129), reused 205 (delta 116), pack-reused 0
Receiving objects: 100% (220/220), 179.13 KiB | 4.17 MiB/s, done.
Resolving deltas: 100% (129/129), done.

```

```
▶ cd d3force
▶ npm install

> fsevents@1.2.9 install /Users/c/Documents/1-Projects/billow/tossme/d3force/node_modules/fsevents
> node install

node-pre-gyp WARN Using needle for node-pre-gyp https download 
[fsevents] Success: "/Users/c/Documents/1-Projects/billow/tossme/d3force/node_modules/fsevents/lib/binding/Release/node-v72-darwin-x64/fse.node" is installed via remote
npm WARN svelte-app@1.0.0 No repository field.
npm WARN svelte-app@1.0.0 No license field.

added 305 packages from 217 contributors and audited 2295 packages in 3.914s
found 0 vulnerabilities
```

### Run

```
▶ npm run dev
  Your application is ready~! 🚀

  - Local:      http://localhost:5000

────────────────── LOGS ──────────────────

rollup v1.20.3
bundles src/main.js → public/bundle.js...
LiveReload enabled
created public/bundle.js in 320ms

[2019-09-20 23:30:35] waiting for changes...

```
