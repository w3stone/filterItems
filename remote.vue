<!--select自动补全(调网络接口)-->
<template>
    <div class="filter_select">
        <el-select v-model="currentValue" :multiple="multiple" :async="async"
            filterable remote reserve-keyword placeholder="请输入关键词" 
            
            :remote-method="remoteMethod"
            
            :collapse-tags="multiple"
            
            @change="updateVal">
            
            <el-option v-for="(item,index) in finalOption"
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
        name: "remote",
        props:{
            value: {
                required: true
            },
            selectitemlist: Array, //选项列表
            apiName: String, //接口名
            ownName: String,
            selectionId: Number,
            parentName: String,
            multiple: Boolean, //是否多选
            dataActionsBox: Boolean, //是否含有全选&全不选
            async: Boolean //是否走接口
        },
        data(){
            return {
                currentValue: this.value,
                originList: [], //完整字典表
                options: []
            }
        },
        computed:{
            finalOption(){
                if(this.multiple && this.dataActionsBox && this.options.length>0){
                    this.options.unshift({value:'', name:'', disabled:true});
                    return this.options;
                }else{
                    return this.options;
                }
            }
        },
        mounted(){
            this.init();
        },
        methods:{
            init(){ //初始化
                this.originList = this.selectitemlist || []; //完整字典表
                this.options = this.selectitemlist.slice(0,100);
            },
            //重置options(自动补全多选网络相关)
            remoteMethod(queryString){
                if(!this.async){ //走本地
                    let list = this.originList;
                    let result = queryString? list.filter((o)=>{ return o.name.indexOf(queryString)!=-1 }): list.slice(0,100);
                    this.options = result;
                    
                }else{ //走网络
                    if(queryString){
                        let params = {};
                        params.selectionId = this.selectionId;

                        if(this.parentName){
                            params[this.parentName] = queryString;
                        }else if(this.ownName){
                            params[this.ownName] = queryString; //补充参数
                        }else{
                            this.options = [];
                            return false;
                        }

                        this.$dataPost(this.apiName, params, (data)=>{
                            let result = data.selectitemlist;
                            this.options = result || [];
                        });
                    }else{
                        this.options = [];
                    }      
                }  
            },
            //全选
            selectAll(){
                this.currentValue = this.options.filter((o,i)=>{
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
            },
            selectitemlist:{
                handler(newVal, oldVal){
                    this.init();
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