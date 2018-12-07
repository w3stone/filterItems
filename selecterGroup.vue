<!--select单选-->
<template>
    <div class="filter_select">
        <el-select v-model="currentValue" :multiple="multiple" :collapse-tags="multiple" 
            placeholder="请选择" @change="updateVal">
            <el-option-group v-for="group in groupSelectitemlist"
                :key="group.name" :label="group.name" @click.native="multiSelectAll(group.name)">
                
                <el-option v-for="(itemc,index) in group.options"
                    :key="index" :label="itemc.name"
                    :value="JSON.stringify(itemc)">
                </el-option>
            </el-option-group>
        </el-select>
    </div>
</template>

<script>
    export default {
        name: "selecterGroup",
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
            groupSelectitemlist(){
                let selectitemlist = this.selectitemlist;
                let level1NameList = Enumerable.from(selectitemlist).select(o=>o.cname).distinct().toArray();
    
                let options = [];
                level1NameList.forEach(itemName =>{
                    options.push({
                        "name": itemName,
                        "options": selectitemlist.filter(o=>{return o.cname==itemName})
                    });
                })
                return options;
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
            //多级select全选
            multiSelectAll(value){
                if(!this.multiple) return false; //如果是单选,退出

                this.currentValue = []; //清空所有
                let level1list = this.groupSelectitemlist.filter((o)=>{return o.name==value;}); //当前一级菜单
                let level2list = level1list[0].options; //所有当前二级菜单集合
                if(this.currentValue.length < level2list.length){ //全选
                    this.currentValue = level2list.map(o=>JSON.stringify(o));
                }else{ //取消全选
                    this.currentValue = [];
                }
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
    .el-select-dropdown{
        .el-select-group__wrap{
            .el-select-group__title {
                font-size: 16px !important;
                line-height: 36px !important;
            }
            .el-select-dropdown__item{
                text-indent: 6px;
            }
        }
    }
</style>