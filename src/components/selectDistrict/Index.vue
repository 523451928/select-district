<template>
  <div class="select-region">
    <el-dialog title="选择配送区域" destroy-on-close :visible="dialogVisible" :close-on-click-modal="false" @close="closeModal">
      <ul class="region-list">
        <li v-for="(item, index) in regionData" v-show="!item.hide" :key="index" class="region-item">
          <el-checkbox v-model="item.checked" :indeterminate="item.isIndeterminate" class="region-checkbox" @change="handleCheckPart(item)">{{ item.part }}</el-checkbox>
          <div class="provinces">
            <el-checkbox
              v-for="province in item.provinces"
              v-show="!province.hide"
              :key="province.regionId"
              v-model="province.checked"
              :value="province.regionId"
              :indeterminate="province.isIndeterminate"
              @change="handleCheckProvince(province)"
              @mouseenter.native="showCities(province, $event)"
              @mouseleave.native="hideCities(province, $event)"
            >
              {{ province.name }}
              <span v-show="province.selectCount" class="count" style="color: #FF8A00;">
                ({{ province.selectCount }})
              </span>
              <i class="el-icon-arrow-down" />
            </el-checkbox>
          </div>
        </li>
      </ul>
      <div class="dialog-footer">
        <el-checkbox v-model="checkAllRegion" :indeterminate="isIndeterminate">全选</el-checkbox>
        <div>
          <span>已选择{{ totalSelectCount }}个区域</span>
          <el-button type="primary" class="test" @click="confirm">确 定</el-button>
        </div>
      </div>
    </el-dialog>
    <select-city ref="selectCity" :visible="visible" :can-select-county="false" :current-province="currentProvince" />
  </div>
</template>

<script>
import { data as regionData } from './districtData'
import popperMixin from './vue-popper'
import SelectCity from './SelectCity.vue'

export default {
  components: {
    SelectCity
  },
  mixins: [popperMixin],
  props: {
    // 是否双向绑定
    binding: {
      type: Boolean,
      default: false
    },
    // 双向绑定的值
    value: {
      type: Array,
      default: () => []
    },
    // 排除已选择的地区
    excludeRegion: {
      type: Array,
      default: () => []
    },
    visible: {
      type: Boolean,
      default() {
        return true
      }
    }
  },
  data() {
    return {
      regionData: JSON.parse(JSON.stringify(regionData)),
      dialogVisible: this.visible,
      currentProvince: {
        cities: []
      }
    }
  },
  computed: {
    selectedRegions() {
      const res = []
      this.regionData.forEach((item) => {
        item.provinces.forEach((province) => {
          if (province.checked && !province.hide && !province.cities.some(city => city.hide)) res.push(province.regionId)
          province.cities.forEach((city) => {
            if (city.checked && !province.checked && !city.hide ||
            province.checked && city.checked && !city.hide && province.cities.some(city => city.hide) && city.counties.filter(county => !county.hide).length === city.counties.length) res.push(city.regionId)
            if (city.counties) {
              city.counties.forEach((county) => {
                if (county.checked && !province.checked && !city.checked && !county.hide ||
                city.checked && !county.hide && city.counties.some(county => county.hide)) res.push(county.regionId)
              })
            }
          })
        })
      })
      return res
    },
    totalSelectCount() {
      return this.regionData.reduce((acc, item) => {
        acc += item.selectCount ? item.selectCount : 0
        return acc
      }, 0)
    },
    isIndeterminate() {
      const { length } = this.regionData.filter(item => item.checked && !item.hide)
      return length > 0 && length < this.regionData.length || this.regionData.some(item => item.isIndeterminate)
    },
    checkAllRegion: {
      get() {
        return this.regionData.every(item => item.checked)
      },
      set(val) {
        this.regionData.forEach((item) => {
          item.checked = val
          item.provinces.forEach((province) => {
            this.$set(province, 'checked', item.checked)
            province.cities.forEach((city) => {
              this.$set(city, 'checked', item.checked)
              if (city.counties) {
                city.counties.forEach((county) => {
                  this.$set(county, 'checked', item.checked)
                })
              }
            })
          })
        })
      }
    }
  },
  watch: {
    visible(newVal) {
      this.dialogVisible = newVal
      this.regionData = JSON.parse(JSON.stringify(regionData))
      newVal && this.fillBack()
    },
    regionData: {
      handler(newVal) {
        newVal.forEach((area) => {
          const provinces = area.provinces.filter(item => !item.hide)
          const { length } = provinces.filter(item => item.checked)
          this.$set(area, 'isIndeterminate', length > 0 && length < provinces.length || provinces.some(item => item.isIndeterminate))
          this.$set(area, 'checked', provinces.every(item => item.checked))
          this.$set(area, 'selectCount', provinces.reduce((acc, next) => {
            acc += next.selectCount && !next.hide ? next.selectCount : 0
            return acc
          }, 0))
          provinces.forEach((province) => {
            this.$set(province, 'selectCount', province.cities.reduce((acc, item) => {
              acc += !item.hide ? item.counties.filter(item => item.checked && !item.hide).length : 0
              return acc
            }, 0))
            const cities = province.cities.filter(item => !item.hide)
            const { length } = cities.filter(item => item.checked)
            this.$set(province, 'isIndeterminate', length > 0 && length < cities.length || cities.some(item => item.isIndeterminate))
            this.$set(province, 'checked', cities.every(item => item.checked))
            province.cities.forEach((city) => {
              const counties = city.counties.filter(item => !item.hide)
              const { length } = counties.filter(item => item.checked)
              this.$set(city, 'isIndeterminate', length > 0 && length < counties.length)
              this.$set(city, 'checked', counties.every(item => item.checked))
            })
          })
        })
        // 双向绑定
        if (this.binding) {
          this.$emit('input', this.selectedRegions)
        }
      },
      deep: true
    }
  },
  mounted() {
    this.fillBack()
  },
  methods: {
    findNameById(id) {
      const regions = this.regionData.reduce((acc, item) => {
        return acc.concat(item.provinces)
      }, [])
      let stacks = [...regions]
      do {
        const current = stacks.shift()
        if (current.cities) {
          stacks = stacks.concat(current.cities.map(item => {
            const res = {
              ...item,
              path: current.path ? [...current.path] : [current]
            }
            res.path.push(item)
            return res
          }))
        }
        if (current.counties) {
          stacks = stacks.concat(current.counties.map(item => {
            const res = {
              ...item,
              path: current.path ? [...current.path] : [current]
            }
            res.path.push(item)
            return res
          }))
        }
        if (current.regionId === id) return current
      } while (stacks.length)
    },
    fillBack() {
      this.regionData.forEach((item) => {
        let partChecked = true
        let partHide = true
        item.provinces.forEach((province) => {
          let provinceChecked = false
          if (this.value.find(val => province.regionId === val)) {
            provinceChecked = true
            this.$set(province, 'checked', true)
            this.$set(item, 'isIndeterminate', true)
          } else {
            partChecked = false
          }
          this.$set(item, 'checked', partChecked)
          if (this.excludeRegion.find(val => province.regionId === val)) {
            this.$set(province, 'hide', true)
          } else {
            partHide = false
          }
          this.$set(item, 'hide', partHide)
          province.cities.forEach((city) => {
            let cityChecked = false
            if (this.value.find(val => city.regionId === val) || provinceChecked) {
              cityChecked = true
              this.$set(city, 'checked', true)
            }
            if (this.value.find(val => city.regionId === val) && !provinceChecked) {
              this.$set(province, 'isIndeterminate', true)
            }
            if (this.excludeRegion.find(val => city.regionId === val)) {
              this.$set(city, 'hide', true)
            }
            if (province.cities.every(city => city.hide)) {
              this.$set(province, 'hide', true)
            }
            if (city.counties) {
              city.counties.forEach((county) => {
                if (this.value.find(val => county.regionId === val) || provinceChecked || cityChecked) {
                  this.$set(county, 'checked', true)
                }
                if (this.value.find(val => county.regionId === val) && !cityChecked) {
                  this.$set(city, 'isIndeterminate', true)
                }
                if (this.excludeRegion.find(val => county.regionId === val)) {
                  this.$set(county, 'hide', true)
                }
                if (city.counties.every(county => county.hide)) {
                  this.$set(city, 'hide', true)
                }
              })
            }
          })
        })
      })
    },
    confirm() {
      this.$emit('confirm', this.selectedRegions)
      this.$emit('closeModal')
    },
    closeModal() {
      this.$emit('closeModal')
    },
    handleCheckProvince(item) {
      item.cities.forEach((city) => {
        this.$set(city, 'checked', item.checked)
        if (city.counties) {
          city.counties.forEach((county) => {
            this.$set(county, 'checked', item.checked)
          })
        }
      })
    },
    handleCheckPart(item) {
      item.provinces.forEach((province) => {
        this.$set(province, 'checked', item.checked)
        province.cities.forEach((city) => {
          this.$set(city, 'checked', item.checked)
          if (city.counties) {
            city.counties.forEach((county) => {
              this.$set(county, 'checked', item.checked)
            })
          }
        })
      })
    },
    showCities(province, event) {
      this.currentProvince = province
      this.referenceElm = event.target
      this.$refs.selectCity.show()
      this.popperElm = this.$refs.selectCity.$refs.popupCities
      this.placement = 'bottom'
      this.createPopper()
    },
    hideCities() {
      if (!this.$refs.selectCity.isEnter) {
        this.$refs.selectCity.cityLeave()
      }
    }
  }
}
</script>

<style lang="scss">
.region-list {
  margin: 0;
  padding: 0;
  .region-item {
    list-style: none;
    display: flex;
    margin-bottom: 20px;
    .region-checkbox {
      width: 50px;
      margin-right: 10px;
      font-weight: bold;
    }
    .provinces {
      margin-left: 40px;
    }
    .el-checkbox {
      margin-bottom: 10px;
    }
    .el-dropdown {
      margin-right: 18px;
    }
  }
}
.select-region {
  .dialog-footer {
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
}
</style>
