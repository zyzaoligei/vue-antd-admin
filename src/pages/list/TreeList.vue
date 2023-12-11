<template>
  <div>
    <a-input-search
      style="margin-bottom: 8px"
      placeholder="Search"
      @change="onChange"
    />
    <a-tree
      :expanded-keys="expandedKeys"
      :auto-expand-parent="autoExpandParent"
      :tree-data="gData"
      @expand="onExpand"
    >
      <template slot="title" slot-scope="{ title }">
        <span v-if="!title.endsWith(' ')" class="iconSpace"
          ><a-icon type="smile-o"
        /></span>
        <span v-if="title.indexOf(searchValue) > -1">
          {{ title.substring(0, title.indexOf(searchValue)) }}
          <span style="color: #f50">{{ searchValue }}</span>
          {{ title.substring(title.indexOf(searchValue) + searchValue.length) }}
        </span>
        <span v-else>{{ title }}</span>
      </template>
    </a-tree>
  </div>
</template>

<script>
import { gData } from "./config/index";

// const dataList = [...gData];

function catchTreeList(data, index) {
  let numIndex = index || 0;

  for (let i = 0; i < data.length; i++) {
    data[i].title = data[i].title + (numIndex === 0 ? " " : "");
    data[i].key = data[i].key + "hhhh"; // parentId + id
    data[i].scopedSlots = {
      title: "title",
    };
    if (
      data[i].children &&
      data[i].children !== null &&
      data[i].children.length > 0
    ) {
      catchTreeList(data[i].children, numIndex + 1);
    } else {
      data[i].children = null;
    }
  }
}
catchTreeList(gData);

const dataList = [];
const generateList = (data) => {
  for (let i = 0; i < data.length; i++) {
    const node = data[i];
    const key = node.key;
    const title = node.title;
    dataList.push({ key, title });
    if (node.children) {
      generateList(node.children);
    }
  }
};
generateList(gData);

console.log("树形数据：", gData);
console.log("扁平数据结构---", dataList);

const getParentKey = (key, tree) => {
  let parentKey;
  for (let i = 0; i < tree.length; i++) {
    const node = tree[i];
    if (node.children) {
      if (node.children.some((item) => item.key === key)) {
        parentKey = node.key;
      } else if (getParentKey(key, node.children)) {
        parentKey = getParentKey(key, node.children);
      }
    }
  }
  return parentKey;
};
export default {
  data() {
    return {
      expandedKeys: [],
      searchValue: "",
      autoExpandParent: true,
      gData,
    };
  },
  methods: {
    onExpand(expandedKeys) {
      this.expandedKeys = expandedKeys;
      this.autoExpandParent = false;
    },
    onChange(e) {
      const value = e.target.value;
      // console.log("输入数据：", value);
      const expandedKeys = dataList
        .map((item) => {
          if (item.title.indexOf(value) > -1) {
            return getParentKey(item.key, gData);
          }
          return null;
        })
        .filter((item, i, self) => item && self.indexOf(item) === i);

      console.log("key组合：", expandedKeys);

      Object.assign(this, {
        expandedKeys,
        searchValue: value,
        autoExpandParent: true,
      });
    },
  },
};
</script>
<style scoped>
.iconSpace {
  margin-right: 6px;
}
</style>
