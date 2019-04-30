<!--多个input-->
<template>
    <div class="mixInput">
        <div v-for="(item, index) in inputModel" :key="index">
            <el-autocomplete class="inline-input" placeholder="请输入内容"
                v-model="inputModel[index]"
                :fetch-suggestions="querySearch"
                @select="selected">
            </el-autocomplete>
            <div class="control_box">
                <i class="el-icon-circle-plus" v-show="inputModel[index]" @click="add(index)"></i>
                <i class="el-icon-remove" v-show="inputModel.length>1" @click="del(index)"></i>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name:'mixInput',
        props:{
            value: {type: Array, required: true},
            options: {type: Array}, //字典表
            xd: {type: Boolean}
        },
        data(){
            return{
                inputModel: [""]
            }
        },
        methods: {
            add(index){
                this.inputModel.splice(index+1, 0, ""); //从index+1索引开始插入
            },
            del(index){
                if(this.inputModel.length>1)
                    this.inputModel.splice(index, 1); //删除当前节点
            },
            //搜索方法
            querySearch(queryString, callback) {
                let options = this.options || [];
                let results = queryString ? options.filter(o => o.value.indexOf(queryString)!=-1): options;
                // 调用 callback 返回建议列表的数据
                callback(results);
            },
            //元素被选中事件
            selected(data){ 
                //console.log(data);
                this.$emit("selected", data);
            }
        },
        watch:{
            value:{ //监测父组件值变化
                handler(newVal, oldVal){
                    //console.log(newVal, oldVal);
                    this.inputModel = newVal.length>0? newVal: [""];
                }
            },
            inputModel:{ //监测自己变化，向父组件传值
                handler(newVal, oldVal){
                    //console.log(newVal, oldVal);
                    // if(newVal[newVal.length-1]=="")
                    //     newVal = newVal.pop(); //删除数组最后一位

                    this.$emit("input", newVal);
                    this.$emit("change", newVal);
                }
            }
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
            width: 50px;
            line-height: 40px;
            padding-left: 10px;
            user-select: none;

            i{
                font-size: 18px;
                color: #333;
                cursor: pointer;
                //user-select: none;

                &:hover{
                    color: #666;
                }
            }
        }
        
    }
</style>