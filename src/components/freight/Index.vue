<template>
  <div class="create-template">
    <el-form ref="ruleForm" :model="ruleForm" :rules="rules" label-width="150px" class="demo-ruleForm">
      <el-form-item label="模板名称" prop="name">
        <el-input v-model="ruleForm.name" />
      </el-form-item>
      <el-form-item label="计费方式" prop="type">
        <el-select v-model="ruleForm.type" placeholder="请选择">
          <el-option
            v-for="item in options"
            :key="item.value"
            :label="item.label"
            :value="item.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="默认配送区域">
        <span style="color: #FF623F;">除可配送区域和不支持配送区域外，其余地区的运费采用默认“默认运费”</span>
        <div class="default-freight">
          <el-form :inline="true" :model="defaultFreightForm" class="demo-form-inline">
            <el-form-item label="默认运费">
              <el-input v-model="defaultFreightForm.user" placeholder="默认运费" />
              件内，
            </el-form-item>
            <el-form-item label="">
              <el-input v-model="defaultFreightForm.user" placeholder="" />元，
            </el-form-item>
            <el-form-item label="">
              每增加<el-input v-model="defaultFreightForm.user" placeholder="" />件，
            </el-form-item>
            <el-form-item label="">
              运费增加<el-input v-model="defaultFreightForm.user" placeholder="" />元
            </el-form-item>
          </el-form>
        </div>
      </el-form-item>

      <el-form-item label="指定区域运费配置：">
        <el-button style="margin-bottom: 20px;" @click="addIncludeRegion">增加指定区域运费配置</el-button>
        <el-table
          :data="includeRegion"
          border
          style="width: 100%"
        >
          <el-table-column
            prop="region"
            label="指定配送区域"
            width="480"
          >
            <template v-slot="{row, $index}">
              {{ formatRegionName(row.region) }}
              <el-button type="text" @click="modifyRegion(row, $index)">修改</el-button>
              <el-button type="text" @click="deleteRegion($index)">删除</el-button>
            </template>
          </el-table-column>
          <el-table-column
            prop="first"
            label="首件（件）"
          >
            <template v-slot="{$index}">

              <el-input-number
                v-model="includeRegion[$index].freight"
                :min="0"
                :controls="false"
                size="small"
              />
            </template>
          </el-table-column>
          <el-table-column
            prop="firstFreight"
            label="首费（元）"
          >
            <template v-slot="{$index}">
              <el-input-number
                v-model="includeRegion[$index].freight"
                :min="0"
                :controls="false"
                size="small"
              />
            </template>
          </el-table-column>
          <el-table-column
            prop="again"
            label="续件（件））"
          >
            <template v-slot="{$index}">
              <el-input-number
                v-model="includeRegion[$index].freight"
                :min="0"
                :controls="false"
                size="small"
              />
            </template>
          </el-table-column>
          <el-table-column
            prop="againFreight"
            label="续费（元）"
          >
            <template v-slot="{$index}">
              <el-input-number
                v-model="includeRegion[$index].freight"
                :min="0"
                :controls="false"
                size="small"
              />
            </template>
          </el-table-column>
        </el-table>
      </el-form-item>
      <el-form-item label="指定区域不配送：">
        <NoDeliveryDistrict ref="noDeliveryDistrict" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="submitForm('ruleForm')">立即创建</el-button>
        <el-button @click="resetForm('ruleForm')">重置</el-button>
      </el-form-item>
    </el-form>
    <SelectDistrict
      ref="selectDistrict"
      :exclude-region="excludeRegion"
      :visible="selectVisible"
      :value="currentRegion"
      @closeModal="selectVisible = false"
      @confirm="confirmSelect"
    />
  </div>

</template>
<script>
import SelectDistrict from '../selectDistrict/Index.vue'
import NoDeliveryDistrict from './NoDeliveryDistrict.vue'

export default {
  name: 'CreateFreightTemplate',
  components: {
    SelectDistrict,
    NoDeliveryDistrict
  },
  data() {
    return {
      ruleForm: {
        name: '',
        type: 0
      },
      selectVisible: false,
      currentRegion: [],
      includeRegion: [],
      editIndex: -1,
      defaultFreightForm: {},
      options: [
        {
          value: 0,
          label: '按件计费'
        }
      ],
      rules: {
        name: [
          { required: true, message: '请输入模板名称', trigger: 'blur' },
          { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        type: [
          { required: true, message: '请输入活动名称', trigger: 'blur' }
        ]
      }
    }
  },
  computed: {
    excludeRegion() {
      return this.includeRegion.reduce((acc, item) => {
        acc = acc.concat(item.region)
        return acc
      }, []).filter(item => !this.currentRegion.includes(item))
    }
  },
  methods: {
    modifyRegion({ region }, index) {
      this.currentRegion = region
      this.selectVisible = true
      this.editIndex = index
    },
    deleteRegion(index) {
      this.includeRegion.splice(index, 1)
    },
    formatRegionName(arr = []) {
      return arr.reduce((acc, item) => {
        acc.push(this.$refs.selectDistrict.findNameById(item).name)
        return acc
      }, []).join(',')
    },
    addIncludeRegion() {
      this.editIndex = -1
      this.currentRegion = []
      this.selectVisible = true
    },
    confirmSelect(region) {
      if (this.editIndex === -1) {
        this.includeRegion.push({
          region
        })
      } else {
        this.includeRegion[this.editIndex].region = region
      }
      this.selectVisible = false
    },
    submitForm(formName) {
      this.$refs[formName].validate((valid) => {
        if (valid) {
          alert('submit!')
        } else {
          console.log('error submit!!')
          return false
        }
      })
    },
    resetForm(formName) {
      this.$refs[formName].resetFields()
    }
  }
}
</script>

<style lang="scss">
.create-template {
  padding: 10px;
  .el-input {
    max-width: 200px;
  }
  .default-freight {
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 8px;
    .el-input {
      max-width: 100px;
    }
  }
}
</style>
