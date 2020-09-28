<template>
  <div>
    <el-button style="margin-bottom: 20px;" @click="selectVisible = true">指定不配送区域</el-button>
    <div>
      已选择： {{ formatRegionName(currentRegion) }}
    </div>
    <SelectDistrict
      ref="selectDistrict"
      v-model="currentRegion"
      :visible="selectVisible"
      @closeModal="selectVisible = false"
      @confirm="confirmSelect"
    />
  </div>
</template>

<script>
import SelectDistrict from '../selectDistrict/Index.vue'
export default {
  components: {
    SelectDistrict
  },
  props: {
    excludeRegion: {
      type: Array,
      default() {
        return []
      }
    }
  },
  data() {
    return {
      selectVisible: false,
      currentRegion: [...this.excludeRegion]
    }
  },
  methods: {
    formatRegionName(arr = []) {
      return arr.reduce((acc, item) => {
        acc.push(this.$refs.selectDistrict.findNameById(item).name)
        return acc
      }, []).join(',')
    },
    confirmSelect(region) {
      this.currentRegion = region
    }
  }
}
</script>

<style>

</style>
