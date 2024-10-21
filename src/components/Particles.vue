<template>
  <div id="particles-container" class="particles-container">
    <!-- Text in the center of the screen -->
    <div class="centered-content">
      <h1 class="intro-text">
        Are you ready to take a tour of the knowledge map
        <span class="blinking">?</span>
      </h1>
      <div class="explore-container">
        <button class="explore-button">
          Explore
          <i class="fas fa-location-arrow"></i>
          <!-- Location arrow icon -->
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "ParticlesComponent",
  mounted() {
    this.initializeParticles();
  },
  methods: {
    initializeParticles() {
      if (window.particlesJS) {
        window.particlesJS("particles-container", {
          particles: {
            number: {
              value: 120,
              density: {
                enable: true,
                value_area: 800,
              },
            },
            color: {
              value: "#ffffff",
            },
            shape: {
              type: "circle",
              stroke: {
                width: 0,
                color: "#000000",
              },
              polygon: {
                nb_sides: 5,
              },
            },
            opacity: {
              value: 0.5,
              random: false,
              anim: {
                enable: false,
                speed: 1,
                opacity_min: 0.1,
                sync: false,
              },
            },
            size: {
              value: 3,
              random: true,
              anim: {
                enable: false,
                speed: 40,
                size_min: 0.1,
                sync: false,
              },
            },
            line_linked: {
              enable: true,
              distance: 150,
              color: "#ffffff",
              opacity: 0.4,
              width: 1,
            },
            move: {
              enable: true,
              speed: 6,
              direction: "none",
              random: false,
              straight: false,
              out_mode: "out",
              bounce: false,
              attract: {
                enable: false,
                rotateX: 600,
                rotateY: 1200,
              },
            },
          },
          interactivity: {
            detect_on: "canvas",
            events: {
              onhover: {
                enable: true,
                mode: "repulse",
              },
              onclick: {
                enable: true,
                mode: "push",
              },
              resize: true,
            },
            modes: {
              grab: {
                distance: 400,
                line_linked: {
                  opacity: 1,
                },
              },
              bubble: {
                distance: 400,
                size: 40,
                duration: 2,
                opacity: 8,
                speed: 3,
              },
              repulse: {
                distance: 200,
              },
              push: {
                particles_nb: 4,
              },
              remove: {
                particles_nb: 2,
              },
            },
          },
          retina_detect: true,
        });
      }
    },
  },
};
</script>

<style scoped>
/* Ensure particles take the whole screen */
.particles-container {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0;
  z-index: -1;
  background-color: #000;
}

/* Center content vertically and horizontally */
.centered-content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  color: white;
}

/* Styling for the intro text */
.intro-text {
  font-size: 48px;
  font-weight: bold;
  font-family: "Arial", sans-serif;
  text-transform: uppercase;
}

/* Blinking effect for the question mark */
.blinking {
  animation: blink 1s infinite;
}

/* Blink keyframes */
@keyframes blink {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

/* Explore button container */
.explore-container {
  margin-top: 20px;
}

.explore-button {
  font-size: 40px; /* Adjust font size for icon */
  padding: 15px 30px; /* Adjust padding */
  background-color: #ffcc00;
  border: none;
  border-radius: 50px; /* Fully rounded for pill shape */
  color: black;
  font-weight: bold;
  cursor: pointer;
  animation: easeInOut 1.5s infinite;
  width: 300px;
  height: 60px;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px; /* Add some space between the text and icon */
  transition: background-color 0.3s ease;
  margin: 0 auto;
}

/* Add hover effect for button */
.explore-button:hover {
  background-color: #ffbb00;
}

/* Ease in and out animation */
@keyframes easeInOut {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.1);
  }
  100% {
    transform: scale(1);
  }
}
</style>
