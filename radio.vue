<template>
    <div class="filter_radio">
        <el-radio-group v-model="currentValue">
            <el-radio v-for="(item, index) in selectitemlist" border class="filter_radio_item"
                :key="index" :label="JSON.stringify(item)" @change="(value)=>{updateVal(value)}">{{item.name}}
            </el-radio>
        </el-radio-group>
    </div>
</template>

<script>
    export default {
        name: "checkbox",
        props:{
            value: String,
            selectitemlist: Array, //选项列表
        },
        data(){
            return {
                currentValue: this.value
            }
        },
        methods:{
            updateVal(value){
                //触发事件，并向父级传入新值
                this.$emit('input', value);
                this.$emit('change', value);
            }
        },
        watch:{
            value:{
                handler(newVal, oldVal){
                    //console.log(newVal);
                    this.currentValue = newVal;
                }
            }
        }
	}
</script>

<style lang="scss" type="text/css">
    .filter_radio{
        //radio重置样式
        .filter_radio_item.is-bordered{
            margin: 4px !important;
            background-color: #fff; 
        }
        .filter_radio_item .el-radio__inner{
            display: none;
        }
    }
</style>