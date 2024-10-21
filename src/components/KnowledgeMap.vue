<template>
  <div ref="graphContainer" class="graph-container">
    <h1 class="knowledge-map-heading">Knowledge Map</h1>
    <svg ref="svg"></svg>
    <!-- Heading for the knowledge map -->

    <!-- Card Component for displaying node details -->
    <CardComponent
      :heading="selectedNodeName"
      :imageUrl="selectedNodeImage"
      :description="selectedNodeDescription"
      :children="selectedNodeChildren"
      :showImage="selectedNode?.main"
      :isVisible="cardVisible"
    />
  </div>
</template>

<script>
import * as d3 from "d3";
import CardComponent from "./CardComponent.vue";

export default {
  name: "KnowledgeMap",
  components: { CardComponent },
  data() {
    return {
      nodes: [
        { id: 0, name: "0. Foundations", main: true },
        { id: 1, name: "1. Basic ROS2", main: true },
        { id: 2, name: "2. ROS Basics", main: true },
        { id: 3, name: "3. Intermediate ROS2", main: true },
        { id: 4, name: "4. Robotics Theory", main: true },
        { id: 5, name: "5. Navigation ROS2", main: true },
        { id: 6, name: "7. Manipulation", main: true },
        { id: 7, name: "8. Robot Creation", main: true },
        { id: 8, name: "9. Artificial Intelligence", main: true },
        { id: 9, name: "10. ROS Debugging", main: true },
        { id: 10, name: "11. Course of Product", main: true },
        { id: 11, name: "12. Web Devel for Robots", main: true },
        { id: 12, name: "13. Simulation", main: true },
        { id: 13, name: "14. Enterprise", main: true },
        { id: 14, name: "Python3 for Robots", main: false },
        { id: 15, name: "Linux for Robotics", main: false },
        { id: 16, name: "C++ Basics for Robotics", main: false },
        { id: 17, name: "Gazebo Sim with ROS2", main: false },
        { id: 18, name: "TF in ROS2", main: false },
        { id: 19, name: "URDF Robot Modelling ROS2", main: false },
        { id: 20, name: "ROS2 Basics in 3 Days (Rust)", main: false },
        { id: 21, name: "ROS2 Basics in 5 Days (Python)", main: false },
        { id: 22, name: "ROS2 Basics in 5 Days (C++)", main: false },
        { id: 23, name: "ROS2 Security", main: false },
        { id: 24, name: "ROS1-ROS2 Migration", main: false },
        { id: 25, name: "Behavior Trees for ROS2", main: false },
        { id: 26, name: "Advanced Modern C++ for robotics", main: false },
        { id: 27, name: "Intermediate ROS2 (Python)", main: false },
        { id: 28, name: "Intermediate ROS2 (C++)", main: false },
        { id: 29, name: "ROS2 Control", main: false },
        { id: 30, name: "RTAB", main: false },
        { id: 31, name: "Nav2 Basics", main: false },
        { id: 32, name: "Nav Advanced", main: false },
      ],
      links: [
        { source: 0, target: 1 },
        { source: 0, target: 2 },
        { source: 0, target: 14 },
        { source: 0, target: 15 },
        { source: 0, target: 16 },

        { source: 1, target: 3 },
        { source: 1, target: 5 },
        { source: 1, target: 18 },
        { source: 1, target: 19 },
        { source: 1, target: 20 },
        { source: 1, target: 21 },
        { source: 1, target: 22 },

        { source: 2, target: 17 },

        { source: 3, target: 23 },
        { source: 3, target: 24 },
        { source: 3, target: 25 },
        { source: 3, target: 26 },
        { source: 3, target: 27 },
        { source: 3, target: 28 },
        { source: 3, target: 29 },

        { source: 5, target: 30 },
        { source: 5, target: 31 },
        { source: 5, target: 32 },
      ],
      isSimulationActive: false,
      simulation: null,
      blurEffectActive: false,
      selectedNode: null, // Store the selected node

      selectedNodeName: "",

      selectedNodeImage: "",

      selectedNodeDescription: "",

      selectedNodeChildren: [],

      cardVisible: false, // Control the card visibility
    };
  },
  mounted() {
    this.createGraph();
    window.addEventListener("resize", this.resizeGraph);
    document.addEventListener("click", this.handleOutsideClick);
  },
  beforeUnmount() {
    window.removeEventListener("resize", this.resizeGraph);
    document.removeEventListener("click", this.handleOutsideClick);
    if (this.simulation) {
      this.simulation.stop();
    }
  },
  methods: {
    setInitialPositions(nodes, width, height) {
      // Position nodes in a circular layout around the center
      nodes.forEach((node, index) => {
        const angle = (Math.PI + index) / nodes.length;
        node.x = width + 100 * Math.cos(angle);
        node.y = height + 100 * Math.sin(angle);
      });
    },

    updateLabels(node, link) {
      const padding = 10;

      const isColliding = (textNode) => {
        const bbox = textNode.getBBox();
        return node.some((otherNode) => {
          if (otherNode === textNode) return false; // Skip self
          const otherBbox = otherNode.getBBox();
          return !(
            bbox.x > otherBbox.x + otherBbox.width + padding ||
            bbox.x + bbox.width < otherBbox.x - padding ||
            bbox.y > otherBbox.y + otherBbox.height + padding ||
            bbox.y + bbox.height < otherBbox.y - padding
          );
        });
      };

      // Iterate over each text element
      link.selectAll("text").each(function () {
        const textNode = this; // Reference to the current text node
        // Check for collisions
        if (isColliding(textNode)) {
          // Adjust the position or style of the text node if needed
          d3.select(textNode).style("opacity", 0); // Example action: hide if colliding
        } else {
          d3.select(textNode).style("opacity", 1); // Show if not colliding
        }
      });
    },

    createGraph() {
      const svg = d3.select(this.$refs.svg);
      svg.selectAll("*").remove();
      const { nodes, links } = this;
      const width = window.innerWidth;
      const height = window.innerHeight;

      svg
        .attr("width", width)
        .attr("height", height)
        .style("background-color", "#121212");

      svg.call(this.dragSvg(svg));

      this.setInitialPositions(nodes, width / 2, height);

      // Adjust attraction forces: shorter distance for parent nodes, longer for child nodes
      const linkDistance = (d) => {
        const isParentLink = d.source.main && d.target.main;
        const isChildLink =
          (d.source.main && !d.target.main) ||
          (!d.source.main && d.target.main);
        return isParentLink ? 210 : isChildLink ? 100 : 90;
      };

      // Start the simulation immediately
      const simulation = d3
        .forceSimulation(nodes)
        .force(
          "link",
          d3
            .forceLink(links)
            .id((d) => d.id)
            .distance(linkDistance)
        )
        .force(
          "charge",
          d3.forceManyBody().strength((d) => (d.main ? -300 : -300))
        )
        .force("center", d3.forceCenter(width / 2, height / 2))
        .force("collision", d3.forceCollide().radius(60))
        .force("x", d3.forceX(width / 2).strength(0.05))
        .force("y", d3.forceY(height / 2).strength(0.05))
        .alphaDecay(0.01);

      this.simulation = simulation;

      const link = svg
        .append("g")
        .attr("stroke-opacity", 0.6)
        .selectAll("line")
        .data(links)
        .join("line")
        .attr("stroke-width", (d) => (d.source.main && d.target.main ? 2.8 : 1))
        .attr("stroke", (d) => {
          // Check if both source and target are parent nodes
          if (d.source.main && d.target.main) {
            return "red"; // Edge color between two parent nodes
          }
          // Check if one is a parent and the other is a child
          if (
            (d.source.main && !d.target.main) ||
            (!d.source.main && d.target.main)
          ) {
            return "grey"; // Edge color between parent and child
          }
          return "orange"; // Default edge color
        });

      const node = svg
        .append("g")
        .selectAll("g")
        .data(nodes)
        .join("g")
        .call(this.drag(simulation));

      node
        .append("circle")
        .attr("r", (d) => (d.main ? 8 * 1.5 : 8))

        .attr("fill", (d) => {
          // Check if the node is a parent and if it has any connections
          const hasConnections = this.links.some(
            (link) => link.source.id === d.id || link.target.id === d.id
          );
          if (d.main && !hasConnections) {
            return "orange"; // Color for parent nodes with no connections
          }
          return d.main ? "#020079" : "#A0D683"; // Parent or child node color
        })

        .attr("stroke", "black")
        .attr("stroke-width", 1.5)
        .on("click", (event, d) => this.selectNode(d, node, link));

      node
        .filter(
          (d) => d.main && !this.links.some((link) => link.source.id === d.id)
        )
        .append("circle")
        .attr("r", 3) // Radius for the inner dot
        .attr("fill", "red");

      node
        .append("text")
        .attr("x", 20)
        .attr("y", 3)
        .text((d) => d.name)
        .style("font-size", (d) => (d.main ? "18px" : "12px"))
        .style("font-weight", (d) => (d.main ? "bold" : "normal"))
        .style("fill", "white");

      simulation.on("tick", () => {
        link
          .attr("x1", (d) => d.source.x)
          .attr("y1", (d) => d.source.y)
          .attr("x2", (d) => d.target.x)
          .attr("y2", (d) => d.target.y);

        node.attr("transform", (d) => translate(${d.x},${d.y}));

        const padding = 20;
        nodes.forEach((d) => {
          if (d.x < padding) {
            d.vx += (padding - d.x) * 0.2; // Increase the force towards the center
          } else if (d.x > width - padding) {
            d.vx -= (d.x - (width - padding)) * 0.2;
          }
          if (d.y < padding) {
            d.vy += (padding - d.y) * 0.2;
          } else if (d.y > height - padding) {
            d.vy -= (d.y - (height - padding)) * 0.2;
          }
        });
      });
    },

    dragSvg(svg) {
      let dx = 0,
        dy = 0; // Store translation values
      let currentTransform = [0, 0];
      return d3
        .drag()
        .on("start", function (event) {
          dx = event.x;
          dy = event.y;

          const transform = svg.attr("transform");
          if (transform) {
            const translateValues = transform.match(
              /translate\(([^,]+),([^)]+)\)/
            );
            if (translateValues) {
              currentTransform = [
                parseFloat(translateValues[1]),
                parseFloat(translateValues[2]),
              ];
            }
          }
        })
        .on("drag", function (event) {
          const newTranslateX = currentTransform[0] + (event.x - dx);
          const newTranslateY = currentTransform[1] + (event.y - dy);

          // Apply the new transform
          svg.attr(
            "transform",
            translate(${newTranslateX}, ${newTranslateY})
          );
        });
    },

    drag(simulation) {
      function dragstarted(event) {
        if (!event.active) simulation.alphaTarget(0.3).restart();
        event.subject.fx = event.subject.x;
        event.subject.fy = event.subject.y;
        this.updateLabels(d3.selectAll("g"), d3.selectAll("line"));
        this.selectNode(event.subject, d3.selectAll("g"), d3.selectAll("line"));
      }

      function dragged(event) {
        event.subject.fx = event.x;
        event.subject.fy = event.y;
        this.updateLabels(d3.selectAll("g"), d3.selectAll("line"));
      }

      function dragended(event) {
        if (!event.active) simulation.alphaTarget(0);
        event.subject.fx = null;
        event.subject.fy = null;
        this.resetBlurEffects();
        this.blurEffectActive = false;
      }

      return d3
        .drag()
        .on("start", dragstarted.bind(this))
        .on("drag", dragged.bind(this))
        .on("end", dragended.bind(this));
    },

    selectNode(d, node, link) {
      // Check if the node has children
      const hasChildren = this.links.some((link) => link.source.id === d.id);
      // Find the parent link if it exists
      const parentLink = this.links.find((link) => link.target.id === d.id);

      if (!hasChildren) {
        // If the node has no children, blur all other nodes
        node
          .selectAll("circle")
          .attr("opacity", (n) =>
            n.id === d.id || (parentLink && n.id === parentLink.source.id)
              ? 1
              : 0.3
          )
          .style("filter", (n) =>
            n.id === d.id || (parentLink && n.id === parentLink.source.id)
              ? "none"
              : "blur(2.1px)"
          );

        node
          .selectAll("text")
          .style("filter", (n) =>
            n.id === d.id || (parentLink && n.id === parentLink.source.id)
              ? "none"
              : "blur(2.1px)"
          );
        // Ensure the parent node's text remains unblurred
        if (parentLink) {
          const parentNodeId = parentLink.source.id;
          node
            .selectAll("text")
            .style("filter", (n) =>
              n.id === parentNodeId
                ? "none"
                : n.id === d.id
                ? "none"
                : "blur(2.1px)"
            );
        }

        link
          .attr("stroke-opacity", (l) => {
            // Keep the parent link fully visible
            if (
              parentLink &&
              l.source.id === parentLink.source.id &&
              l.target.id === parentLink.target.id
            ) {
              return 1;
            }
            return 0.3;
          })

          .style("filter", (l) => {
            // Don't blur the parent link
            if (
              parentLink &&
              l.source.id === parentLink.source.id &&
              l.target.id === parentLink.target.id
            ) {
              return "none";
            }
            return "blur(2.1px)";
          });
      } else {
        // The original blur logic when the node has children

        const connectedNodes = new Set();

        this.links.forEach((link) => {
          if (link.source.id === d.id) {
            connectedNodes.add(link.target.id); // Child nodes
          }
          if (link.target.id === d.id) {
            connectedNodes.add(link.source.id); // Parent nodes
          }
        });

        node
          .selectAll("circle")
          .attr("opacity", (n) => {
            if (n.id === d.id || connectedNodes.has(n.id)) {
              return 1;
            }
            return 0.3;
          })
          .style("filter", (n) => {
            if (n.id === d.id || connectedNodes.has(n.id)) {
              return "none";
            }
            return "blur(2.1px)";
          });

        node.selectAll("text").style("filter", (n) => {
          if (n.id === d.id || connectedNodes.has(n.id)) {
            return "none";
          }
          return "blur(2.1px)";
        });

        link
          .attr("stroke-opacity", (l) => {
            if (
              l.source.id === d.id ||
              l.target.id === d.id ||
              connectedNodes.has(l.target.id) ||
              connectedNodes.has(l.source.id)
            ) {
              return 1;
            }
            return 0.3;
          })
          .style("filter", (l) => {
            if (
              l.source.id === d.id ||
              l.target.id === d.id ||
              connectedNodes.has(l.target.id) ||
              connectedNodes.has(l.source.id)
            ) {
              return "none";
            }
            return "blur(2.1px)";
          });
      }

      this.blurEffectActive = true;

      if (!this.isSimulationActive) {
        this.simulation.alpha(0.1).restart();
        this.isSimulationActive = true;
      }

      if (d.main || d.id <= 14) {
        // For nodes where main: true or ID <= 14, show image and description
        this.selectedNode = d;
        this.selectedNodeName = d.name;
        this.selectedNodeImage = this.getImageForNode(d); // Only show image for main nodes
        this.selectedNodeDescription = this.generateDescription(d); // Show description

        // Find child nodes
        const childNodes = this.links
          .filter((link) => link.source.id === d.id)
          .map((link) => this.nodes.find((node) => node.id === link.target.id));
        this.selectedNodeChildren = childNodes;

        // Show the card for any selected node
        this.cardVisible = true;
      } else if (d.id > 14) {
        // For nodes with ID > 14 (not main), show description without image
        this.selectedNode = d;
        this.selectedNodeName = d.name;
        this.selectedNodeImage = ""; // No image for child nodes
        this.selectedNodeDescription = this.generateDescription(d);

        // Find child nodes
        const childNodes = this.links
          .filter((link) => link.source.id === d.id)
          .map((link) => this.nodes.find((node) => node.id === link.target.id));
        this.selectedNodeChildren = childNodes;

        // Show the card for non-main nodes as well
        this.cardVisible = true;
      } else {
        // In case no valid node is selected, hide the card
        this.cardVisible = false;
      }
    },

    handleOutsideClick(event) {
      const nodes = this.$refs.svg.querySelectorAll("circle");
      const card = this.$refs.card;

      let clickedOnNode = false;
      let clickedOnCard = card && card.contains(event.target);
      nodes.forEach((node) => {
        if (node.contains(event.target)) {
          clickedOnNode = true;
        }
      });

      if (this.blurEffectActive && !clickedOnNode) {
        console.log("Click detected outside nodes, resetting blur effects");
        this.resetBlurEffects(); // Reset all styles if blur effect is active
        this.blurEffectActive = false; // Reset blur effect flag
      }
      if (!clickedOnCard && !clickedOnNode) {
        this.cardVisible = false; // Hide the card if clicking outside
      }
    },
    resetBlurEffects() {
      // Reset nodes and links to their default styles
      const svg = d3.select(this.$refs.svg);
      svg.selectAll("circle").attr("opacity", 1).style("filter", "none");
      svg.selectAll("text").style("filter", "none");
      svg.selectAll("line").attr("stroke-opacity", 1).style("filter", "none");
    },
    resizeGraph() {
      this.createGraph();
    },

    getImageForNode(node) {
      const images = {
        0: https://dummyimage.com/300x200/000/fff&text=Node+0,
        1: https://dummyimage.com/300x200/000/fff&text=Node+1,
        2: https://dummyimage.com/300x200/000/fff&text=Node+2,
        3: https://dummyimage.com/300x200/000/fff&text=Node+3,
        4: https://dummyimage.com/300x200/000/fff&text=Node+4,
        5: https://dummyimage.com/300x200/000/fff&text=Node+5,
        6: https://dummyimage.com/300x200/000/fff&text=Node+6,
        7: https://dummyimage.com/300x200/000/fff&text=Node+7,
        8: https://dummyimage.com/300x200/000/fff&text=Node+8,
        9: https://dummyimage.com/300x200/000/fff&text=Node+9,
        10: https://dummyimage.com/300x200/000/fff&text=Node+10,
        11: https://dummyimage.com/300x200/000/fff&text=Node+11,
        12: https://dummyimage.com/300x200/000/fff&text=Node+12,
        13: https://dummyimage.com/300x200/000/fff&text=Node+13,
        14: https://dummyimage.com/300x200/000/fff&text=Node+14,
      };
      return images[node.id] || "";
    },

    // Function to generate a description for each node
    generateDescription(node) {
      const descriptions = {
        0: "This section covers the foundations of robotics, AI, and their intersection.",
        1: "Introduction to basic ROS2 concepts and tutorials for beginners.",
        2: "A detailed overview of ROS basics and fundamental robotic concepts.",
        3: "Intermediate ROS2 topics, including advanced message passing and node interaction.",
        4: "Robotics theory involving kinematics, dynamics, and control systems.",
        5: "ROS2 Navigation and path-planning for autonomous robots.",
        6: "Manipulation of robotic arms and understanding the basics of robotic control.",
        7: "Creating your own robots with ROS2, including hardware and software integration.",
        8: "Artificial Intelligence and its applications in modern robotics.",
        9: "Debugging ROS2 programs and understanding error handling and troubleshooting.",
        10: "Product lifecycle course, from development to deployment in robotics.",
        11: "Web development skills specifically for robotics applications and control.",
        12: "Simulation environments for robotics, focusing on Gazebo and other tools.",
        13: "Enterprise-level robotics, scaling up solutions for industrial needs.",
        14: "Learning Python3 for robotics programming and integration with ROS.",
        15: "Linux commands and tools for robotics, including file handling, permissions, and scripting.",
        16: "C++ programming basics, including object-oriented concepts for writing efficient robotics code.",
        17: "Gazebo simulation with ROS2, focusing on environment creation, sensor integration, and robot control.",
        18: "TF (Transform) library in ROS2, used for tracking robot frames and transformations between coordinates.",
        19: "URDF (Unified Robot Description Format) for modeling robots in ROS2 and defining joint and link configurations.",
        20: "Learn ROS2 basics using Rust, a modern and safe systems programming language.",
        21: "ROS2 basics in 5 days with Python, offering a quick start guide for Python developers.",
        22: "ROS2 basics in 5 days with C++, providing a foundation for using ROS2 with C++ programming.",
        23: "ROS2 security concepts, focusing on secure communication, encryption, and authentication mechanisms.",
        24: "Migration guide from ROS1 to ROS2, covering differences, best practices, and migration strategies.",
        25: "Behavior Trees in ROS2 for structuring complex robotic behavior in an organized and modular way.",
        26: "Advanced modern C++ programming concepts applied to robotics, including memory management and concurrency.",
        27: "Intermediate ROS2 concepts with Python, diving into more advanced topics like action servers and services.",
        28: "Intermediate ROS2 with C++, covering more advanced topics such as message filters, action servers, and multithreading.",
        29: "ROS2 Control for managing hardware interfaces and controllers in robotic systems.",
        30: "RTAB (Real-Time Appearance-Based Mapping) for ROS2, focusing on SLAM (Simultaneous Localization and Mapping).",
        31: "Nav2 Basics in ROS2 for autonomous navigation, path planning, and obstacle avoidance.",
        32: "Advanced Navigation in ROS2, covering complex path-planning algorithms and multi-robot coordination.",
      };
      return descriptions[node.id] || "No description available.";
    },
  },
};
</script>

<style>
body {
  background-color: #121212;
}

.knowledge-map-heading {
  color: white;
  text-align: center;
  margin-top: 20px;
  font-size: 36px;
  font-weight: bold;
  font-family: "Courier New", Courier, monospace;
  letter-spacing: 1px;
  text-transform: uppercase;
  animation: slideIn 2s ease-out forwards;
  opacity: 0; /* Set initial opacity to 0 */
}

/* Keyframes for slide-in and fade-in animation */
@keyframes slideIn {
  0% {
    transform: translateY(-50px); /* Start 50px above the final position */
    opacity: 0; /* Start with invisible */
  }
  100% {
    transform: translateY(0); /* End at the original position */
    opacity: 1; /* Fully visible */
  }
}

/* Additional styles for the graph container */
.graph-container {
  position: relative;
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
}
</style>