<template>
    <div class="filter_checkbox">
        <el-checkbox :indeterminate="currentIndeterminate" v-model="currentCheckAll" class="filter_checkall"
            @change="checkAllFun">全选
        </el-checkbox>
        <el-checkbox-group v-model="currentValue"
            @change="updateVal">
            <el-checkbox v-for="(item, index) in selectitemlist" border class="filter_checkbox_item"
                :key="index" :label="JSON.stringify(item)">{{item.name}}
            </el-checkbox>
        </el-checkbox-group>
    </div>
</template>

<script>
    export default {
        name: "checkbox",
        props:{
            value: { required: true },
            selectitemlist: Array, //选项列表
            isIndeterminate: Boolean,
            checkAll: Boolean
        },
        data(){
            return {
                currentValue: this.value,
                currentIndeterminate: this.isIndeterminate,
                currentCheckAll: this.checkAll
            }
        },
        methods:{
            //全选按钮点击事件
            checkAllFun(value){
                this.currentValue = value? this.transSearchList(this.selectitemlist): [];
                this.currentIndeterminate = false;
                this.$emit('input', this.currentValue);
            },
            //全选半选不选判断
            checkAllJudge(){
                let selectLength = this.selectitemlist.length;
                let checkedCount = this.currentValue.length;
                //this.currentCheckAll = (checkedCount==selectitemlist.legnth);
                this.currentCheckAll = checkedCount? true: false;
                this.currentIndeterminate = checkedCount>0 && checkedCount<selectLength;
            },
            //转换列表
            transSearchList(list){
                let newArr = [];
                list.forEach((item)=>{
                    newArr.push( JSON.stringify(item) )
                });
                return newArr;
            },
            //向父级传值
            updateVal(value){
                this.checkAllJudge();
                this.$emit('input', value); //触发事件，并向父级传入新值
            }
        },
        watch:{
            value:{
                handler(newVal, oldVal){
                    this.currentValue = newVal;
                    this.checkAllJudge();
                }
            }
        }
	}
</script>

<style lang="scss" type="text/css">
    .filter_checkbox{
        position: relative;

        .filter_checkall{
            position: absolute;
            right: 0;
            top: -40px;
        }

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