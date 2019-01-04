<!--多个input-->
<template>
    <div class="params_plus">
        <div v-for="(item, index) in inputModel" :key="index" class="clearfix">
            <el-input class="input_item" v-model="inputModel[index].key" placeholder="请输入key" 
                @change="paramsChanged">
            </el-input>
            
            <el-input class="input_item" v-model="inputModel[index].value" placeholder="请输入value" 
                @change="paramsChanged">
            </el-input>

            <div class="control_box">
                <i class="el-icon-circle-plus" @click="add(index)"></i>
                <i class="el-icon-remove" v-show="inputModel.length>1" @click="del(index)"></i>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name:'paramsPlus',
        props:{
            value: {type: Object, required: true}
        },
        data(){
            return{
                inputModel: [{key:"",value:""}]
            }
        },
        methods: {
            add(index){
                this.inputModel.splice(index+1, 0, {key:"",value:""}); //从index+1索引开始插入
            },
            del(index){
                if(this.inputModel.length>1)
                    this.inputModel.splice(index, 1); //删除当前节点
            },
            paramsChanged(){
                let params = {};
                this.inputModel.forEach(item => {
                    if(item.key!="") params[item.key] = item.value;
                });

                this.$emit("input", params);
                this.$emit("change", params);
            }
        },
        watch:{
            value:{ //监测父组件值变化
                deep: true,
                handler(newVal, oldVal){
                    //console.log(newVal, oldVal);
                    this.inputModel = [];
                    for(var key in newVal){
                        this.inputModel.push({"key":key, "value":newVal[key]});
                    }
                }
            }
        }

	}
</script>

<style lang="scss" scoped="" type="text/css">
    .params_plus{
        .input_item{
            float: left;
            width: 45%;
            margin-right: 5px;
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