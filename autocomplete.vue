<!--select自动补全(调网络接口)-->
<template>
    <div class="filter_autocomplete">
        <el-autocomplete class="inline-input"
            v-model="currentValue" :fetch-suggestions="querySearch"
            placeholder="请输入内容" @change="updateVal">
        </el-autocomplete>
    </div>
</template>

<script>
    import {defaultProps} from "./config.js"

    export default {
        name: "autocomplete",
        props:{
            value: {
                required: true
            },
            options: Array, //选项列表
            special: Boolean, //返回完整JSON.stringify(item)
            apiName: String, //接口名
            async: Boolean, //是否走接口
            //其它
            selectionId: Number,
            ownName: String,
            parentName: String
        },
        data(){
            return {
                currentValue: this.value,
                originOptions: [], //完整字典表
                currentOptions: []
            }
        },
        mounted(){
            this.init();
        },
        methods:{
            init(){ //初始化
                this.originOptions = this.options || []; //完整字典表
                this.currentOptions = this.originOptions.slice(0,100);
            },
            querySearch(queryString, callback){
                if(!this.async){ //走本地
                    this.querySearchLocal(queryString, callback);
                }else{

                }
            },
            //重置options(自动补全多选本地相关)
            querySearchLocal(queryString, callback){
                let list = this.originOptions;
                let result = queryString? list.filter(o => o.name.indexOf(queryString)!=-1): list.slice(0,100);
                result = result.map((o)=>{return {"value": o.name}});
                callback(result);
            },
            //重置options(自动补全多选网络相关)
            querySearchAsync(queryString, callback, selectionId, ownName) {
                if(!(queryString && this.apiName && this.selectionId && this.ownName)){
                    callback([]);
                    return false;
                };

                let params = {};
                params.selectionId = this.selectionId;
                params[this.ownName] = queryString; //补充参数

                this.$dataPost(this.apiName, params, (data)=>{
                    let result = data.selectitemlist.map((o)=>{return {"value": o.name}}); //?
                    callback(result);
                });
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
            options:{
                handler(newVal, oldVal){
                    //console.log(newVal);
                    this.init();
                }
            }
        }
	}
</script>

<style lang="scss" type="text/css">
    
</style>