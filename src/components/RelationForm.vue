<template>
  <el-dialog
    v-model="visible"
    title="新增关系"
    width="1200px"
    :close-on-click-modal="false"
  >
    <div class="form-container">
      <!-- 左侧表单 -->
      <div class="form-section">
        <el-divider content-position="center">配置</el-divider>
        <!-- 基础信息配置 -->
        <el-form :model="form" label-width="120px">
          <el-row>
            <el-col :span="8">
              <el-form-item label="关系ID">
                <el-input v-model="form.logicId" disabled />
              </el-form-item>
            </el-col>
            <el-col :span="8">
              <el-form-item label="源节点ID">
                <el-input v-model="form.sourceId" disabled />
              </el-form-item>
            </el-col>
            <el-col :span="8">
              <el-form-item label="目标节点ID">
                <el-input v-model="form.targetId" disabled />
              </el-form-item>
            </el-col>
          </el-row>

          <!-- 动作配置区域 -->
          <el-divider content-position="center">动作配置</el-divider>
          <!-- 动作类型选择 -->
          <el-form-item label="动作类型">
            <el-select
              v-model="selectedActionType"
              placeholder="请选择动作类型"
              @change="addAction"
            >
              <el-option
                v-for="item in actionOptions"
                :key="item.type"
                :label="item.label"
                :value="item.type"
              />
            </el-select>
          </el-form-item>

          <!-- 动作标签栏 -->
          <div class="action-tabs">
            <el-tag
              v-for="action in form.actions"
              :key="action.id"
              :type="
                form.currentAction && form.currentAction.id === action.id
                  ? 'primary'
                  : 'info'
              "
              class="action-tag"
              closable
              @click="onActionClick(action)"
              @close="removeAction(action)"
            >
              {{ action.label }}
            </el-tag>
          </div>

          <!-- 动作参数配置 -->
          <el-row :gutter="20">
            <el-col 
              v-for="param in currentActionParams" 
              :key="param.paramKey"
              :span="12"
            >
              <el-form-item :label="param.label">
                <el-input
                  v-if="param.input === 'text'"
                  v-model="form.actionParams[param.paramKey]"
                />
                <el-input-number
                  v-else
                  v-model="form.actionParams[param.paramKey]"
                />
              </el-form-item>
            </el-col>
          </el-row>

          <!-- 验证配置 -->
          <el-divider content-position="center">验证配置</el-divider>
          <el-form-item label="验证类型">
            <el-select v-model="form.verifyType" placeholder="请选择验证类型">
              <el-option
                v-for="item in verifyTypeOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value"
              />
            </el-select>
          </el-form-item>

          <el-row :gutter="20">
            <el-col 
              v-for="param in currentVerifyParams" 
              :key="param.name"
            >
              <el-form-item :label="param.label">
                <el-input v-model="form.verifyParams[param.name]" />
              </el-form-item>
            </el-col>
          </el-row>

          <!-- 回退配置区域 -->
          <el-divider content-position="center">回退配置</el-divider>
          <!-- 回退类型选择 -->
          <el-form-item label="回退类型">
            <el-select
              v-model="selectedRollbackType"
              placeholder="请选择回退类型"
              @change="addRollback"
            >
              <el-option
                v-for="item in actionOptions"
                :key="item.type"
                :label="item.label"
                :value="item.type"
              />
            </el-select>
          </el-form-item>

          <!-- 回退标签栏 -->
          <div class="action-tabs">
            <el-tag
              v-for="rollback in form.rollbacks"
              :key="rollback.id"
              :type="
                form.currentRollback && form.currentRollback.id === rollback.id
                  ? 'primary'
                  : 'info'
              "
              class="action-tag"
              closable
              @click="onRollbackClick(rollback)"
              @close="removeRollback(rollback)"
            >
              {{ rollback.label }}
            </el-tag>
          </div>

          <!-- 回退参数配置 -->
          <el-row :gutter="20">
            <el-col 
              v-for="param in currentRollbackParams" 
              :key="param.paramKey"
              :span="12"
            >
              <el-form-item :label="param.label">
                <el-input
                  v-if="param.input === 'text'"
                  v-model="form.rollbackParams[param.paramKey]"
                />
                <el-input-number
                  v-else
                  v-model="form.rollbackParams[param.paramKey]"
                />
              </el-form-item>
            </el-col>
          </el-row>

          <!-- 其他配置 -->
          <el-divider content-position="center">其他配置</el-divider>
          <el-form-item label="设备ID">
            <el-input v-model="form.deviceId" placeholder="请输入设备ID" />
          </el-form-item>
          <el-form-item label="应用版本ID">
            <el-input
              v-model="form.appVersionId"
              placeholder="请输入应用版本ID"
            />
          </el-form-item>
          <el-form-item label="设备版本ID">
            <el-input
              v-model="form.deviceVersionId"
              placeholder="请输入设备版本ID"
            />
          </el-form-item>
          <el-form-item label="应用ID">
            <el-input v-model="form.appId" placeholder="请输入appId" />
          </el-form-item>
          <el-form-item label="autoAdd">
            <el-input v-model="form.autoAdd" placeholder="请输入autoAdd" />
          </el-form-item>
        </el-form>
      </div>

      <!-- 右侧手机屏幕显示 -->
      <div class="screen-section">
        <el-divider content-position="center">手机屏幕</el-divider>
        <!-- <PhoneScreen /> -->
      </div>
    </div>

    <!-- 对话框底部按钮 -->
    <template #footer>
      <el-button @click="visible = true">取消</el-button>
      <el-button type="primary" @click="submit">保存</el-button>
    </template>
  </el-dialog>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onBeforeUnmount } from "vue";
import { ElMessage } from "element-plus";


// 对话框显示状态
const visible = ref(true);

// 动作类型选择框的值
const selectedActionType = ref("");
// 回退类型选择框的值
const selectedRollbackType = ref("");

const backendData = {
  logicId: "42",
  objId: "29",
  parentId: "12",
  actions:
    '[{"type":"click_point","label":"点击坐标","params":{"x":2,"y":3}},{"type":"click_image","label":"点击图片","params":{"image_path":"4545","match_threshold":2}},{"type":"click_text","label":"点击文字","params":{"text":"4545","match_mode":"45"}},{"type":"input_text","label":"输入文字","params":{"text":"566666"}}]',
  verify: '{"type": "image_match","image_path":"1145"}',
  back: '[{"type": "click_point", "label": "点击坐标", "params": {"x": 0, "y": 0}}, {"type": "swipe", "label": "滑动", "params": {"end_x": 2, "end_y": 2, "start_x": 3, "start_y": 2, "duration": 2}}, {"type": "swipe", "label": "滑动", "params": {"end_x": 0, "end_y": 0, "start_x": 3, "start_y": 2, "duration": 0}}, {"type": "wait", "label": "等待", "params": {"duration": 3}}]',
  deviceId: "1",
  appVersionId: "1",
  deviceVersionId: "1",
  appId: "1",
  autoAdd: 0,
};

// 解析动作与参数
const parsedActions = JSON.parse(backendData.actions);
const actionParams = {};
parsedActions.forEach((action) => {
  const id = Date.now() + Math.random();
  action.id = id;
  for (const [key, value] of Object.entries(action.params)) {
    actionParams[`${id}_${key}`] = value;
  }
  delete action.params;
});

// 解析回退动作与参数
const parsedRollbacks = JSON.parse(backendData.back);
const rollbackParams = {};
parsedRollbacks.forEach((rollback) => {
  const id = Date.now() + Math.random();
  rollback.id = id;
  for (const [key, value] of Object.entries(rollback.params)) {
    rollbackParams[`${id}_${key}`] = value;
  }
  delete rollback.params;
});

// 解析验证动作与参数
const parsedVerify = JSON.parse(backendData.verify);
const verifyParams = reactive({});
// 初始化验证参数
if (parsedVerify) {
  Object.entries(parsedVerify).forEach(([key, value]) => {
    if (key !== 'type') {
      verifyParams[key] = value;
    }
  });
}

// 表单数据
const form = reactive({
  logicId: backendData.logicId,
  sourceId: backendData.objId,
  targetId: backendData.parentId,

  // 动作配置
  actions: parsedActions,
  currentAction: parsedActions[0] || null,
  actionParams,

  // 验证配置
  verifyType: parsedVerify?.type || 'none',
  verifyParams,

  // 回退配置
  rollbacks: parsedRollbacks,
  currentRollback: parsedRollbacks[0] || null,
  rollbackParams,

  // 其他配置
  deviceId: backendData.deviceId,
  appVersionId: backendData.appVersionId,
  deviceVersionId: backendData.deviceVersionId,
  appId: backendData.appId,
  autoAdd: backendData.autoAdd,
});

// 动作类型选项配置
const actionOptions = [
  {
    type: "click_point",
    label: "点击坐标",
    params: [
      { name: "x", label: "X 坐标", input: "number" },
      { name: "y", label: "Y 坐标", input: "number" },
    ],
  },
  {
    type: "click_image",
    label: "点击图片",
    params: [
      { name: "image_path", label: "图片路径", input: "text" },
      { name: "match_threshold", label: "匹配阈值", input: "number" },
    ],
  },
  {
    type: "click_text",
    label: "点击文字",
    params: [
      { name: "text", label: "文字内容", input: "text" },
      { name: "match_mode", label: "匹配模式", input: "text" },
    ],
  },
  {
    type: "swipe",
    label: "滑动",
    params: [
      { name: "start_x", label: "起始X", input: "number" },
      { name: "start_y", label: "起始Y", input: "number" },
      { name: "end_x", label: "结束X", input: "number" },
      { name: "end_y", label: "结束Y", input: "number" },
      { name: "duration", label: "持续时间(ms)", input: "number" },
    ],
  },
  {
    type: "input_text",
    label: "输入文字",
    params: [{ name: "text", label: "输入内容", input: "text" }],
  },
  {
    type: "wait",
    label: "等待",
    params: [{ name: "duration", label: "等待时间(ms)", input: "number" }],
  },
  {
    type: "click_exists",
    label: "存在图片则点击",
    params: [
      { name: "image_path", label: "图片路径", input: "text" },
      { name: "match_threshold", label: "匹配阈值", input: "number" },
    ],
  },
];

// 计算当前选中动作的参数配置
const currentActionParams = computed(() => {
  if (!form.currentAction) return [];
  const action = actionOptions.find((a) => a.type === form.currentAction.type);
  if (!action) return [];

  const params = [];
  action.params.forEach((p) => {
    const paramKey = `${form.currentAction.id}_${p.name}`;
    if (!(paramKey in form.actionParams)) {
      form.actionParams[paramKey] = p.input === "number" ? 0 : "";
    }
    params.push({
      ...p,
      paramKey,
    });
  });
  return params;
});

// 验证类型选项配置
const verifyTypeOptions = [
  {
    value: "none",
    label: "无验证",
    params: [],
  },
  {
    value: "text_exists",
    label: "文本存在",
    params: [{ name: "text", label: "验证文字", input: "text" }],
  },
  {
    value: "image_match",
    label: "截图比对",
    params: [{ name: "image_path", label: "图片路径", input: "text" }],
  },
  {
    value: "fail_if_text_exists",
    label: "存在文字失败",
    params: [{ name: "text", label: "验证文字", input: "text" }],
  },
  {
    value: "fail_if_image_exists",
    label: "存在图片失败",
    params: [{ name: "image_path", label: "图片路径", input: "text" }],
  },
  {
    value: "fail_if_text_missing",
    label: "不存在文字失败",
    params: [{ name: "text", label: "验证文字", input: "text" }],
  },
  {
    value: "fail_if_image_missing",
    label: "不存在图片失败",
    params: [{ name: "image_path", label: "图片路径", input: "text" }],
  },
];

const currentVerifyParams = computed(() => {
  const selected = verifyTypeOptions.find((v) => v.value === form.verifyType);
  if (!selected) return [];
  // 初始化字段
  selected.params?.forEach((param) => {
    if (!(param.name in verifyParams)) verifyParams[param.name] = "";
  });
  return selected.params || [];
});

// 计算当前选中回退动作的参数配置
const currentRollbackParams = computed(() => {
  if (!form.currentRollback) return [];
  const action = actionOptions.find(
    (a) => a.type === form.currentRollback.type
  );
  if (!action) return [];

  const params = [];
  action.params.forEach((p) => {
    const paramKey = `${form.currentRollback.id}_${p.name}`;
    if (!(paramKey in form.rollbackParams)) {
      form.rollbackParams[paramKey] = p.input === "number" ? 0 : "";
    }
    params.push({
      ...p,
      paramKey,
    });
  });
  return params;
});

// 添加新动作
const addAction = (type) => {
  const id = Date.now() + Math.random(); // 生成唯一ID
  const newAction = {
    id,
    type,
    label: actionOptions.find((a) => a.type === type).label,
  };
  form.actions.push(newAction);
  form.currentAction = newAction;
  selectedActionType.value = ""; // 清空选择框的值
};

// 删除动作
const removeAction = (action) => {
  const index = form.actions.findIndex((a) => a.id === action.id);
  if (index !== -1) {
    form.actions.splice(index, 1);
    if (form.currentAction && form.currentAction.id === action.id) {
      form.currentAction = form.actions[0] || null;
    }
  }
};

// 切换当前动作
const onActionClick = (action) => {
  form.currentAction = action;
};

// 添加新回退动作
const addRollback = (type) => {
  const id = Date.now() + Math.random(); // 生成唯一ID
  const newRollback = {
    id,
    type,
    label: actionOptions.find((a) => a.type === type).label,
  };
  form.rollbacks.push(newRollback);
  form.currentRollback = newRollback;
  selectedRollbackType.value = ""; // 清空选择框的值
};

// 删除回退动作
const removeRollback = (rollback) => {
  const index = form.rollbacks.findIndex((r) => r.id === rollback.id);
  if (index !== -1) {
    form.rollbacks.splice(index, 1);
    if (form.currentRollback && form.currentRollback.id === rollback.id) {
      form.currentRollback = form.rollbacks[0] || null;
    }
  }
};

// 切换当前回退动作
const onRollbackClick = (rollback) => {
  form.currentRollback = rollback;
};

// 提交表单
const submit = () => {
  // 构建符合后端数据库结构的数据
  const formattedData = {
    // 基础信息
    logicId: form.logicId,
    objId: form.sourceId, // 源节点ID作为objId
    parentId: form.targetId, // 目标节点ID作为parentId

    // 动作配置
    actions: form.actions.map((action) => ({
      type: action.type,
      label: action.label,
      params: actionOptions
        .find((a) => a.type === action.type)
        ?.params.reduce((acc, param) => {
          const paramKey = `${action.id}_${param.name}`;
          acc[param.name] = form.actionParams[paramKey];
          return acc;
        }, {}),
    })),

    // 回退配置
    back: form.rollbacks.map((rollback) => ({
      type: rollback.type,
      label: rollback.label,
      params: actionOptions
        .find((a) => a.type === rollback.type)
        ?.params.reduce((acc, param) => {
          const paramKey = `${rollback.id}_${param.name}`;
          acc[param.name] = form.rollbackParams[paramKey];
          return acc;
        }, {}),
    })),

     // 验证配置
     verify: {
      type: form.verifyType,
      ...verifyTypeOptions
        .find((v) => v.value === form.verifyType)
        ?.params.reduce((acc, param) => {
          acc[param.name] = verifyParams[param.name];
          return acc;
        }, {}),
    },

    // 其他配置
    deviceId: form.deviceId,
    appVersionId: form.appVersionId,
    deviceVersionId: form.deviceVersionId,
    appId: form.appId,
    autoAdd: form.autoAdd,
  };

  // 输出格式化后的数据
  console.log("保存的表单数据：", JSON.stringify(formattedData, null, 2));

  // 关闭对话框
  visible.value = false;
};
</script>

<style scoped>
.form-container {
  display: flex;
  gap: 20px;
}

.form-section {
  flex: 1;
  overflow-y: auto;
  max-height: 70vh;
  padding: 0 20px;
}

.screen-section {
  flex: 1;
  display: flex;
  flex-direction: column;
  border-left: 1px solid #eee;
  padding-left: 20px;
}

.el-form-item {
  margin-bottom: 16px;
}

.action-tabs {
  margin-bottom: 16px;
}

.action-tag {
  margin-right: 8px;
  cursor: pointer;
}

.device-screen {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 400px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-top: 20px;
  padding: 10px;
}

.screen-display {
  position: relative;
  width: 100%;
  height: 400px;
  background-color: #000;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 10px;
}

.screen-display img,
.screen-display video {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  display: block;
  margin: 0 auto;
}

.screen-controls {
  margin-top: 10px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  width: 100%;
}

.connection-controls {
  display: flex;
  flex-direction: column;
  width: 100%;
  gap: 10px;
}

.mb-2 {
  margin-bottom: 10px;
}

/* 添加新的样式 */
:deep(.el-form-item__content) {
  width: 100%;
}

:deep(.el-input-number) {
  width: 100%;
}

.el-row {
  margin-bottom: 8px;
}
</style>
