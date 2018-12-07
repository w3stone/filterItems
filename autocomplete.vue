<!--select自动补全(调网络接口)-->
<template>
    <div class="filter_select">
        <el-select v-model="currentValue" :multiple="multiple" filterable remote reserve-keyword 
            placeholder="请输入关键词" :remote-method="(queryString)=>{
                remoteMethodAsync(queryString, item.id, ownName, parentName);
            }" :collapse-tags="multiple"
            @change="(value)=>{updateVal(value)}">
            <el-option v-for="item in options"
                :key="item.value" :label="item.name" :value="JSON.stringify(item)">
            </el-option>
        </el-select>
    </div>
</template>

<script>
    export default {
        name: "remoteAsync",
        props:{
            value: {
                required: true
            },
            selectitemlist: Array, //选项列表
            apiName: String, //接口名
            ownName: String,
            parentName: String,
            multiple: Boolean
        },
        data(){
            return {
                currentValue: this.value,
                originList: this.selectitemlist,
                options: this.selectitemlist.slice(0,100)
            }
        },
        methods:{
            //重置options(自动补全多选本地相关)
            querySearchLocal(queryString, callback, list){
                //console.log(list);
                let result = queryString? list.filter((o)=>{ return o.name.indexOf(queryString)!=-1 }): list.slice(0,100);
                result = result.map((o)=>{return {"value": o.name}});
                callback(result);
            },
            //重置options(自动补全多选网络相关)
            querySearchAsync(queryString, callback, selectionId, ownName) {
                if(!queryString){
                    callback([]);
                    return false;
                };

                let params = {};
                params.selectionId = selectionId;
                params[ownName] = queryString; //补充参数

                this.$dataPost(this.apiName, params, (data)=>{
                    //console.log(data);
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
            selectitemlist:{
                handler(newVal, oldVal){
                    this.originList = newVal;
                    this.options = newVal.slice(0,100);
                }
            }
        }
	}
</script>

<style lang="scss" type="text/css">
    
</style>