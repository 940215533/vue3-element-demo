<!-- 部门树 -->
<template>
  <el-card shadow="never">
    <el-input v-model="deptName" placeholder="部门名称" clearable>
      <template #prefix>
        <el-icon><Search /></el-icon>
      </template>
    </el-input>

    <el-tree
      ref="deptTreeRef"
      class="mt-2"
      :data="deptList"
      :props="{ children: 'children', label: 'label', disabled: '' }"
      :expand-on-click-node="false"
      :filter-node-method="handleFilter"
      default-expand-all
      @node-click="handleNodeClick"
    />
  </el-card>
</template>

<script setup lang="ts">
import DeptAPI from "@/api/system/dept";
const props = defineProps({
  modelValue: {
    type: [Number],
    default: undefined,
  },
});

const deptList = ref<OptionType[]>(); // 部门列表
const deptTreeRef = ref(ElTree); // 部门树
const deptName = ref(); // 部门名称

const emits = defineEmits(["node-click"]);

const deptId = useVModel(props, "modelValue", emits);

watchEffect(
  () => {
    deptTreeRef.value.filter(deptName.value);
  },
  {
    // watchEffect默认会在DOM挂载或者更新之前就会触发，此属性控制回调函数在DOM元素更新后运行
    flush: "post",
  }
);

/**
 * 部门筛选
 * @param {string} value - 用于筛选的关键字
 * @param {any} data - 当前节点的数据
 * @returns {boolean} - 如果节点应显示则返回 true，否则返回 false
 */
function handleFilter(value: string, data: any) {
  // 如果没有输入筛选关键字，则显示所有节点
  if (!value) {
    return true;
  }
  // 检查节点的 label 属性是否包含筛选关键字
  return data.label.indexOf(value) !== -1;
}

/**
 * 部门树节点 Click 事件处理函数
 * @param {Object} data - 点击节点的数据，键值对形式
 */
function handleNodeClick(data: { [key: string]: any }) {
  // 将点击节点的值赋值给 deptId
  deptId.value = data.value;
  // 触发自定义的 node-click 事件
  emits("node-click");
}

/**
 * 在组件挂载前执行的生命周期钩子
 * 用于获取部门列表数据
 */
onBeforeMount(() => {
  // 调用 DeptAPI 的 getOptions 方法获取部门列表数据
  DeptAPI.getOptions().then((data) => {
    // 将获取到的数据赋值给 deptList
    deptList.value = data;
  });
});
</script>
