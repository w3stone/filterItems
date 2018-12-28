<!--多个input-->
<template>
    <div class="mixInput">
        <div v-for="(item, index) in inputModel" :key="index">
            <el-autocomplete
                class="inline-input"
                v-model="inputModel[index]"
                :fetch-suggestions="querySearch"
                placeholder="请输入内容"
                @select="selected">
            </el-autocomplete>
            <div class="control_box" v-show="xd">
                <i class="el-icon-circle-plus" @click="addFn(index)"></i>
                <i class="el-icon-remove" @click="delFn(index)"></i>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name:'mixInput',
        props:{
            value: {type: String, required: true},
            options: {type: Array}, //字典表
            xd: {type: Boolean}
        },
        data(){
            return{
                inputModel: [""]
            }
        },
        methods: {
            addFn(index){
                this.inputModel.splice(index+1, 0, "");
            },
            delFn(index){
                if(this.inputModel.length > 1)
                    this.inputModel.splice(index, 1); //删除当前节点
            },
            querySearch(queryString, cb) {
                let options = this.options;
                let results = queryString ? options.filter(o=>{ return o.value.indexOf(queryString)!=-1 }): options;
                // 调用 callback 返回建议列表的数据
                cb(results);
            },
            selected(data){
                //console.log(data);
                this.$emit("selected", data);
            }
        },
        watch:{
            value:{ //监测父组件值变化
                handler(newVal, oldVal){
                    //console.log(newVal);
                    this.inputModel = newVal? newVal.split(","): [""];
                }
            },
            inputModel:{ //监测自己变化，向父组件传值
                immediate: true,
                handler(newVal, oldVal){
                    let finalArray = newVal.deepCopy().noEmpty(); //数组深复制
                    //console.log(finalArray.join(","));
                    this.$emit("input", finalArray.join(","));
                    this.$emit("change", finalArray.join(","));
                }
            },
        }

	}
</script>

<style lang="scss" scoped="" type="text/css">
    .mixInput{
        display: inline-block;

        .inline-input{
            float: left;
            margin-bottom: 10px;
        }
        .control_box{
            float: left;
            line-height: 40px;
            padding-left: 10px;
            user-select: none;

            i{
                font-size: 18px;
                color: #333;
                cursor: pointer;
                //user-select: none;
            }
        }
        
    }
</style>