<template>
  <div>
    <button @click="rerender" class="rerender-button">Rerender</button>
    <ul>
      <tree-node
        v-for="(node, index) in treeData"
        :key="node.id"
        :node="node"
        :depth="0"
        :index="index"
        :global-index="index"
        :expanded-nodes="expandedNodes"
        @toggle="handleToggle"
      />
    </ul>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue';
import axios from 'axios';
import TreeNode from './TreeNode.vue';

interface TreeNodeData {
  title: string;
  parent_id: string | null;
  id: string;
  children?: TreeNodeData[];
}

export default defineComponent({
  name: 'TreeComponent',
  components: {
    TreeNode,
  },
  setup() {
    const treeData = ref<TreeNodeData[]>([]);
    const expandedNodes = ref<Set<string>>(new Set(JSON.parse(localStorage.getItem('expandedNodes') || '[]')));

    const fetchTreeData = async () => {
      const { data } = await axios.get<TreeNodeData[]>('https://64b4c8450efb99d862694609.mockapi.io/tree/items');
      treeData.value = buildTree(data);
    };

    const buildTree = (data: TreeNodeData[]): TreeNodeData[] => {
      const tree: TreeNodeData[] = [];
      const lookup: { [key: string]: TreeNodeData } = {};

      data.forEach(item => {
        lookup[item.id] = { ...item, children: [] };
      });

      data.forEach(item => {
        if (item.parent_id === null) {
          tree.push(lookup[item.id]);
        } else {
          lookup[item.parent_id]?.children?.push(lookup[item.id]);
        }
      });

      return tree;
    };

    const handleToggle = (id: string) => {
      if (expandedNodes.value.has(id)) {
        expandedNodes.value.delete(id);
      } else {
        expandedNodes.value.add(id);
      }
    };

    const rerender = async () => {
      treeData.value = [];
      await fetchTreeData();
    };

    onMounted(() => {
      fetchTreeData();
    });

    return {
      treeData,
      expandedNodes,
      handleToggle,
      rerender,
    };
  },
});
</script>

<style scoped>
.rerender-button {
  margin-bottom: 10px;
  padding: 8px 16px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.rerender-button:hover {
  background-color: #0056b3;
}

ul {
  list-style-type: none;
  padding-left: 1rem;
}
</style>
