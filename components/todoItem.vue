<script setup>
import { dayjs } from 'element-plus'

const props = defineProps({
  id: {
    type: String,
    default: '',
  },
  date: {
    type: String,
    default: '',
  },
  undo: {
    type: Boolean,
    default: true,
  },
  width: {
    type: Number,
    default: 1,
  },
  item: {
    type: String,
    default: '',
  },
  background: {
    type: String,
    default: '',
  },
  showSelect: {
    type: Boolean,
    default: false,
  },
})

const taskColors = {
  1: '#B0C4DE',
  2: '#67C23A',
  3: '#FFA500',
  4: '#FF0F50',
}

const taskTexts = [
  'Eliminate',
  'Delegate',
  'Schedule',
  'Do Immediately',
]

const iconPrefix = 'i-line-md-'
const finishIconName = computed(() => `${iconPrefix}${props.undo ? 'circle' : 'circle-to-confirm-circle-transition'}`)

function getDateOffset() {
  const today = dayjs().startOf('day').valueOf()
  const propDate = dayjs(props.date).valueOf()
  const offset = propDate > today
    ? 1
    : propDate === today
      ? 0
      : -1
  return offset
}

const form = reactive({
  item: props.item,
  width: props.width,
  undo: props.undo,
  background: props.background,
  todoDateOffset: getDateOffset(),
})

const rules = {
  name: [
    { required: true, message: 'Please input Todo Item', trigger: 'blur' },
    { min: 1, max: 150, message: 'Length should be 1 to 150', trigger: 'blur' },
  ],
}

const dialogFormVisible = ref(false)
const itemLoading = ref(false)

function updatePage() {
  emitter.emit('update-balance')
  emitter.emit('update-todo-list')
}

async function setItemUndo() {
  itemLoading.value = true
  emitter.emit('update-todo-item-operate-lock-status', true)
  const date = dayjs(props.date).valueOf()
  const undo = !props.undo

  setTodoItem({
    item: props.item,
    date,
    undo,
    width: props.width,
    background: props.background,
    id: props.id,
  }).then((res) => {
    updatePage()
  }).catch((err) => {
    ElMessage.error(err?.message)
  }).finally(() => {
    itemLoading.value = false
    emitter.emit('update-todo-item-operate-lock-status', false)
  })
}

async function setItem() {
  itemLoading.value = true
  emitter.emit('update-todo-item-operate-lock-status', true)
  dialogFormVisible.value = false
  let date = dayjs(props.date).valueOf()

  if (form.todoDateOffset > -1) {
    const now = dayjs().add(form.todoDateOffset, 'day')
    const day = now.startOf('day')
    date = day.valueOf()
  }

  setTodoItem({
    item: form.item,
    date,
    undo: form.undo,
    width: form.width,
    background: '',
    id: props.id,
  }).then((res) => {
    updatePage()
  }).catch((err) => {
    ElMessage.error(err?.message)
  }).finally(() => {
    itemLoading.value = false
    emitter.emit('update-todo-item-operate-lock-status', false)
  })
}

async function removeItem() {
  itemLoading.value = true
  emitter.emit('update-todo-item-operate-lock-status', true)

  removeTodoItem(props.id).then((res) => {
    updatePage()
  }).catch((err) => {
    ElMessage.error(err?.message)
  }).finally(() => {
    itemLoading.value = false
    emitter.emit('update-todo-item-operate-lock-status', false)
  })
}

function onDialogOpen() {
  form.item = props.item
  form.undo = props.undo
  form.background = props.background
  form.width = props.width
  form.todoDateOffset = getDateOffset()
}

function closeDialog() {
  dialogFormVisible.value = false
}
</script>

<template>
  <div v-loading="itemLoading" class="todo-item" :class="[{ 'is-finish': !undo }]" element-loading-custom-class="loading">
    <div class="item-content">
      <ElText truncated :style="{ color: taskColors[width] }">
        {{ item }}
      </ElText>
    </div>

    <div v-show="!showSelect" class="operates">
      <ElPopconfirm title="Are you sure to delete this?" @confirm="removeItem">
        <template #reference>
          <Icon class="remove-icon" name="i-line-md-close" style="color: #FF0F50;" />
        </template>
      </ElPopconfirm>
      <Icon class="setting-icon" name="i-line-md-cog" @click="dialogFormVisible = true" />
      <Icon class="undo-icon" :name="finishIconName" @click="setItemUndo" />
    </div>
  </div>

  <ElDialog v-model="dialogFormVisible" title="Edit Todo Item" width="500" destroy-on-close @open="onDialogOpen">
    <ElForm :model="form" :rules="rules">
      <ElFormItem>
        <ElInput
          v-model="form.item"
          placeholder="add todo item"
          maxlength="150"
          show-word-limit
          clearable
          type="textarea"
          :rows="5"
        />
      </ElFormItem>
      <ElFormItem>
        <ElRate v-model="form.width" :max="4" show-text :texts="taskTexts" :colors="taskColors" />
      </ElFormItem>
      <ElFormItem>
        <ElSwitch
          :model-value="!form.undo"
          active-text="Done"
          inactive-text="Undone"
          @change="val => form.undo = !val"
        />
      </ElFormItem>
      <ElFormItem>
        <ElRadioGroup v-model="form.todoDateOffset" size="small">
          <ElRadioButton label="today" :value="0" />
          <ElRadioButton label="tomorrow" :value="1" />
        </ElRadioGroup>
      </ElFormItem>
    </ElForm>
    <template #footer>
      <div class="dialog-footer">
        <el-button @click="closeDialog">
          Cancel
        </el-button>
        <el-button type="primary" @click="setItem">
          Confirm
        </el-button>
      </div>
    </template>
  </ElDialog>
</template>

<style lang="scss" scoped>
.todo-item {
  display: flex;
  flex: 1;
  height: 30px;
  padding: 5px;
  justify-content: space-between;
  align-items: center;
  overflow: hidden;

  &.is-finish {
    opacity: .5;

    .undo-icon {
      color: #67C23A;
    }
  }
}

.item-content {
  flex: 1;
  overflow: hidden;
  margin-right: 10px;
}

:deep(.content-tooltip) {
  max-width: 200px;
}

.operates {
  display: flex;
  width: 100px;
  justify-content: space-between;
  height: 100%;
  align-items: center;
  font-size: 20px;

  .iconify {
    cursor: pointer;
  }
}

:deep(.loading) {
  .circular {
    width: 25px;
  }
}
</style>
