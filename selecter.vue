<!--select单选-->
<template>
    <div class="filter_select">
        <el-select v-model="currentValue" :multiple="multiple" :collapse-tags="multiple"
            placeholder="请选择" @change="updateVal">

            <el-option v-for="(item, index) in finalSelectitemlist"
                :key="index" :disabled="item.disabled"
                :label="item.name" :value="JSON.stringify(item)">
            
                <button @click="selectAll" class="select_btn" style="float:left;" 
                    v-if="index==0 && multiple && dataActionsBox">全选</button>

                <button @click="selectNone" class="select_btn" style="float:right;border-left:none;" 
                    v-if="index==0 && multiple && dataActionsBox">全不选</button>

            </el-option>
        </el-select>
    </div>
</template>

<script>
    export default {
        name: "selecter",
        props:{
            value: {
                required: true
            },
            selectitemlist: Array, //选项列表
            multiple: Boolean, //是否多选
            dataActionsBox: Boolean//是否含有全选&全不选
        },
        data(){
            return {
                currentValue: this.value
            }
        },
        computed:{
            finalSelectitemlist(){
                if(this.multiple && this.dataActionsBox && this.selectitemlist.length>0){
                    this.selectitemlist.unshift({value:'', name:'', disabled:true});
                    return this.selectitemlist;
                }else{
                    return this.selectitemlist;
                }
            }
        },
        methods:{
            //全选
            selectAll(){
                this.currentValue = this.finalSelectitemlist.filter((o,i)=>{
                    return i!=0 && o.value
                }).map(o=>{return JSON.stringify(o)});

                this.updateVal(this.currentValue);
            },
            //全不选
            selectNone(){
                this.currentValue = [];
                this.updateVal(this.currentValue);
            },
            //向父级传值
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
    .el-select-dropdown.is-multiple{
        .select_btn{
            width: 50%;
            box-sizing: border-box;
            background-color: transparent;
            height: 30px;
            line-height: 30px;
            border: 1px solid #ccc;
            border-radius: 2px;
            margin: 0;
            cursor: pointer;
        }
    }
</style>