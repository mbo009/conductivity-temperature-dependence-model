<template>
  <div ref="physicsModel"></div>
</template>

<script>
import Matter from "matter-js";

export default {
  props: {
    atomSize: {
      type: Number,
      required: true,
    },
  },
  name: "PhysicsModel",
  mounted() {
    this.initPhysics();
  },
  methods: {
    initPhysics() {
      this.engine = Matter.Engine.create({ gravity: { y: 0 } });

      let render = Matter.Render.create({
        element: this.$refs.physicsModel,
        engine: this.engine,
        options: {
          width: 1500,
          height: 500,
          wireframes: false,
        },
      });
      this.addElectrodes();
      this.addAtoms();
      Matter.Render.run(render);
      let runner = Matter.Runner.create();
      Matter.Runner.run(runner, this.engine);

      setInterval(() => {
        this.applyForces();
      }, 16);
    },

    addElectrodes() {
      let anode = Matter.Bodies.rectangle(20, 0, 40, 1000, {
        label: "electrode",
        render: {
          text: "-",
          fillStyle: "blue",
        },
      });
      let cathode = Matter.Bodies.rectangle(1480, 0, 40, 1000, {
        label: "electrode",
        render: {
          fillStyle: "red",
        },
      });
      Matter.Composite.add(this.engine.world, [anode, cathode]);
    },

    addAtoms() {
      this.atoms = [];
      for (let i = 0; i < 50; i++) {
        let atom = Matter.Bodies.circle(
          100 + ((i % 10) * 1300) / 9,
          50 + (Math.floor(i / 10) * 400) / 4,
          this.atomSize,
          {
            restitution: 0.9,
            friction: 0.01,
            render: {
              fillStyle: "green",
            },
          }
        );
        atom.ogPosition = { x: atom.position.x, y: atom.position.y };
        this.atoms.push(atom);
      }
      Matter.Composite.add(this.engine.world, this.atoms);
    },

    updateAtoms() {
      Matter.Composite.clear(this.engine.world, this.atoms);
      this.addAtoms();
      this.addElectrodes();
    },

    moveAtoms() {
      //
    },

    moveCarriers() {
      //
    },

    applyForces() {
      this.moveAtoms();
      this.moveCarriers();
    },
  },

  watch: {
    atomSize: {
      immediate: true,
      handler() {
        if (this.engine) {
          this.updateAtoms();
        }
      },
    },
  },
};
</script>
