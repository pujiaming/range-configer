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
      arr: [
        {from:0, to:1200},
        {from:1201, to:2200},
        {from:2201, to:3200},
      ],
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
      label="Recharge amount" 
      :purpose="true" 
      :step="1" 
      :aimAttri="aimAttri"
      :add-step="1000"
      :infinite="false"
      :slidable="true"
    >
      <template v-slot:purpose="scope">
        <label for="discount">Gift amount ratio</label><input type="number" name="discount" v-model="arr[scope.row.index].discount" :min="1" :max="100">%
        <label for="desc">desc</label><input name="desc" v-model="arr[scope.row.index].desc" type="text">
      </template>
    </range-controler>
  </div>
</template>
