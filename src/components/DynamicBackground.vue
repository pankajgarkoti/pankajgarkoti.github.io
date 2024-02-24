<script lang="ts">
import { loadFull } from "tsparticles";
import { ParticlesComponent } from "vue3-particles";
import type { Container, Engine } from "tsparticles-engine";

export default {
  name: "DynamicBackground",
  components: { ParticlesComponent },
  methods: {
    particlesInit: async (engine: Engine) => {
      await loadFull(engine);
    },
    particlesLoaded: async (container: Container) => {
      console.log("Particles......OK", container);
    }
  }
};
</script>

<template>
  <div class="w-screen h-screen absolute">
    <div class="mx-auto my-auto">
      <ParticlesComponent
        :id="'tsparticles'"
        :particlesInit="particlesInit"
        :particlesLoaded="particlesLoaded"
        url="http://foo.bar/particles.json"
      />

      <ParticlesComponent
        id="tsparticles"
        :particlesInit="particlesInit"
        :particlesLoaded="particlesLoaded"
        :options="{
          background: {
            color: {
              value: '#161616'
            }
          },
          fpsLimit: 120,
          interactivity: {
            events: {
              onClick: {
                enable: false,
                mode: 'push'
              },
              onHover: {
                enable: false,
                mode: 'repulse'
              },
              resize: true
            },
            modes: {
              bubble: {
                distance: 400,
                duration: 2,
                opacity: 0.8,
                size: 40
              },
              push: {
                quantity: 2
              },
              repulse: {
                distance: 32,
                duration: 4
              }
            }
          },
          particles: {
            smooth: true,
            color: {
              value: '#afaeb444'
            },
            links: {
              color: '#ffffff',
              distance: 48,
              enable: true,
              opacity: 0.3,
              width: 1
            },
            collisions: {
              enable: true
            },
            move: {
              direction: 'none',
              enable: true,
              outMode: 'bounce',
              random: true,
              speed: 2,
              straight: true
            },
            number: {
              density: {
                enable: true,
                area: 800
              },
              value: 40
            },
            opacity: {
              value: 0.4
            },
            shape: {
              type: 'circle'
            },
            size: {
              random: true,
              value: 3
            }
          },
          detectRetina: true
        }"
      />
    </div>
  </div>
</template>

<style scoped>
#tsparticles {
  height: 100%;
  width: 100%;
  position: fixed;
  top: 0;
  left: 0;
  z-index: -50;
}
</style>
