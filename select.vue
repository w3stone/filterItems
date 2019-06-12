<!--select封装-->
<template>
    <div class="filter_select">
        <el-select v-model="currentValue" :multiple="multiple" :async="async" :disabled="disabled"
            :filterable="remote" :remote="remote" :reserve-keyword="remote" 
            :placeholder="!placeholder? remote?'请输入关键词':'请选择': placeholder"
            
            :remote-method="remoteMethod"
            
            :collapse-tags="multiple && collapseTags"
            
            @change="updateVal">

            <el-option label="" value="" :disabled="true" v-if="multiple && dataActionsBox && originOptions.length<=100">
                <button @click="selectAll" class="select_btn" style="float:left;">全选</button>
                <button @click="selectNone" class="select_btn" style="float:right;border-left:none;">全不选</button>
            </el-option>
            
            <el-option v-for="(item,index) in currentOptions"
                :key="index" :disabled="item.disabled"
                :label="item[finalProps.label]" :value="finalValue(item)">
            </el-option>
        </el-select>
    </div>
</template>

<script>
    import {defaultProps} from "./config.js"

    export default {
        name: "selects",
        props:{
            value: {
                required: true
            },
            options: Array, //选项列表
            props: Object,
            disabled: Boolean, //禁用状态
            placeholder: String,
            special: Boolean, //返回完整JSON.stringify(item)
            /**多选相关**/
            multiple: Boolean, //是否多选
            dataActionsBox: Boolean, //是否含有全选&全不选
            collapseTags: Boolean, //选中项是否折叠
            /**是否可搜索相关**/
            remote: Boolean,
            /**调接口相关**/
            async: Boolean, //是否走接口
            apiName: String, //接口名
            method: String, //Get or Post
            params: Object, //参数
            /**其它**/
            ownName: String,
            selectionId: Number,
            parentName: String
        },
        data(){
            return {
                currentValue: this.value,
                originOptions: [], //完整字典表
                currentOptions: []
            }
        },
        computed:{
            finalProps(){
                return this.props || defaultProps;
            }
        },
        mounted(){
            this.init();
            this.checkValue();
        },
        methods:{
            init(){ //初始化
                //console.log("初始化");
                this.originOptions = JSON.parse(JSON.stringify(this.options)) || []; //完整字典表
                this.currentOptions = (this.options.length>100 && this.remote)? this.options.slice(0,50): this.options;
            },
            //判断值是否在currentOptions里，如果没有则插入
            checkValue(){
                let check = (value) => {
                    let item = null;
                    if(this.special){
                        let item = JSON.parse(value);
                        if(this.currentOptions.filter(o => o[this.finalProps.value]==item[this.finalProps.value]).length>0) return true;
                        this.currentOptions.push(item);

                    }else{
                        // console.log(value);
                        // console.log(this.currentOptions);
                        if(this.currentOptions.filter(o => o[this.finalProps.value]==value).length>0) return true;

                        let temp = this.originOptions.filter(o => o[this.finalProps.value]==value);
                        this.currentOptions = this.currentOptions.concat(temp); //合并
                    }
                };

                //执行
                if(this.currentValue){
                    if(this.multiple){ //多选
                        this.currentValue.forEach(value => {
                            check(value);
                        });
                    }else{ //单选
                        let value = this.currentValue;
                        check(value);
                    }
                }
            },
            //重置options(自动补全多选网络相关)
            remoteMethod(queryString){
                if(!this.async){ //走本地
                    let list = this.originOptions;
                    //console.log(queryString, list);
                    let result = queryString? list.filter(o => o[this.finalProps.label].indexOf(queryString)!=-1).slice(0,50): list.slice(0,50);
                    this.currentOptions = result;
                    
                }else{ //走网络
                    if(queryString && this.apiName){
                        let params = {};
                        params.selectionId = this.selectionId;

                        if(this.parentName){
                            params[this.parentName] = queryString;
                        }else if(this.ownName){
                            params[this.ownName] = queryString; //补充参数
                        }else{
                            return false;
                        }

                        //console.log("可调接口！");
                        //console.log(this.apiName, this.ownName, this.method)

                        this.$dataPostCommon(this.apiName, params, (data)=>{
                            let result = data.selectitemlist;
                            this.currentOptions = result || [];
                        });
                    }else{
                        this.currentOptions = [];
                    }      
                }  
            },
            //全选事件
            selectAll(){
                this.currentValue = this.originOptions.filter(o => { //从原始数据中选择
                    return o[this.finalProps.value] || o[this.finalProps.value]==0
                }).map(o=>{return this.finalValue(o)});

                this.updateVal(this.currentValue);
            },
            //全不选事件
            selectNone(){
                this.currentValue = [];
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
                    //console.log(newVal, oldVal);
                    this.currentValue = newVal;
                    this.checkValue();
                }
            },
            options:{
                handler(newVal, oldVal){
                    //console.log(newVal, oldVal);
                    this.init();
                    this.checkValue();
                }
            }
        }
	}
</script>

<style lang="scss" type="text/css">
    .filter_select{
        display: inline-block;
    }

    .el-select-dropdown.is-multiple{
        .select_btn{
            width: 50%;
            box-sizing: border-box;
            background-color: transparent;
            height: 100%;
            line-height: 30px;
            border: 1px solid #ccc;
            border-radius: 2px;
            margin: 0;
            cursor: pointer;
        }
    }
</style>