<template>
  <div class="filter-box-container-son">
    <div class="filter-box-item">
      <div class="link-contain">
        <!--      连接符-->
        <el-button type="primary" text @click="changeLink()">
          {{ data.link }}
        </el-button>
      </div>
      <div class="filter-contain">
        <!--      字段名，条件，数值-->
        <div class="filter-contain-item" v-for="(item,index) in data.conditionArray">
          <el-select v-model="item.columnName" placeholder="请选择字段" style="width: 150px;">
            <el-option v-for="(fieldItem,index) in fields" :disabled="fieldItem.disabled" :key="index"
                       :label="fieldItem.label"
                       :value="fieldItem.value"></el-option>
          </el-select>
          <el-select v-model="item.condition" placeholder="请选择条件" style="width: 150px; margin-left: 20px">
            <el-option v-for="(item,index) in conditions" :key="index" :label="item.label"
                       :value="item.value"></el-option>
          </el-select>
          <el-input v-model="item.val" placeholder="请输入数值"
                    style="width: 150px;height: 32px;margin-left: 20px"></el-input>
          <el-icon size="30" color="red" v-if="data.conditionArray.length > 1" @click="deleteCondition(index)" style="cursor: pointer;margin-left: 10px;">
            <CloseBold/>
          </el-icon>
        </div>
        <div>
          <filter-box-son v-if="data.combinationConditionArray.length > 0"
                          v-for="(item,index) in data.combinationConditionArray"
                          :index="index"
                          :fields="fields"
                          @deleteCombinationConditionEmit="deleteCombinationConditionHandler"
                          :parent-data="item"></filter-box-son>
        </div>
      </div>
      <div class="toolbar">
        <!--      增加工具栏-->
        <el-button type="primary" :icon="CirclePlus" size="small" @click="addCondition()">
        </el-button>
        <el-button type="primary" :icon="CirclePlusFilled" size="small" @click="addCombinationConditions()"
                   style="margin-top: 10px">
        </el-button>
        <el-button type="primary" :icon="CloseBold" size="small" @click="deleteCombinationConditionClick()"
                   style="margin-top: 10px">
        </el-button>
      </div>
    </div>
  </div>
</template>

<script setup>
import {ref} from "vue";
import {
  CirclePlus,
  CirclePlusFilled,
  CloseBold
} from '@element-plus/icons-vue'
import FilterBoxSon from "./FilterBoxSon.vue";

const props = defineProps({
  parentData: {
    type: Object
  },
  index: {
    type: Number
  },
  fields: {
    type: Array
  }
})

const emits = defineEmits([
  "deleteCombinationConditionEmit"
])

let parentData = ref(props.parentData);

let conditions = ref([
  {"label": "等于", "value": "="},
  {"label": "不等于", "value": "!="},
  {"label": "大于", "value": ">"},
  {"label": "小于", "value": "<"},
  {"label": "大于等于", "value": ">="},
  {"label": "小于等于", "value": "<="},
  {"label": "正则判断", "value": "regexp"},
]);

let data = ref(parentData);

const changeLink = () => {
  data.value.link = data.value.link === 'and' ? 'or' : 'and';
};

const addCondition = () => {
  // 添加一个空的条件
  data.value.conditionArray.push({
    columnName: '',
    condition: '',
    val: ''
  })

};

const deleteCondition = (index) => {
  data.value.conditionArray.splice(index, 1);
};

const deleteCombinationConditionHandler = (index) => {
  data.value.combinationConditionArray.splice(index, 1);
}

const deleteCombinationConditionClick = () => {
  emits("deleteCombinationConditionEmit", props.index)
};

const addCombinationConditions = () => {
  data.value.combinationConditionArray.push(
      {
        link: 'and',
        conditionArray: [
          {
            columnName: '',
            condition: '',
            val: ''
          }
        ],
        combinationConditionArray: []
      }
  )
};


</script>

<style scoped>

.link-contain {
  display: flex;
  justify-content: center;
  align-items: center;
  border: 1px solid #ccc;
  padding-left: 1px;
  padding-right: 1px;
  width: 50px;
}

.filter-box-container-son {
  display: flex;
  width: auto;
}

.toolbar {
  display: flex;
  flex-direction: column;
  border: 1px solid #ccc;
  padding-left: 10px;
  padding-right: 10px;
}

.filter-contain {
  display: flex;
  margin-left: 15px;
  margin-right: 15px;
  flex-direction: column;
}

.filter-box-item {
  border: 1px solid #ccc;
  display: flex;
  justify-content: center;
  flex-direction: row;
}

/deep/ .el-button + .el-button {
  margin-left: 0;
}


</style>