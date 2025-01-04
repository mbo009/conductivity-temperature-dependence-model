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

    temperature: {
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
      this.windowWidth = 1500;
      this.windowHeight = 500;
      this.electricFieldForce = 0.0001;
      this.temperatureFactor = 0.0002;
      this.dampingFactor = 0.99;

      this.engine = Matter.Engine.create({ gravity: { y: 0 } });

      let render = Matter.Render.create({
        element: this.$refs.physicsModel,
        engine: this.engine,
        options: {
          width: this.windowWidth,
          height: this.windowHeight,
          wireframes: false,
        },
      });
      
      this.addWalls();
      this.addElectrodes();
      this.addAtoms();
      this.beforeUpdateLoop()

      Matter.Render.run(render);
      let runner = Matter.Runner.create();
      Matter.Runner.run(runner, this.engine);
    },

    beforeUpdateLoop() {
      Matter.Events.on(this.engine, 'beforeUpdate', () => {
        this.moveAtoms();
      });
    },
    
    gaussianRandom() {
      let rand = 0;
      for (let i = 0; i < 6; i++) {
        rand += Math.random();
      }
      return (rand / 6) * 2 - 1;
    },
    
    moveAtoms() {
      this.atoms.forEach(atom => {
          this.moveAtom(atom);
      });
    },

    moveAtom(atom) {
      atom.velocity.x += this.gaussianRandom() * this.temperatureFactor * (this.temperature + 273);
      atom.velocity.y += this.gaussianRandom() * this.temperatureFactor * (this.temperature + 273);
      Matter.Body.setVelocity(atom, { x: atom.velocity.x, y: atom.velocity.y });
      atom.velocity.x *= this.dampingFactor;
      atom.velocity.y *= this.dampingFactor;
    },

    moveCarriers() {
      //
    },

    addWalls(){
      const walls = [
        Matter.Bodies.rectangle(this.windowWidth / 2, - 50 / 2, this.windowWidth, 50, { isStatic: true }),
        Matter.Bodies.rectangle(this.windowWidth / 2, this.windowHeight + 50 / 2, this.windowWidth, 50, { isStatic: true }),
      ];
    
      Matter.Composite.add(this.engine.world, walls);
    },

    addElectrodes() {
      let anode = Matter.Bodies.rectangle(20, 0, 40, 1000, {
        label: "electrode",
        isStatic: true,
        render: {
          text: "-",
          fillStyle: "blue",
        },
      });

      let cathode = Matter.Bodies.rectangle(1480, 0, 40, 1000, {
        label: "electrode",
        isStatic: true,
        render: {
          fillStyle: "red",
        },
      });

      Matter.Composite.add(this.engine.world, [anode, cathode]);
    },

    addAtoms() {
      this.atoms = [];
      this.constraints = [];
      const springStiffness = 0.0006;
      const rows = 5;
      const cols = 10;

      for (let y = 0; y < rows; y++) {
        for (let x = 0; x < cols; x++) {
          let atom = Matter.Bodies.circle(
            100 + (x * 1300) / (cols - 1),
            50 + (y * 400) / (rows - 1),
            this.atomSize,
            {
              mass: 1,
              restitution: 0.9,
              friction: 0.01,
              render: {
                fillStyle: "green",
              },
            }
          );
          atom.ogPosition = { x: atom.position.x, y: atom.position.y };
          this.atoms.push(atom);
          
          this.constraints.push(Matter.Constraint.create({
            bodyA: atom,
            pointB: { x: atom.position.x, y: atom.position.y },
            stiffness: springStiffness,
            render: {
              visible: false,
            },
          }));
        }
      }

      Matter.Composite.add(this.engine.world, this.atoms);
      Matter.Composite.add(this.engine.world, this.constraints);
    },

    updateAtoms() {
      Matter.Composite.clear(this.engine.world, this.atoms);
      this.addAtoms();
      this.addElectrodes();
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
    temperature: {
      immediate: true,
    }
  },
};
</script>
