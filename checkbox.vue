<template>
    <div class="filter_checkbox">
        <el-checkbox :indeterminate="indeterminate" v-model="checkAll" class="filter_checkall"
            @change="checkAllFun" v-show="isCheckAll">全选
        </el-checkbox>
        <el-checkbox-group v-model="currentValue"
            @change="updateVal">
            <el-checkbox v-for="(item, index) in options" border class="filter_checkbox_item"
                :key="index" :label="finalValue(item)">{{item[finalProps.label]}}
            </el-checkbox>
        </el-checkbox-group>
    </div>
</template>

<script>
    import {defaultProps} from "./config.js"

    export default {
        name: "checkbox",
        props:{
            value: { required: true },
            options: Array, //选项列表
            props: Object, 
            special: Boolean, //返回完整JSON.stringify(item)
            isCheckAll: Boolean //是否出现全选按钮
        },
        data(){
            return {
                currentValue: this.value,
                indeterminate: false,
                checkAll: false
            }
        },
        computed:{
            finalProps(){
                return this.props || defaultProps;
            }
        },
        methods:{
            //全选按钮点击事件
            checkAllFun(bol){
                this.currentValue = bol? this.transOptions(this.options): [];
                this.indeterminate = false;
                this.$emit('input', this.currentValue);
                this.$emit('change', this.currentValue);
            },
            //全选半选不选判断
            checkAllJudge(){
                let selectLength = this.options.length;
                let checkedCount = this.currentValue.length;
                this.checkAll = checkedCount==selectLength;
                //this.checkAll = checkedCount? true: false;
                this.indeterminate = checkedCount>0 && checkedCount<selectLength;
            },
            //转换列表
            transOptions(list){
                if(this.special){
                    return list.map(o=>{return JSON.stringify(o) });
                }else{
                    return list.map(o=>{return o[this.finalProps.value]});
                }
            },
            //返回出去的值
            finalValue(item){ 
                return this.special? JSON.stringify(item): item[this.finalProps.value];
            },
            //向父级传值
            updateVal(value){
                //console.log(value);
                if(this.isCheckAll) this.checkAllJudge();
                
                //触发事件，并向父级传入新值
                this.$emit('input', value);
                this.$emit('change', value);
            }
        },
        watch:{
            value:{
                handler(newVal, oldVal){
                    //console.log(newVal, oldVal);
                    this.currentValue = newVal;
                    if(this.isCheckAll) this.checkAllJudge();
                }
            }
        }
	}
</script>

<style lang="scss" type="text/css">
    .filter_checkbox{
        position: relative;

        // .filter_checkall{
        //     position: absolute;
        //     right: 0;
        //     top: -40px;
        // }

        //重置样式
        .el-checkbox-group{
            margin-top: 4px;
        }
        .filter_checkbox_item.is-bordered{
            margin: 4px !important;
            background-color: #fff; 
        }
        .filter_checkbox_item .el-checkbox__inner{
            display: none;
        } 
    }
</style>