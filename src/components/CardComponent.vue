<template>
  <div v-if="isVisible" class="card-container">
    <!-- Animate the entire card's appearance and disappearance -->
    <transition name="fade-slide-card" mode="out-in">
      <div class="card-content" :key="selectedNodeId">
        <h2>{{ heading }}</h2>
        <div v-if="showImage && imageUrl" class="image-container">
          <img :src="imageUrl" alt="Node Image" />
        </div>
        <p>{{ description }}</p>
        <div v-if="filteredChildren && filteredChildren.length">
          <h3>Subjects:</h3>
          <ul class="child-list">
            <li
              v-for="child in filteredChildren"
              :key="child.id"
              class="child-item"
            >
              {{ child.name }}
            </li>
          </ul>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  props: {
    heading: String,
    imageUrl: String,
    description: String,
    children: Array,
    showImage: Boolean,
    isVisible: Boolean,
    selectedNodeId: Number,
    selectedNodeColor: String, // New prop for node color
  },
  computed: {
    // Computed property to filter children
    filteredChildren() {
      return this.children.filter((child) => {
        // Filter out any name starting with a number
        return !/^\d/.test(child.name);
      });
    },
  },
};
</script>

<style scoped>
/* Animate the card when it appears or disappears */
.fade-slide-card-enter-active,
.fade-slide-card-leave-active {
  transition: all 0.6s cubic-bezier(0.68, -0.55, 0.27, 1.55); /* Ease-in-out back animation for a pronounced effect */
}

.fade-slide-card-enter,
.fade-slide-card-leave-to {
  opacity: 0;
  transform: translateY(30px) scale(0.95); /* Start with a slight downward translation and scale down */
}

/* Card container styles */
.card-container {
  position: absolute;
  top: 20px;
  right: 20px;
  width: 300px;
  background-color: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(5px);
  border-radius: 10px;
  padding: 20px;
  color: white;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
  border: 1px solid white;
}

/* Animated content inside the card */
.fade-slide-card-enter-active .card-content,
.fade-slide-card-leave-active .card-content {
  transition: all 0.4s ease;
}

.fade-slide-card-enter .card-content,
.fade-slide-card-leave-to .card-content {
  opacity: 0;
  transform: translateY(15px);
}

.card-content h2 {
  font-size: 20px;
  margin-bottom: 10px;
}

.image-container img {
  width: 100%;
  height: auto;
  border-radius: 8px;
  margin-bottom: 10px;
  border: 2px solid black; /* Dark black border to the image */
}

/* Style for child list and items */
.child-list {
  padding-left: 20px;
  margin-top: 5px; /* Reduced the gap between Subjects and children */
  list-style: none; /* Remove the default bullets */
}

.child-item {
  margin-bottom: 8px;
  font-family: "Arial", sans-serif;
  font-size: 14px;
  font-weight: bold;
  position: relative;
  padding-left: 25px; /* Space for the bullet */
}

/* Custom bullet before each list item, use the selectedNodeColor prop */
.child-item::before {
  content: "●";
  color: v-bind(selectedNodeColor); /* Dynamic bullet color */
  position: absolute;
  left: 0;
  top: 0;
  font-size: 16px;
  line-height: 1;
}
</style>
