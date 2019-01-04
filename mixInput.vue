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
                <i class="el-icon-circle-plus" @click="add(index)"></i>
                <i class="el-icon-remove" v-show="inputModel.length>1" @click="del(index)"></i>
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
                let results = queryString ? options.filter(o=>{ return o.value.indexOf(queryString)!=-1 }): options;
                // 调用 callback 返回建议列表的数据
                callback(results);
            },
            //元素被选中事件
            selected(data){ 
                //console.log(data);
                this.$emit("selected", data);
            },
            // stringChanged(data){
            //     let finalArray = JSON.parse(JSON.stringify(this.inputModel));
            //     finalArray = this.removeEmpty(finalArray);
            //     this.$emit("input", finalArray.join(","));
            //     this.$emit("change", finalArray.join(","));
            // },
            //去除数组空元素
            removeEmpty(list){
                for(var i=0; i<list.length; i++){
                    if(list[i]=="" || typeof(list[i])=="undefined"){
                        list.splice(i,1);
                        i--;
                    }
                }
                return list;
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
                //immediate: true,
                handler(newVal, oldVal){
                    let finalArray = JSON.parse(JSON.stringify(newVal));
                    finalArray = this.removeEmpty(finalArray);
                    this.$emit("input", finalArray.join(","));
                    this.$emit("change", finalArray.join(","));
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