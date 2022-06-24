<template>
  <div class="filter-box-container">
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
          <el-icon size="30" color="red" v-if="data.conditionArray.length > 1" @click="deleteCondition(index)"
                   style="cursor: pointer;margin-left: 10px;">
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
        <el-button type="primary" :icon="CirclePlus" :disabled="data.conditionArray.length >= fields.length"
                   size="small" @click="addCondition()">
        </el-button>
        <el-button type="primary" :icon="CirclePlusFilled" size="small" @click="addCombinationConditions()"
                   style="margin-top: 10px">
        </el-button>
      </div>
    </div>
  </div>
  <el-button type="primary" @click="createSql(data)">生成</el-button>
  <el-button type="primary" @click="executeSql">执行SQL</el-button>
  <el-input type="textarea" v-model="sqlData"></el-input>
  <div class="table-contain" style="margin-top: 50px;width: 900px">
    <el-table :data="tableData" style="width: 100%" border>
      <el-table-column v-for="item in fields" :prop="item.value" :label="item.label" width="180" />
    </el-table>
  </div>
</template>

<script setup>
import {onMounted, ref} from "vue";
import {
  CirclePlus,
  CirclePlusFilled,
  CloseBold
} from '@element-plus/icons-vue'
import FilterBoxSon from "./FilterBoxSon.vue";
import axios from "axios";
import tableFieldsJson from "../json/tableFields.json"

// 动态获取表字段
let fields = ref([]);

// 判断条件写死
let conditions = ref([
  {"label": "等于", "value": "="},
  {"label": "不等于", "value": "!="},
  {"label": "大于", "value": ">"},
  {"label": "小于", "value": "<"},
  {"label": "大于等于", "value": ">="},
  {"label": "小于等于", "value": "<="},
  {"label": "正则判断", "value": "regexp"}
]);

let sqlData = ref('');

let tableName = ref('');

let tableData = ref([]);

/**
 * 此处存放条件判断
 * 数据结构为：
 * {
 *   连接符: and，or
 *   一级条件: [
 *     {
 *       字段名: '',
 *       条件: '',
 *       数值: ''
 *     }
 *   ],
 *   二级条件: [
 *      {同一级条件且嵌套二级条件}
 *   ]
 * }
 */
let data = ref(
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
    });

const changeLink = () => {
  data.value.link = data.value.link === 'and' ? 'or' : 'and';
};

// 执行sql
const executeSql = ()=>{
  axios.post('http://localhost:8081/sql/execute',{
    sql: sqlData.value
  }).then(res=>{
    tableData.value = res.data.data;
  })
};

const addCondition = () => {
  data.value.conditionArray.push({
    columnName: '',
    condition: '',
    val: ''
  })

};

/**
 * 删除一级条件
 * @param index 删除的索引
 */
const deleteCondition = (index) => {
  data.value.conditionArray.splice(index, 1);
};

/**
 * 删除二级条件
 * @param index 删除的索引
 */
const deleteCombinationConditionHandler = (index) => {
  data.value.combinationConditionArray.splice(index, 1);
}

/**
 * 增加二级条件,直接push结构进去
 */
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

// 构建第一层的sql
const createSql = (data) => {
  // 拼接sql
  let sql = 'select * from ' + tableName.value + ' where ';
  for (let item of data.conditionArray) {
    if (item.columnName && item.condition && item.val) {
      sql += `${item.columnName} ${item.condition} '${item.val}' ${data.link} `;
    }
  }
  for (let item of data.combinationConditionArray) {
    if (item.conditionArray.length > 0) {
      sql += `(${createSecondSql(item)}) ${item.link} `;
    }
  }
  // 去掉最后一个and或者or
  sql = sql.substring(0, sql.length - 4);
  sql += ';';
  sqlData.value = sql;
}

// 构建二级条件里面的sql，利用递归
const createSecondSql = (data) => {
  let sql = '';
  for (let item of data.conditionArray) {
    if (item.columnName && item.condition && item.val) {
      sql += `${item.columnName} ${item.condition} '${item.val}' ${data.link} `;
    }
  }
  for (let item of data.combinationConditionArray) {
    if (item.conditionArray.length > 0) {
      sql += `(${createSecondSql(item)}) ${item.link} `;
    }
  }
  sql = sql.substring(0, sql.length - 4);
  return sql;
}

onMounted((() => {
  // 动态获取所有列名
  axios.get('http://localhost:8081/sql/getColumns').then(res => {
    let data = res.data.data;
    for (let dataItem of data) {
      fields.value.push({
        label: dataItem.columnComment,
        value: dataItem.columnName
      })
    }
    tableName.value = res.data.tableName
  })

  // 直接本地获取,读取json文件
  // let dataJson = tableFieldsJson;
  //   for (let dataItem of dataJson.data) {
  //     fields.value.push({
  //       label: dataItem.columnComment,
  //       value: dataItem.columnName
  //     })
  //   }
  //   tableName.value = dataJson.tableName
}))


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

.filter-box-container {
  display: flex;
  justify-content: center;
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
  flex-direction: row;
}

/deep/ .el-button + .el-button {
  margin-left: 0;
}


</style>