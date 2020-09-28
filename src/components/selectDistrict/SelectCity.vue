<template>
  <div>
    <div v-show="popupCitiesVisible" ref="popupCities" class="popup-cities popup" @mouseleave="cityLeave" @mouseenter="cityEnter">
      <div v-show="popupCitiesVisible" class="list">
        <div
          v-for="city in currentProvince.cities"
          v-show="!city.hide"
          :key="city.regionId"
          class="city-item"
          @mouseenter="showCounties(city, $event)"
        >
          <el-checkbox
            v-model="city.checked"
            :value="city.regionId"
            :indeterminate="city.isIndeterminate"
            @change="handleCheckCity(city)"
          >
            {{ city.name }}
            <span v-if="city.counties.filter(item => item.checked && !item.hide).length" class="count" style="color: #FF8A00;">({{ city.counties.filter(item => item.checked && !item.hide).length }})</span>
            <i class="el-icon-arrow-right" />
          </el-checkbox>
        </div>
      </div>
    </div>
    <div v-show="popupCountyVisible" ref="popupCounties" class="popup-counties popup" @mouseenter="countyEnter" @mouseleave="popupCountyVisible = false">
      <div v-show="popupCountyVisible" class="list">
        <div
          v-for="county in currentCity.counties"
          v-show="!county.hide"
          :key="county.regionId"
          class="city-item"
        >
          <el-checkbox
            :key="county.regionId"
            v-model="county.checked"
            :value="county.regionId"
          >
            {{ county.name }}
          </el-checkbox>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import popperMixin from './vue-popper'
export default {
  mixins: [popperMixin],
  props: {
    currentProvince: {
      type: Object,
      default() {
        return {
          cities: []
        }
      }
    },
    visible: {
      type: Boolean,
      default: true
    },
    canSelectCounty: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      popupCitiesVisible: false,
      popupCountyVisible: false,
      isEnter: false,
      currentCity: {
        counties: []
      }
    }
  },
  watch: {
    visible(newVal) {
      if (!newVal) {
        this.popupCitiesVisible = false
        this.popupCountyVisible = false
      }
    },
    popupCitiesVisible(newVal) {
      if (!newVal) this.popupCountyVisible = false
    }
  },
  methods: {
    handleCheckCity(city) {
      city.counties.forEach((county) => {
        this.$set(county, 'checked', city.checked)
      })
    },
    show() {
      this.popupCitiesVisible = true
    },
    countyEnter() {
      this.popupCitiesVisible = true
      this.popupCountyVisible = true
    },
    cityEnter() {
      this.popupCitiesVisible = true
      this.isEnter = true
    },
    cityLeave() {
      this.popupCitiesVisible = false
      this.isEnter = false
      this.popupCountyVisible = false
    },
    showCounties(city, event) {
      this.currentCity = city
      this.referenceElm = event.target
      this.popupCountyVisible = true
      this.popperElm = this.$refs.popupCounties
      this.placement = 'right'
      this.createPopper()
    }
  }
}
</script>

<style lang="scss">
.popup-cities, .popup-counties {
  // display: flex;
  // flex-direction: column;
  background: #fff;
  z-index: 10000;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);
  border: 1px solid #ebeef5;
  .list {
    max-height: 300px;
    overflow: auto;
    .city-item {
      padding: 8px 20px;
      &:hover {
        background: #f5f5f5;
      }
      .el-checkbox {
        display: block;
      }
    }
  }
}
.popper__arrow {
  position: absolute;
  display: block;
  width: 0;
  height: 0;
  border-color: transparent;
  border-style: solid;
  border-width: 6px;
  margin-right: 3px;
  filter: drop-shadow(0 2px 12px rgba(0,0,0,.03));
}
.popup[x-placement^=bottom] {
  .popper__arrow {
    border-top-width: 0;
    border-bottom-color: #fff;
    top: -6px;
  }
}
.popup[x-placement^=left] {
  .popper__arrow {
    border-right-width: 0;
    border-left-color: #fff;
    right: -6px;
  }
}
.popup[x-placement^=right] {
  .popper__arrow {
    border-left-width: 0;
    border-right-color: #fff;
    left: -6px;
  }
}
.popup[x-placement^=top] {
  .popper__arrow {
    border-bottom-width: 0;
    border-top-color: #fff;
    bottom: -6px;
  }
}
</style>
