<template>
    <div class="filter_radio">
        <el-radio-group v-model="currentValue">
            <el-radio v-for="(item, index) in options" border class="filter_radio_item"
                :key="index" :label="finalValue(item)" @change="(value)=>{updateVal(value)}">{{item[finalProps.label]}}
            </el-radio>
        </el-radio-group>
    </div>
</template>

<script>
    import {defaultProps} from "./config.js"

    export default {
        name: "checkbox",
        props:{
            value: {
                required: true
            },
            options: Array, //选项列表
            props: Object, 
            special: Boolean //返回完整JSON.stringify(item)
        },
        data(){
            return {
                currentValue: this.value
            }
        },
        computed:{
            finalProps(){
                return this.props || defaultProps;
            }
        },
        methods:{
            finalValue(item){ //返回出去的值
                return this.special? JSON.stringify(item): item[this.finalProps.value];
            },
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