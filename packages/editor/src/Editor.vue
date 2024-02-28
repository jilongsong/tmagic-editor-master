<template>
  <div class="editor-container">
    <div class="header">
      <div class="layout-controls"></div>
    </div>

    <!-- 根据layout状态决定是否渲染 -->
    <transition name="fade">
      <div class="left-sidebar">Left Sidebar (工具栏等)</div>
    </transition>

    <div class="main-content">
      <div ref="target" style="width: 100px; height: 100px">
        <div ref="initMoveable"></div>
      </div>
    </div>

    <!-- 同理，根据layout状态决定是否渲染 -->
    <transition name="fade">
      <div class="right-sidebar">Right Sidebar (预览区域或其他功能)</div>
    </transition>
  </div>
</template>

<script lang="ts" setup>
import { computed, onBeforeUnmount, onMounted, ref } from 'vue';
import VanillaMoveable from 'moveable';
defineOptions({
  name: 'MEditor',
});

interface Position {
  left: number;
  top: number;
}

interface Rect {
  width: number | string;
  height: number | string;
}

const props = withDefaults(defineProps<{ visible: boolean; position?: Position; rect?: Rect; title?: string }>(), {
  visible: false,
  title: '',
  position: () => ({ left: 0, top: 0 }),
  rect: () => ({ width: 'auto', height: 'auto' }),
});

const style = computed(() => ({
  left: `${props.position.left}px`,
  top: `${props.position.top}px`,
  width: typeof props.rect.width === 'string' ? props.rect.width : `${props.rect.width}px`,
  height: typeof props.rect.height === 'string' ? props.rect.height : `${props.rect.height}px`,
}));

console.log(style);

const dragTarget = ref<HTMLDivElement>();
const target = ref<HTMLDivElement>();
let moveable: VanillaMoveable | null = null;

const initMoveable = () => {
  moveable = new VanillaMoveable(document.querySelector('.main-content') as HTMLElement, {
    className: 'm-editor-floating-box-moveable',
    target: target.value,
    draggable: true,
    resizable: true,
    edge: true,
    keepRatio: false,
    origin: false,
    snapGap: true,
    snappable: true,
    dragTarget: dragTarget.value,
    dragTargetSelf: false,
    linePadding: 10,
    controlPadding: 10,
    verticalGuidelines: [0, 200, 400],
    horizontalGuidelines: [0, 200, 400],
  });
  moveable.on('drag', (e) => {
    e.target.style.transform = e.transform;
  });

  moveable.on('resize', (e) => {
    e.target.style.width = `${e.width}px`;
    e.target.style.height = `${e.height}px`;
    e.target.style.transform = e.drag.transform;
  });
};

const destroyMoveable = () => {
  moveable?.destroy();
  moveable = null;
};

onBeforeUnmount(() => {
  destroyMoveable();
});

onMounted(() => {
  initMoveable();
});

// 控制布局的变量
</script>
<style>
.editor-container {
  display: grid;
  grid-template-columns: minmax(284px, 250px) 1fr minmax(250px, 250px);
  grid-template-rows: auto 1fr; /* 头部和主体部分 */
  height: 100vh;
  overflow-x: hidden; /* 隐藏水平滚动条 */
  background-color: #141517;
}

.header {
  grid-row: 1 / 2;
  grid-column: 1 / -1; /* 横跨所有列 */
  background-color: #141517;
  padding: 1rem;
  text-align: center;
  font-size: 1.2em;
  font-weight: bold;
  margin: 2px 2px;
  border-bottom: 1px solid rgb(55, 58, 64);
  /* box-shadow: rgba(0, 0, 0, 0.05) 0px 1px 3px, rgba(0, 0, 0, 0.05) 0px 10px 15px -5px,
    rgba(0, 0, 0, 0.04) 0px 7px 7px -5px; */
}

.left-sidebar,
.main-content,
.right-sidebar {
  padding: 1rem;
}

.left-sidebar {
  width: 250px;
  background-color: #141517;
  border-right: 1px solid rgb(55, 58, 64);
  z-index: 10;
}

.main-content {
  background-color: rgb(182, 181, 181);
  overflow-y: auto;
}

.right-sidebar {
  width: 250px;
  background-color: #141517;
  border-left: 1px solid rgb(55, 58, 64);
  z-index: 10;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
