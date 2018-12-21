<!--select单选-->
<template>
    <div class="filter_select">
        <el-select v-model="currentValue" :multiple="multiple" :collapse-tags="multiple && collapseTags" 
            placeholder="请选择" @change="updateVal">
            <el-option-group v-for="group in groupOptions"
                :key="group.name" :label="group.name" @click.native="multiSelectAll(group.name)">
                
                <el-option v-for="(itemc,index) in group.options"
                    :key="index" :label="itemc[finalProps.label]"
                    :value="finalValue(itemc)">
                </el-option>
            </el-option-group>
        </el-select>
    </div>
</template>

<script>
    import {defaultProps} from "./config.js"

    export default {
        name: "selecterGroup",
        props:{
            value: {
                required: true
            },
            options: Array, //选项列表
            multiple: Boolean, //是否多选
            collapseTags: Boolean, //选中项是否折叠
            special: Boolean //返回完整JSON.stringify(item)
        },
        data(){
            return {
                currentValue: this.value
            }
        },
        computed:{
            groupOptions(){
                let level1NameList = Enumerable.from(this.options).select(o=>o[this.finalProps.otherLabel]).distinct().toArray();
    
                let options = [];
                level1NameList.forEach(itemName =>{
                    options.push({
                        "name": itemName,
                        "options": this.options.filter(o=>{return o[this.finalProps.otherLabel]==itemName})
                    });
                })
                return options;
            },
            finalProps(){
                return this.props || defaultProps;
            }
        },
        methods:{
            //多级select全选
            multiSelectAll(value){
                if(!this.multiple) return false; //如果是单选,退出

                let level1list = this.groupOptions.filter((o)=>{return o[this.finalProps.label]==value;}); //当前一级菜单
                let level2list = level1list[0].options;
                let level2ValueList = level2list.map(o=>this.finalValue(o));

                let num=0; //计数器
                level2ValueList.forEach((value)=>{
                    this.currentValue.forEach((model, index)=>{
                        if(model==value){
                            //debugger
                            this.currentValue.splice(index, 1);
                            num++;
                            return true;
                        }
                    })
                });

                if(num < level2ValueList.length){ //全选
                    this.currentValue = this.currentValue.concat(level2ValueList); //合并数组
                }

                this.updateVal(this.currentValue);
            },
            //返回出去的值
            finalValue(item){ 
                return this.special? JSON.stringify(item): item[this.finalProps.value];
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