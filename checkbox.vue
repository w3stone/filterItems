<!--筛选条件(核心)-->
<template>
    <div class="filter_body clearfix">
        <!--3、checkbox多选-->
        <el-checkbox :indeterminate="modelList[index].isIndeterminate" v-model="modelList[index].checkAll"
            @change="(val)=>{
                modelList[index].model = val? transSearchList(item.selectitemlist): [];
                modelList[index].isIndeterminate = false;
            }" slot="supply">全选
        </el-checkbox>
        <el-checkbox-group v-model="modelList[index].model"
            @change="(value)=>{
                checkALLFn(index, item.selectitemlist.length);
            }">
            <el-checkbox v-for="(itemc, indexc) in item.selectitemlist" border class="filter_checkbox"
                :key="indexc" :label="JSON.stringify(itemc)">{{itemc.name}}
            </el-checkbox>
        </el-checkbox-group>
    </div>
</template>

<script>
    import {mapState, mapMutations} from 'vuex'
    import filterInputItem from './filterInputItem'
    
    export default {
        name: "filterBody",
        props:{
            searchList: Array,
            hasOthers: Boolean, //是否有其它条件
            visible: Boolean //其它条件单框显示状态
        },
        data(){
            return {
                params:{}, //参数
                allList: [],
                modelList:[], //所有的modelList
                apiName: "/selection/selectiondelaydata", //实时调接口api
                loading: false
            }
        },
        computed:{
            //获取vuex的传值
            ...mapState({
                "filterListItem": state=>state.filterListItem,
                "modelListCopy": state=>state.modelListCopy,
                "needcontrast": state=>state.needcontrast
            }),
            eachWidth(){
                return parseInt(100/this.searchList.length) + "%";
            }
        },
        methods:{
            ...mapMutations({
                changeModelListCopy:"changeModelListCopy",
                changeContrastDgVisible:"changeContrastDgVisible"
            }),
            //搜索条件分类,并动态生成v-model(核心方法)
            modifySelectType(){
                //console.log(this.searchList);
                this.searchList.forEach((item, index) => { //动态生成v-model
                    let obj = {"id":item.id, "name":item.inputname, "title":item.Title, "required":item.required, "type":item.inputtype,
                        "listLength":item.selectitemlist.length, "model":[]};

                    let inputType = item.inputtype; //表单样式类型
                    let selectOptions = item.selectitemlist; //select条件

                    if(inputType==1){ //input框

                    }else if(inputType==2){ //radio单选
                        
                    }else if(inputType==3){ //checkbox多选
                        obj.checkAll = false;
                        obj.isIndeterminate = false;

                    }
                    //4:select单选; 5:select多选; 6:自动补全单选(本地&网络); 7:自动补全多选(本地&网络)
                    else if(inputType==4 || inputType==5 || inputType==6 || inputType==7){
                        obj.options = selectOptions.length? selectOptions.slice(0,20): [];

                    }

                    this.modelList.push(obj);

                    //给父组件设置childrenId, childrenName
                    if(item.loadtype!=1 && item.parentid && item.parentname){
                        this.$nextTick(()=>{
                            let parentRef = this.$refs[item.parentname+item.parentid];

                            $(parentRef[0].$el).attr("childId", item.id);
                            $(parentRef[0].$el).attr("childName", item.inputname);
                            $(parentRef[0].$el).attr("childIndex", index);
                        });
                    } 

                });
            },
            //拼接参数方法
            setParas(){
                let unfilledList = []; //未填写的必填项

                this.modelList.forEach((obj)=>{
                    let inputType = obj.inputtype; //表单控件类型
                    let inputModel = obj.model; //表单model
                    let paramsName = (obj.name).toLowerCase(); //作为参数名(全部转为小写)

                    if(!inputModel.length>0){
                        if(obj.required) unfilledList.push(obj.title); //判断必填项
                        this.params[paramsName] = "";

                    }else{
                        if(inputType==1){ //普通input
                            this.params[paramsName] = inputModel[0];

                        }else{
                            let tempArr = [];

                            inputModel.forEach((item)=>{ //item:{value:"",name:""}
                                tempArr.push( JSON.parse(item).value );
                            });
                            this.params[paramsName] = tempArr.join(","); //数组转字符串并添加到参数中
                        }
                    }
                });
                
                this.$emit("hasUnfilled", unfilledList);

                if(unfilledList.length==0){ //如果没有未填项目
                    //console.log(this.params);
                    this.$emit("paramsChanged", this.params); //向父组件传递参数
                }          
            },
            /**********/
            //自动补全单选
            querySearchLocal(queryString, callback, list){
                //console.log(list);
                let result = queryString? list.filter((o)=>{ return o.name.indexOf(queryString)!=-1 }): list.slice(0,20);
                result = result.map((o)=>{return {"value": o.name}});
                callback(result);
            },
            //实时调接口方法(自动补全单选)
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
            //重置options(自动补全多选相关)
            remoteMethodLocal(queryString, index, lists) { //lists:原始数据, index:autocomplete.models的索引
                let result = queryString? lists.filter((o)=>{ return o.name.indexOf(queryString)!=-1 }): lists.slice(0,20);
                this.modelList[index].options = result;
            },
            //重置options(自动补全多选相关)
            remoteMethodAsync(queryString, index, selectionId, ownName, parentName){
                if(queryString){
                    let params = {};
                    params.selectionId = selectionId;

                    if(parentName){
                        params[parentName] = queryString;
                    }else{
                        params[ownName] = queryString; //补充参数
                    }

                    this.$dataPost(this.apiName, params, (data)=>{
                        let result = data.selectitemlist;
                        this.modelList[index].options = result;
                    });
                }else{
                    this.modelList[index].options = [];
                }      
            },
            //根据index删除对应的model内的项
            delModel(model, index, type){
                //console.log(model, index, type);
                let allModel = this.modelList[index].model;
                allModel.forEach((item,i)=>{
                    if(item==model){
                        this.modelList[index].model.splice(i, 1);
                    }
                });
            },
            //全选半选不选判断
            checkALLFn(index, selectLength){
                let checkedCount = this.modelList[index].model.length;
                //this.modelList[index].checkAll = (checkedCount==item.selectitemlist.legnth);
                this.modelList[index].checkAll = checkedCount? true: false;
                this.modelList[index].isIndeterminate = checkedCount>0 && checkedCount<selectLength;
            },
            //多级select全选
            multiCheckAll(value, index){
                let level1list = Enumerable.from(this.modelList[index].options).where((o)=>{return o.name==value;}).toArray();
                let level2list = level1list[0].options;
                if(this.modelList[index].model.length < level2list.length){ //全选
                    this.modelList[index].model = Enumerable.from(level2list).select(o=>o.value).toArray();
                }else{ //取消全选
                    this.modelList[index].model = [];
                }
            },
            transSearchList(list){
                var newArr = [];
                list.forEach((item)=>{
                    newArr.push( JSON.stringify(item) )
                });
                return newArr;
            },
            tempChange(){
                alert("事件触发");
            },
            //select两个联动
            linkSelect(id, name, index){
                //判断是否是别的元素的子集
                this.$nextTick(()=>{
                    let ownRef = this.$refs[name+id];

                    let childId = $(ownRef[0].$el).attr("childId");
                    let childName = $(ownRef[0].$el).attr("childName");
                    let childIndex = $(ownRef[0].$el).attr("childIndex");

                    //如果有childId, childName, 调用接口改child
                    if(childName && childId){
                        //清空child
                        this.modelList[childIndex].options = [];
                        this.modelList[childIndex].model = []; 
                        //获取自己被选中的值
                        let ownModel = this.modelList[index].model;
                        let ownValue = "";

                        if(typeof ownModel == "object"){ //["{"cname":null,"name":"CCU","value":"CCU"}", "{"cname":null,"name":"RICU","value":"RICU"}"]
                            let temp = [];
                            ownModel.forEach(item=>{
                                temp.push(JSON.parse(item).value);
                            });
                            ownValue = temp.join(",");
                        }else{ //"{"cname":null,"name":"CCU","value":"CCU"}"
                            ownValue = JSON.parse(ownModel).value;
                        }
                        
                        //调用接口，给child绑定新字典
                        let params = {};
                        params.hospitalid = this.$getSession("currentHosId");
                        params.selectionId = childId;
                        params[name] = ownValue; //补充参数

                        this.$dataPostCommon("/selection/delaydata", params, (data)=>{
                            let result = data.selectitemlist;
                            this.modelList[childIndex].options = result;
                        });
                    }
                    
                })
            },
            showOtherfilterDg(){
                this.$emit('update:visible', true); //改变父级的值
            }
        },
        watch:{
            searchList:{
                handler(newVal, oldVal){
                    //console.log(newVal, oldVal);
                    this.params = {}; //清空
                    this.modelList = []; //清空
                    this.modifySelectType();
                    
                    let usefulModelList = Enumerable.from(this.modelListCopy).where((o)=>{return o.model.length>0;}).toArray();
                    this.$emit("filterListChange", usefulModelList.length); //向父组件传递已选择项的长度

                    //console.log(usefulModelList);
                    // usefulModelList.forEach((mitem)=>{ //循环已经选择的项
                    //     newVal.forEach((sitem, index)=>{ //循环新条件
                    //         if(sitem.inputname==mitem.name){
                    //             this.modelList[index].model = mitem.model; //设置新model默认值
                    //             return true;
                    //         }
                    //     });
                    // }) 
                }
            },
            modelList:{
                deep:true,
                handler(newVal, oldVal){
                    this.changeModelListCopy(newVal); //改变vuex值,向filterList传值
                    this.setParas();
                }
            },
            filterListItem:{
                handler(newVal, oldVal){
                    let model=newVal.model, index=newVal.index, type=newVal.type, selectLength=newVal.length; 
                    //console.log(newVal);
                    //console.log(model, index, type);
                    this.delModel(model, index, type);
                    //取消全选状态
                    if(type==2)
                        this.checkALLFn(index, selectLength);
                }
            }
        },
        components:{
            filterInputItem
        }

	}
</script>

<style lang="scss" type="text/css">
    .filter_body{
        background: #f8f8f8;
        padding: 10px 0;

        .filter_moudle{
            float: left;
        }
        .filterInput_item{
            //float: left;
            display: table-cell;
            border-right: 1px dashed #ddd;
            padding: 0 8px;
        }

        //重置样式
        .el-input{
            margin-top: 4px !important;
            //margin-left: 4px !important;
        }
        .filter_checkbox.is-bordered{
            margin: 4px !important;
            background-color: #fff; 
        }
        .filter_checkbox .el-checkbox__inner{
            display: none;
        }
        .filter_radio.is-bordered{
            margin: 4px !important;
            background-color: #fff; 
        }
        .filter_radio .el-radio__inner{
            display: none;
        }

        // //重置样式
        // .el-input{
        //     margin-top: 4px !important;
        //     //margin-left: 4px !important;
        // }
        // .el-checkbox-button{
        //     margin: 4px;
        // }
        // .el-checkbox-button__inner{
        //     border-left: 1px solid #dcdfe6;
        // }
        // .el-radio-button{
        //     margin: 4px;
        // }
        // .el-radio-button__inner{
        //     border-left: 1px solid #dcdfe6;
        // }
    }
</style>