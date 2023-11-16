# range-controler

## Description
This is a Vue2 based component and is used to represent what users want to do in different ranges.  For example, this component can be used to control customers to receive different recharge rewards as the recharge amount changes in a recharge activity.   

## Installation
You need node.js and npm
```
npm i range-controler
```

in xxx.vue
```
import inputRangeConfiger from 'range-controler'

components: {
    inputRangeConfiger
},

```
   
For example, When used with the [el-dialog](https://element.eleme.cn) component:
```
<el-dialog title="" :visible.sync="dialogVisible" width="60%">
    <el-form>
    <inputRangeConfiger 
        :configArr.sync="arr" 
        label="Recharge amount" 
        :purpose="true" 
        :step="1" 
        :aimAttri="aimAttri"
        :add-step="1000"
        :infinite="false"
    >
        <template v-slot:purpose="scope">
        <el-form :inline="true">
            <el-form-item label="Additional gift ratio：">
            <el-input-number v-model="arr[scope.row.index].discount" :min="1" :max="100" ></el-input-number>%
            </el-form-item>
            <el-form-item label="Gifts：">
            <el-input v-model="arr[scope.row.index].gift" type="text"></el-input>
            </el-form-item>
        </el-form>
        </template>
    </inputRangeConfiger>
    </el-form>
    <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">cancel</el-button>
    <el-button type="primary" @click="dialogVisible = false">confirm</el-button>
    </span>
</el-dialog>

data(){
    return {
        arr:[
            {
                from:0,
                to:1000,
            },
            {
                from:1001,
                to:2001
            }
        ],
        aimAttri: {
            discount: 0,
            gift: ''
        },
    }
}
```
after I set the values of `discount` and `gift`:

![image](https://github.com/pujiaming/range-configer/assets/109517537/0acc0038-a411-44b0-ae98-07b6b2446ca7)

and its corresponding results:

```javascript
 [{ from: 0, to: 1000, discount: 1, gift: 'Random S-level skin' }, { from: 1001, to: 2001, discount: 2, gift: 'Custom spraying' }]
```   

Of course, you can modify the field name through different attributes in the following document.   

Reminder Information: The ranges of the interval here will `not overlap`, and when any one range is modified, the values of other ranges will also change, with the manually modified range having the `highest priority`.   

and here you can add a attribute `slideble` to change the range by sliding
![image](https://github.com/pujiaming/range-configer/assets/109517537/9a339241-64e7-426f-8113-750eafe9bb0d)

## Attributes
| Attribute |  Description | Type | Default |
| --- | --- | --- | --- |
| configArr | binding value |  Array | [{ from : 0, to: 5 } ] |
| step | incremental step and mutatioin step | Number | 1 |
| label | label text | String | Range |
| purpose | Whether to use slots for specific purposes | Boolean | false |
| aimAttri | specific purposes object. It only takes effect when `purpose` is true | Object | { aimrow: 0 } |
| readonly | same as readonly in native input, but pay attention to the form elements in "templete v-slot:purpose", you can add readonly attribute manually | Boolean | false |
| minNum | minimum number of intervals | Number | 1 |
| addStep | The range size of the newly added item | Number | 1000 |
| infinite | Can it end with 0 and represent infinity | Boolean | true |
| from | Custom fields represent the starting point of the range | String | from |
| to | Custom fields represent the endpoint of the range | String | to |
| numLabel | range amount label | String | range amount |      
| slidable | change the value by sliding | Boolean | false |
