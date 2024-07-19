<template>
  <li>
    <div
      @click="toggle"
      :class="nodeClasses"
    >
      <span class="icon" v-if="node.children.length">
        {{ isOpen ? '▼' : '▶' }}
      </span>
      {{ node.title }}
    </div>
    <ul v-if="isOpen && node.children.length">
      <tree-node
        v-for="(child, index) in node.children"
        :key="child.id"
        :node="child"
        :depth="depth + 1"
        :index="index"
        :global-index="globalIndex + index + 1"
        :expanded-nodes="expandedNodes"
        @toggle="$emit('toggle', $event)"
      />
    </ul>
  </li>
</template>

<script lang="ts">
import { defineComponent, ref, computed, watch } from 'vue';

interface TreeNodeProps {
  node: {
    title: string;
    parent_id: string | null;
    id: string;
    children?: TreeNodeProps['node'][];
  };
  depth: number;
  index: number;
  globalIndex: number;
  expandedNodes: Set<string>;
}

export default defineComponent({
  name: 'TreeNode',
  props: {
    node: {
      type: Object as () => TreeNodeProps['node'],
      required: true,
    },
    depth: {
      type: Number,
      required: true,
    },
    index: {
      type: Number,
      required: true,
    },
    globalIndex: {
      type: Number,
      required: true,
    },
    expandedNodes: {
      type: Object as () => Set<string>,
      required: true,
    },
  },
  emits: ['toggle'],
  setup(props: TreeNodeProps, { emit }) {
    const isOpen = ref(props.expandedNodes.has(props.node.id));
    const isRootNode = computed(() => props.node.parent_id === null);
    const isEvenChild = computed(() => props.globalIndex % 2 === 0);

    const nodeClasses = computed(() => [
      'node-title',
      {
        'root-node': isRootNode.value,
        'even-child-node': isEvenChild.value && !isRootNode.value,
        'odd-child-node': !isEvenChild.value && !isRootNode.value,
      },
    ]);

    const toggle = () => {
      isOpen.value = !isOpen.value;
      if (isOpen.value) {
        props.expandedNodes.add(props.node.id);
      } else {
        props.expandedNodes.delete(props.node.id);
      }
      localStorage.setItem('expandedNodes', JSON.stringify([...props.expandedNodes]));
      emit('toggle', props.node.id);
    };

    watch(
      () => props.expandedNodes,
      (newVal) => {
        isOpen.value = newVal.has(props.node.id);
      }
    );

    return {
      isOpen,
      toggle,
      isRootNode,
      isEvenChild,
      nodeClasses,
    };
  },
});
</script>

<style scoped>
.node-title {
  margin: 0.5rem 0;
  cursor: pointer;
  display: flex;
  align-items: center;
  padding: 10px;
  border-radius: 4px;
  width: 300px; /* Adjust width as needed */
  box-shadow: 1px 1px 3px rgba(0, 0, 0, 0.1);
  transition: background-color 0.3s;
  font-weight: normal;
}

.node-title:hover {
  background-color: #f1f1f1;
}

.root-node {
  background-color: #e4e3e8; /* Background color for root nodes */
  color: #606276; /* Font color for root nodes */
  font-weight: bold;
}

.even-child-node {
  background-color: #ffffff; /* Background color for even child nodes */
  color: #717386; /* Font color for even child nodes */
}

.odd-child-node {
  background-color: #f6f6f7; /* Background color for odd child nodes */
  color: #717386; /* Font color for odd child nodes */
}

.icon {
  margin-right: 8px;
  font-weight: bold;
}

ul {
  list-style-type: none;
  padding-left: 1rem;
  margin: 0; /* Remove margin for nested lists */
}
</style>
