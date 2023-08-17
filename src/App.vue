<template>
  <div class="graph-container">
    <div
      v-for="node in nodes"
      :key="node.id"
      class="node"
      :style="nodeStyle(node)"
      @mousedown="onNodeMouseDown(node, $event)"
      @mousemove="onNodeMouseMove(node, $event)"
      @mouseup="onNodeMouseUp(node)"
      @mouseleave="onNodeMouseUp(node)"
    >
      {{ node.id }}
    </div>
    <div v-for="(edge, index) in edges" :key="edge.id" class="edge" :style="edgeStyle(edge, index)"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

type Position = { x: number; y: number };
type Node = { id: string; position: Position };
type Edge = { id: string; startNodeId: string; endNodeId: string };

const nodes = ref<Node[]>([]);
const edges = ref<Edge[]>([]);

onMounted(async () => {
  try {
    const response = await fetch('/graph_data.json'); 
    const data = await response.json();
    nodes.value = data.nodes;
    edges.value = data.edges;
  } catch (error) {
    console.error('Error fetching data:', error);
  }
});

function nodeStyle(node: Node) {
  return {
    position: 'absolute',
    width: '30px',
    height: '30px',
    backgroundColor: 'red', 
    borderRadius: '50%',
    left: `${node.position.x * 100}px`, 
    top: `${node.position.y * 100}px`, 
    display: 'flex',
    justifyContent: 'center',
    alignItems: 'center',
    color: '#fff', 
    fontWeight: 'bold',
    cursor: 'move', 
    userSelect: 'none' 
  };
}

type EdgeStyle = {
  position: 'absolute';
  width: string;
  height: string;
  backgroundColor: string;
  left: string;
  top: string;
  transformOrigin: string;
  transform: string;
};

function edgeStyle(edge: Edge, index: number): EdgeStyle {
  const startNode = nodes.value.find((node) => node.id === edge.startNodeId);
  const endNode = nodes.value.find((node) => node.id === edge.endNodeId);

  if (!startNode || !endNode) {
    return {
      position: 'absolute',
      width: '0',
      height: '0',
      backgroundColor: 'transparent',
      left: '0',
      top: '0',
      transform: 'none'
    };
  }

  const deltaX = endNode.position.x - startNode.position.x;
  const deltaY = endNode.position.y - startNode.position.y;
  const length = Math.sqrt(deltaX ** 2 + deltaY ** 2);
  const angle = Math.atan2(deltaY, deltaX);

  return {
    position: 'absolute',
    width: `${length * 100}px`,
    height: '2px', 
    backgroundColor: '#000',
    left: `${startNode.position.x * 100 + 15}px`, 
    top: `${startNode.position.y * 100 + 15}px`, 
    transformOrigin: '0 0',
    transform: `rotate(${angle}rad)`
  };
}


let activeNode: Node | null = null;
let offsetX: number = 0;
let offsetY: number = 0;

function onNodeMouseDown(node: Node, event: MouseEvent) {
  activeNode = node;
  offsetX = event.pageX - node.position.x * 100;
  offsetY = event.pageY - node.position.y * 100;
}
// unteren Teil
function onNodeMouseMove(node: Node, event: MouseEvent) {
  if (activeNode === node) {
    node.position.x = (event.pageX - offsetX) / 100;
    node.position.y = (event.pageY - offsetY) / 100;
  }
}

function onNodeMouseUp(node: Node) {
  if (activeNode === node) {
    activeNode = null;
  }
}
</script>

<style scoped>
.graph-container {
  position: relative;
  width: 100%;
  height: 100%;
}

.node {
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #ccc;
  border-radius: 50%;
  background-color: red;
  color: #fff;
  width: 30px;
  height: 30px;
  position: absolute;
}

.edge {
  position: absolute;
  background-color: #000;
}
</style>
