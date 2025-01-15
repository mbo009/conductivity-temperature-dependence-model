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

    atomMass: {
      type: Number,
      required: true,
    },

    carrierSize: {
      type: Number,
      required: true,
    },

    electricFieldForce: {
      type: Number,
      required: true,
    },

    temperature: {
      type: Number,
      required: true,
    },

    carrierConcentration: {
      type: Number,
      required: true,
    },

    isElectricFieldOn: {
      type: Boolean,
      required: true,
    },

    isInteractive: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
      driftVelocity: 0,
    };
  },
  name: "PhysicsModel",
  mounted() {
    this.initPhysics();
  },
  methods: {
    initPhysics() {
      this.windowWidth = 1500;
      this.windowHeight = 500;
      this.temperatureFactor = 0.0002;
      this.dampingFactor = 0.99;
      this.springStiffness = 0.0006;
      this.carrierMass = 0.00055;
      this.carrierCharge = 1.6e-19;

      this.engine = Matter.Engine.create({ gravity: { y: 0 } });

      this.render = Matter.Render.create({
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

      this.carriers = [];
      this.addCarriers();
      if (this.isInteractive) {
        this.addMouseInteraction();
      }
      this.beforeUpdateLoop();
      this.afterUpdateLoop();
      this.checkCollisions();

      Matter.Render.run(this.render);
      let runner = Matter.Runner.create();
      Matter.Runner.run(runner, this.engine);
    },

    addMouseInteraction() {
      this.mouse = Matter.Mouse.create(this.render.canvas);
      this.mouseConstraint = Matter.MouseConstraint.create(this.engine, {
        mouse: this.mouse,
        constraint: {
          stiffness: 0.2,
          render: {
            visible: false,
          },
        },
      });

      Matter.Composite.add(this.engine.world, this.mouseConstraint);
      this.render.mouse = this.mouse;
    },

    beforeUpdateLoop() {
      Matter.Events.on(this.engine, "beforeUpdate", () => {
        this.moveAtoms();
        this.moveCarriers();
      });
    },

    afterUpdateLoop() {
      Matter.Events.on(this.engine, "afterUpdate", () => {
        this.renderTrail(this.carriers[0]);
        this.emitDriftVelocity();
      });
    },

    renderTrail(carrier) {
      carrier.trail.push({ x: carrier.position.x, y: carrier.position.y });

      if (carrier.trail.length > 300) {
        carrier.trail.shift();
      }

      const canvas = this.render.canvas;
      const context = canvas.getContext("2d");

      context.beginPath();
      for (let i = 1; i < carrier.trail.length; i++) {
        const current = carrier.trail[i];
        const previous = carrier.trail[i - 1];

        context.moveTo(previous.x, previous.y);
        context.lineTo(current.x, current.y);
      }

      context.strokeStyle = "gray";
      context.lineWidth = 3;
      context.stroke();
    },

    checkCollisions() {
      Matter.Events.on(this.engine, "collisionStart", (event) => {
        const pairs = event.pairs;

        pairs.forEach((pair) => {
          const bodyA = pair.bodyA;
          const bodyB = pair.bodyB;

          if (bodyA.label == "cathode" && bodyB.label == "carrier") {
            this.resetCarrier(bodyB);
          }
        });
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
      this.atoms.forEach((atom) => {
        this.moveAtom(atom);
      });
    },

    moveAtom(atom) {
      atom.velocity.x +=
        this.gaussianRandom() *
        this.temperatureFactor *
        (this.temperature + 273);
      atom.velocity.y +=
        this.gaussianRandom() *
        this.temperatureFactor *
        (this.temperature + 273);
      Matter.Body.setVelocity(atom, { x: atom.velocity.x, y: atom.velocity.y });
      atom.velocity.x *= this.dampingFactor;
      atom.velocity.y *= this.dampingFactor;
    },

    moveCarriers() {
      this.carriers.forEach((carrier) => {
        this.moveCarrier(carrier);
      });
    },

    moveCarrier(carrier) {
      if (this.isElectricFieldOn) {
        Matter.Body.applyForce(carrier, carrier.position, {
          x: this.electricFieldForce * this.carrierCharge * 10000,
          y: 0,
        });
      }
    },

    addWalls() {
      const walls = [
        Matter.Bodies.rectangle(
          this.windowWidth / 2,
          -50 / 2,
          this.windowWidth,
          50,
          { isStatic: true, restitution: 1, friction: 0 }
        ),
        Matter.Bodies.rectangle(
          this.windowWidth / 2,
          this.windowHeight + 50 / 2,
          this.windowWidth,
          50,
          { isStatic: true, restitution: 1, friction: 0 }
        ),
      ];

      Matter.Composite.add(this.engine.world, walls);
    },

    addElectrodes() {
      let anode = Matter.Bodies.rectangle(20, 0, 40, 1000, {
        label: "anode",
        isStatic: true,
        render: {
          text: "-",
          fillStyle: "blue",
        },
      });

      let cathode = Matter.Bodies.rectangle(1480, 0, 40, 1000, {
        label: "cathode",
        isStatic: true,
        render: {
          fillStyle: "red",
        },
      });

      Matter.Composite.add(this.engine.world, [anode, cathode]);
    },

    createAtom(x, y) {
      let atom = Matter.Bodies.circle(x, y, this.atomSize, {
        label: "atom",
        mass: 190,
        restitution: 0.9,
        friction: 0,
        inertia: Infinity,
        render: {
          fillStyle: "red",
        },
      });
      atom.ogPosition = { x: atom.position.x, y: atom.position.y };
      return atom;
    },

    createConstraint(atom) {
      return Matter.Constraint.create({
        bodyA: atom,
        pointB: { x: atom.ogPosition.x, y: atom.ogPosition.y },
        stiffness: this.springStiffness,
        length: 0,
        render: {
          visible: false,
        },
      });
    },

    addAtoms() {
      this.atoms = [];
      this.constraints = [];
      const rows = 5;
      const cols = 10;

      for (let y = 0; y < rows; y++) {
        for (let x = 0; x < cols; x++) {
          let atom = this.createAtom(
            100 + (x * 1300) / (cols - 1),
            50 + (y * 400) / (rows - 1)
          );

          this.atoms.push(atom);

          let constraint = this.createConstraint(atom);
          this.constraints.push(constraint);
        }
      }

      Matter.Composite.add(this.engine.world, this.atoms);
      Matter.Composite.add(this.engine.world, this.constraints);
    },

    resizeAtoms() {
      this.atoms.forEach((atom, index) => {
        let newAtom = this.createAtom(atom.position.x, atom.position.y);
        Matter.Body.setVelocity(newAtom, atom.velocity);

        Matter.Composite.remove(this.engine.world, atom);
        Matter.Composite.remove(this.engine.world, this.constraints[index]);

        newAtom.ogPosition = { x: atom.ogPosition.x, y: atom.ogPosition.y };

        let newConstraint = this.createConstraint(newAtom);

        this.atoms[index] = newAtom;
        this.constraints[index] = newConstraint;

        Matter.Composite.add(this.engine.world, [newAtom, newConstraint]);
      });
    },

    changeAtomMass() {
      this.atoms.forEach((atom, index) => {
        let newAtom = this.createAtom(atom.position.x, atom.position.y);
        Matter.Body.setVelocity(newAtom, atom.velocity);

        Matter.Composite.remove(this.engine.world, atom);
        Matter.Composite.remove(this.engine.world, this.constraints[index]);

        newAtom.ogPosition = { x: atom.ogPosition.x, y: atom.ogPosition.y };

        let newConstraint = this.createConstraint(newAtom);

        this.atoms[index] = newAtom;
        this.constraints[index] = newConstraint;

        Matter.Composite.add(this.engine.world, [newAtom, newConstraint]);
      });
    },

    addCarriers() {
      let carrierCount =
        (this.windowHeight * this.windowWidth * this.carrierConcentration) /
        37500;
      for (let i = 0; i < carrierCount; i++) {
        this.carriers.push(this.createCarrier());
      }
      Matter.Composite.add(this.engine.world, this.carriers);
    },

    createCarrier() {
      let carrier = Matter.Bodies.circle(
        45,
        Math.random() * this.windowHeight,
        this.carrierSize,
        {
          label: "carrier",
          mass: this.carrierMass,
          friction: 0,
          restitution: 0.9,
          render: {
            fillStyle: "blue",
          },
          collisionFilter: {
            group: -1,
          },
        }
      );
      carrier.trail = [];
      return carrier;
    },

    resetCarrier(carrier) {
      Matter.Body.setPosition(carrier, {
        x: 45,
        y: Math.random() * this.windowHeight,
      });
      Matter.Body.setVelocity(carrier, { x: 0, y: 0 });
      carrier.trail = [];
    },

    updateCarriers() {
      let carrierCount = this.carrierCount();
      if (this.carriers.length > carrierCount) {
        this.carriers
          .slice(0, this.carriers.length - carrierCount)
          .forEach((carrier) => {
            Matter.Composite.remove(this.engine.world, carrier);
          });
        this.carriers = this.carriers.slice(
          this.carriers.length - carrierCount,
          this.carriers.length
        );
      } else {
        for (let i = this.carriers.length; i < carrierCount; i++) {
          const carrier = this.createCarrier();
          this.carriers.push(carrier);
          Matter.Composite.add(this.engine.world, carrier);
        }
      }
    },

    resizeCarriers() {
      this.carriers.forEach((carrier, index) => {
        let newCarrier = this.createCarrier();
        Matter.Body.setVelocity(newCarrier, carrier.velocity);
        Matter.Body.setPosition(newCarrier, carrier.position);

        Matter.Composite.remove(this.engine.world, carrier);

        this.carriers[index] = newCarrier;

        Matter.Composite.add(this.engine.world, newCarrier);
      });
    },

    carrierCount() {
      return (
        (this.windowHeight * this.windowWidth * this.carrierConcentration) /
        37500
      );
    },

    resetSimulation() {
      Matter.Composite.clear(this.engine.world, this.atoms);
      this.addAtoms();
      this.addElectrodes();
      this.addCarriers();
    },

    emitDriftVelocity() {
      this.driftVelocity = Math.sqrt(
        this.carriers[0].velocity.x ** 2 + this.carriers[0].velocity.y ** 2
      );
      this.driftVelocity = Math.round(this.driftVelocity * 100) / 100;
      this.$emit("drift-velocity", this.driftVelocity);
    },
  },
  watch: {
    atomSize: {
      immediate: true,
      handler() {
        if (this.engine) {
          this.resizeAtoms();
        }
      },
    },
    carrierConcentration: {
      immediate: true,
      handler() {
        if (this.engine) {
          this.updateCarriers();
        }
      },
    },
    isInteractive: {
      immediate: true,
      handler() {
        if (this.engine) {
          if (this.isInteractive) {
            this.addMouseInteraction();
          } else {
            Matter.Composite.remove(this.engine.world, this.mouseConstraint);
          }
        }
      },
    },
    carrierSize: {
      immediate: true,
      handler() {
        if (this.engine) {
          this.resizeCarriers();
        }
      },
    },
    atomMass: {
      immediate: true,
      handler() {
        if (this.engine) {
          this.changeAtomMass();
        }
      },
    },
  },
};
</script>
