<template>
  <div class="el-vue-search-box-container"
      :class="classToggle?'mini':''"
       @keydown.up="selectTip('up')"
       @keydown.down="selectTip('down')">
    <div class="search-box-wrapper">
      <i class="el-input__icon el-icon-arrow-right" @click="toggle(false)"></i>
      <input type="text"
        v-model="keyword"
        @keyup.enter="search"
        @input="autoComplete">
         <i class="search-clear el-icon-error" @click="searchClear" v-show="keyword.length!==0 ? true : false"></i>
      <span class="search-btn" @click="classToggle?toggle(true):search"><i class="el-icon-search el-input__icon"></i></span>
    </div>
    <div class="search-tips">
      <ul>
        <li v-for="(tip, index) in tips"
          :key="index"
          @click="changeTip(tip)"
          @mouseover="selectedTip=index"
          :class="{'autocomplete-selected': index === selectedTip}">{{tip.name}}</li>
      </ul>
    </div>
  </div>
</template>
<script>
import RegisterComponentMixin from '../mixins/register-component';
import {lazyAMapApiLoaderInstance} from '../services/injected-amap-api-instance';
export default {
  name: 'el-amap-search-box',
  mixins: [RegisterComponentMixin],
  props: ['searchOption', 'onSearchResult', 'events', 'default'],
  data() {
    return {
      keyword: this.default || '',
      tips: [],
      selectedTip: -1,
      loaded: false,
      classToggle: true
    };
  },
  mounted() {
    let _loadApiPromise = lazyAMapApiLoaderInstance.load();
    _loadApiPromise.then(() => {
      this.loaded = true;
      this._onSearchResult = this.onSearchResult;
      // register init event
      this.events && this.events.init && this.events.init({
        autoComplete: this._autoComplete,
        placeSearch: this._placeSearch
      });
    });
  },
  computed: {
    _autoComplete() {
      if (!this.loaded) return;
      return new AMap.Autocomplete(this.searchOption || {});
    },
    _placeSearch() {
      if (!this.loaded) return;
      return new AMap.PlaceSearch(this.searchOption || {});
    }
  },
  methods: {
    searchClear() {
      this.keyword = [];
      this.tips = [];
    },
    autoComplete() {
      if (!this.keyword || !this._autoComplete) return;
      this._autoComplete.search(this.keyword, (status, result) => {
        if (status === 'complete') {
          this.tips = result.tips;
        }
      });
    },
    search() {
      this.tips = [];
      if (!this._placeSearch) return;
      this._placeSearch.search(this.keyword, (status, result) => {
        if (result && result.poiList && result.poiList.count) {
          let {poiList: {pois}} = result;
          let LngLats = pois.map(poi => {
            poi.lat = poi.location.lat;
            poi.lng = poi.location.lng;
            return poi;
          });
          this._onSearchResult(LngLats);
        } else if (result.poiList === undefined) {
          throw new Error(result);
        }
      });
    },
    changeTip(tip) {
      this.keyword = tip.name;
      this.search();
    },
    selectTip(type) {
      if (type === 'up' && this.selectedTip > 0) {
        this.selectedTip -= 1;
        this.keyword = this.tips[this.selectedTip].name;
      } else if (type === 'down' && this.selectedTip + 1 < this.tips.length) {
        this.selectedTip += 1;
        this.keyword = this.tips[this.selectedTip].name;
      }
    },
    toggle(bool) {
      this.classToggle = bool;
      this.keyword = '';
      this.tips = [];
    }
  },
  watch: {
    keyword() {
      let that = this;
      console.log(this.keyword);
      setTimeout(()=>{
        if (that.keyword.length === 0) {
          that.tips = [];
        }
      }, 300);
    }
  }
};
</script>
