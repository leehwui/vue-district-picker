<template>
  <div class="picker-wrapper">
    <div class="picker-lg">
      <div class="picker-control" @click="showDropdown">
        <span>{{ selectedDistrictStr ? selectedDistrictStr : 'Please Select' }}</span>
      </div>
      <div class="area-dropdown" v-if="showOptions">
        <div class="city-list">
          <div class="city-item"  v-for="(item, index) in options"
                        v-bind:key="index"
                        v-bind:class="{active: item.id === selectedCity.id }"
                        @click="setSelectedCity(index)">
            {{ item.label }}
          </div>
        </div>
        <div class="district-list">
          <div class="district-item" v-for="(item, index) in
                                     selectedCity.children"
                                     v-bind:key="index" 
                                     v-bind:class="{ active: item === selectedDistrict }"
                                     @click=setSelectedDist(index)>
            {{ item.label }}
          </div>
        </div>
      </div>

      <div>
      
      </div>
      
    </div>

    <div class="picker-sm">
      <div class="picker-control" @click="shouldShowScrollPicker = 'true'">
        <span>{{ selectedDistrictStr }}</span>
      </div>
      <div class="picker-bg" v-if="shouldShowScrollPicker">
        <div class="dist-picker">
          <div class="dist-picker-btn-box">
            <div class="picker-btn picker-btn-cancel"
              @click="cancelSelection">取消</div>
            <div class="picker-btn picker-btn-ok"
              @click="confirmSelection">确定</div>
          </div>

          <div class="dist-picker-list">
            <scroll-picker-group class="flex">
              <scroll-picker 
                :options="citiesScrollPicker"
                @input="setSelectedCityFromScrollPicker"
                v-model="selectedCityID"
                >
              </scroll-picker>
              <scroll-picker 
                :options="districtsScrollPicker"
                @input="setSelectedDistFromScrollPicker"
                v-model="selectedDistrictID">
              </scroll-picker>
            </scroll-picker-group>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ScrollPicker, ScrollPickerGroup } from 'vue-scroll-picker';
export default {
  name: 'VueDistrictPicker',
  components: {
    ScrollPicker,
    ScrollPickerGroup,
  },
  props: {
    options: Array,
  },
  computed: {
    selectedDistrictStr: function() {
      var str = '请选择';
      if (this.selectedCity !== null && this.selectedDistrict !== null) {
        for (let city of this.options) {
          if (city.id === this.selectedCity.id) {
            for (let dist of city.children) {
              if (dist.id == this.selectedDistrict.id) {
                str = dist.label;
              }
            }
          }
        }
      }
      return str;
    },

    citiesScrollPicker: function() {
      var cities = [];
      for(let city of this.options) {
        cities.push({name: city.label, value: city.id})
      }
      return cities;
    },

    districtsScrollPicker: function() {
      var districts = [];
      if (this.selectedCity !== null) {
        for (let district of this.selectedCity.children) {
          districts.push({name: district.label, value: district.id});
        }
      }
      return districts;
    }
  }, 

  data() {
    return {
      showOptions: false,
      shouldShowScrollPicker: false,
      selectedDistrict: null,
      selectedCity: null,
      selectedCityID: '',
      selectedDistrictID: '',
    }
  },

  created() {
    this.selectedCity = this.options[0];
  },

  methods: {
    cancelSelection() {
      this.shouldShowScrollPicker = false;
    },
    showDistrictPicker() {
      this.shouldShowScrollPicker = true;
    },
    showDropdown() {
      this.showOptions = !this.showOptions;
    },

    setSelectedCity(index) {
      this.selectedCity = this.options[index];
      this.selectedCityID = this.selectedCity.id;
    },
    setSelectedDist(index) {
      this.selectedDistrict = this.selectedCity.children[index];
      this.selectedDistrictID = this.selectedDistrict.id;
      this.showOptions = false;
      this.confirmSelection();
    },

    setSelectedCityFromScrollPicker(value) {
      for(let city of this.options) {
        if (city.id == value) {
          this.selectedCity = city;
        }
      }
    },

    setSelectedDistFromScrollPicker(value) {
      for(let dist of this.selectedCity.children) {
        if (dist.id == value) {
          this.selectedDistrict = dist;
        }
      }
    },

    confirmSelection() {
      this.$emit('district-set', {'city': this.selectedCity.id, 'district': this.selectedDistrict.id});
      this.showOptions = false;
      this.shouldShowScrollPicker = false;
    }
  }
}

</script>

<style scoped>
.district-item {
  display: inline-block;
  line-height: 20px;
  padding: 10px 5px;
  margin: 5px;
  text-align: left;
  cursor: pointer;
  border: 1px solid #efefef;
  border-radius: 4px;
}

.district-item.active {
  color: #ffbe00;
}

.district-list {
  padding: 10px 20px;
  text-align: left;
}
.picker-btn-ok {
  position: absolute;
  right: 0;
}

.picker-btn-cancel {
  position: absolute;
  left: 0;
}

.picker-btn {
  display: inline-block;
  padding: 0 20px;
  color: black;
}
.dist-picker-list {
  position:absolute;
  top: 50px;
  width: 100%;
  font-size: 24px;
}
.dist-picker-btn-box {
  position: absolute;
  height: 50px;
  line-height:50px;
  width: 100%;
  background: #ddd;
}
.dist-picker {
  position: absolute;
  bottom: 0;
  height: 250px;
  width: 100%;
  background-color: #FFF;
}
.picker-bg {
  position: fixed;
  top: 0;
  left: 0;
  background: rgba(75,75,75,.65);
  height: 100%;
  width: 100%;
},
.picker-wrapper {
  z-index: 2;
}
.picker-control {
  height: 50px;
  line-height: 50px;
  margin: 0 30px;
  text-align: right;
}
.area-dropdown {
  position: absolute;
  left: 0;
  top: 51px;
  width: 100%;
  background: #f7f9fa;
  border-radius: 5px;
}
.city-list {
  height: 50px;
  background: #f0f0f0;
  list-style: none;
  display: flex;
  flex-flow: row;
}
.city-item {
  margin: 0 25px;
  line-height: 50px;
  cursor: pointer;
}

.city-item.active {
  border-bottom: 2px solid #ffbe00;
}

.city-item:hover {
  color: #ffbe00;
}

.picker-lg {
  position: relative;
  display: none;
}
.picker-sm {
  display: block;
}


@media screen and (min-width: 999px) {
  .picker-control {
    height: 50px;
    line-height: 50px;
    margin: 0 30px;
    text-align: left;
  }
  .picker-lg {
    display: block;
    position: relative;
  }
  .picker-sm {
    display: none;
  }
}
</style>
<style>
.vue-scroll-picker-item.-selected {
  color: #007BFF;
}

.vue-scroll-picker-layer .top {
  height: 43% !important;
}

.vue-scroll-picker-layer .bottom {
  height: 44% !important;
}
</style>
