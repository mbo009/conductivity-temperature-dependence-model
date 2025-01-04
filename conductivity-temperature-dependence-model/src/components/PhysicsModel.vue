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

    carrierConcentration: {
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
      this.springStiffness = 0.0006;

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
      
      this.carriers = [];
      this.addCarriers();
      this.beforeUpdateLoop();
      this.checkCollisions();

      Matter.Render.run(render);
      let runner = Matter.Runner.create();
      Matter.Runner.run(runner, this.engine);
    },

    beforeUpdateLoop() {
      Matter.Events.on(this.engine, 'beforeUpdate', () => {
        this.moveAtoms();
        this.moveCarriers();
      });
    },
    
    checkCollisions(){
      Matter.Events.on(this.engine, 'collisionStart', (event) => {
        const pairs = event.pairs;

        pairs.forEach(pair => {
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
      this.carriers.forEach(carrier => {
        this.moveCarrier(carrier);
      });
    },

    moveCarrier(carrier) {
      Matter.Body.applyForce(carrier, carrier.position, { x: this.electricFieldForce, y: 0 });
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
        mass: 1,
        restitution: 0.9,
        friction: 0.01,
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

    updateAtoms(){
      this.atoms.forEach((atom, index) => {
        let newAtom = this.createAtom(atom.position.x, atom.position.y);
        Matter.Body.setVelocity(newAtom, atom.velocity);
    
        Matter.Composite.remove(this.engine.world, atom);
        Matter.Composite.remove(this.engine.world, this.constraints[index]);

        newAtom.ogPosition = { x: atom.ogPosition.x, y: atom.ogPosition.y };

        let newConstraint = this.createConstraint(newAtom)

        this.atoms[index] = newAtom;
        this.constraints[index] = newConstraint;

        Matter.Composite.add(this.engine.world, [newAtom, newConstraint]);
      });
    },

    addCarriers() {
      let carrierCount = (this.windowHeight * this.windowWidth) * this.carrierConcentration / 37500;
      for (let i = 0; i < carrierCount; i++) {
        this.addCarrier();
      }
    },

    addCarrier() {
      let carrier = Matter.Bodies.circle(
        45,
        Math.random() * this.windowHeight,
        5,
        {
          label: "carrier",
          mass: 0.1,
          restitution: 0.9,
          friction: 0.01,
          render: {
            fillStyle: "blue",
          },
        }
      );
      this.carriers.push(carrier);
      Matter.Composite.add(this.engine.world, carrier)
    },

    resetCarrier(carrier) {
      Matter.Body.setPosition(carrier, { x: 45, y: Math.random() * this.windowHeight });
      Matter.Body.setVelocity(carrier, { x: 0, y: 0 });
    },
    
    updateCarriers(){

    },


    resetSimulation() {
      Matter.Composite.clear(this.engine.world, this.atoms);
      this.addAtoms();
      this.addElectrodes();
      this.addCarriers();

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
    carrierConcentration: {
      immediate: true,
      handler() {
        if (this.engine) {
          this.resetSimulation();
        }
      } 
    }
  },
};
</script>
