<template>
  <div ref="physicsModel"></div>
</template>

<script>
import Matter from "matter-js";

export default {
  name: "PhysicsModel",
  mounted() {
    let Engine = Matter.Engine,
      Render = Matter.Render,
      Runner = Matter.Runner,
      Bodies = Matter.Bodies,
      Composite = Matter.Composite;

    let engine = Engine.create();

    let render = Render.create({
      element: this.$refs.physicsModel,
      engine: engine,
      options: {
        width: 800,
        height: 600,
        wireframes: false,
      },
    });

    let boxA = Bodies.rectangle(400, 200, 80, 80);
    let boxB = Bodies.rectangle(450, 50, 80, 80);

    let ground = Bodies.rectangle(400, 610, 810, 60, { isStatic: true });

    Composite.add(engine.world, [boxA, boxB, ground]);

    Render.run(render);
    let runner = Runner.create();
    Runner.run(runner, engine);
  },
};
</script>
