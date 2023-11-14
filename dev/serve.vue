<script>
import Vue from 'vue';
import RangeControler from '@/range-controler.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    RangeControler
  },
  data(){
    return{
      arr: [],
      aimAttri: {
        discount: 0,
        desc: ''
      },
    }
  }
});
</script>

<template>
  <div id="app">
    <range-controler 
      :configArr.sync="arr" 
      label="充值点券范围区间" 
      :purpose="true" 
      :step="1" 
      :aimAttri="aimAttri"
      :add-step="1000"
      :infinite="false" 
    >
      <template v-slot:purpose="scope">
        <label for="discount">Gift amount ratio</label><input type="number" name="discount" v-model="arr[scope.row.index].discount" :min="1" :max="100" label="描述文字">%
        <label for="desc">赠送物品</label><input name="desc" v-model="arr[scope.row.index].desc" type="text">
      </template>
    </range-controler>
  </div>
</template>
