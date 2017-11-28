<template>
    <div class="wrap" id="search">
        <div class="portlet-title">
            <div class="caption">
                <i class="fa fa-list"></i> 维修配件
            </div>
            <div class="col-md-2 text-right">
                <el-cascader placeholder="选择操作仓库" expand-trigger="hover" :options="option" @change="chooseStoreHouse" v-model="key_do">
                </el-cascader>

            </div>
            <div class="col-md-2 text-right">
                <el-cascader placeholder="选择资产类型" expand-trigger="hover" :options="items" @change="chooseItems">
                </el-cascader>
            </div>
            <div class="actions"><button class="btn green btn-outline uppercase" v-on:click="submit">
                <i class="fa fa-plus" ></i>提交</button>
                <!-- <el-button type="primary">主要按钮</el-button> -->
            </div>
        </div>
        <span v-if="key_do[0] !='reclaim'">
            <table class="table table-bordered table-striped">
                <thead>
                    <tr>
                        <th width="200">编号</th>
                        <th width="200">产品名称</th>
                        <th width="200">现库存总量</th>
                        <th width="200">预计库存量</th>
                        <th width="200">加减数量</th>
                        <th width="200">库存剩余量</th>
                        <th width="200">计量单位</th>
                        <th width="200">描述</th>
                        <!-- <th width="200">删除</th> -->
                    </tr>
                </thead>
                    
                <tbody>
                    <tr v-for="(value ,key ,index ) in goods">
                        <td>{{ value[0].asset_uuid.split('-')[1] }}</td>
                        <td>{{ value[0].title }}</td>
                        <td>{{ value[0].store_num }}</td>
                        <td>{{ value[0].predict_num }}</td>
                        
                        <td>
                            <span v-if="key_do['0'] == 'in'">
                                <el-input-number v-model="value[0].num_self_initialization" @change="handleChange" :min="0" :max="999"></el-input-number>
                            </span>
                            <span v-if="key_do['0'] == 'out'">
                                <el-input-number v-model="value[0].num_self_initialization" @change="handleChange" :min="0" :max="value[0].store_num"></el-input-number>
                            </span>
                        </td>
                        <td v-if="key_do['0'] == 'in'">{{value[0].store_num+value[0].num_self_initialization}}</td>
                        <td v-if="key_do['0'] == 'out'">{{value[0].store_num-value[0].num_self_initialization}}</td>
                        <td>{{value[0].unit}}</td>
                        <td>{{value[0].description}}</td>
                        <!-- <td>
                            <i class="glyphicon glyphicon-minus btn" v-on:click="delect"></i>
                        </td> -->
                    </tr>

                </tbody>
            </table>
        </span>
        <span v-if="key_do[0] =='reclaim'">
            <table class="table table-bordered table-striped">
                <thead>
                    <tr>
                        <th width="200">维修单号</th>
                        <th width="200">配件编号</th>
                        <th width="200">产品名称</th>
                        <th width="200">待回收数量</th>
                        <th width="200">回收数量</th>
                        <th width="200">剩余数量</th>
                        <th width="200">描述</th>
                    </tr>
                </thead>
                <!-- <tbody>
                    <tr>
                        <td></td>
                        <td></td>
                        <td></td>
                        <td></td>
                        <td>
                            <span >
                                <el-input-number ></el-input-number>
                            </span>
                            <span>
                                <el-input-number ></el-input-number>
                            </span>
                        </td>
                        <td></td>
                        <td>
                            <el-input>
                            </el-input>
                        </td>
                    </tr>
                </tbody> -->
            </table>
        </span>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                items:[],//这个是暂时存储 选择仓库-后的内容
                goods:[],//这个是当前要操作的项目的暂存器
                key_do:[],//操作类型
                user_id:'',//用户id
                storehouse_id:'',//仓库id
                boolean_admin:'',
                onOff:true,
                option: [
                    {
                    value: "in",
                    label: "入库"
                    },
                    {
                    value: "out",
                    label: "出库"
                    }
                ],
            };
        },
        //items 是各个项目，key_do是本次操作的关键字（in/out）
        // 暂存器 里 ：
        mounted: function() {
            this.getStorehouse_id();
            this.getUser_id();
        },
        methods: {
            //获取登录用户管理的仓库uuid
            getStorehouse_id() {
                let _this = this;
                let path = window.location.href;
                path = path.match(/(\S*)admin/)[1];
                this.$http.post(path + "admin/api/storehouse").then(function(res) {
                    _this.storehouse_id = res.data.data;
                });


            },
            //获取登录用户的uuid
            getUser_id() {
                let _this = this;
                let path = window.location.href;
                path = path.match(/(\S*)admin/)[1];
                this.$http.post(path + "admin/api/user").then(function(res) {
                    _this.user_id = res.data.data;
                    //console.log(_this.user_uuid);
                    if (_this.user_id === "admin") {
                        _this.boolean_admin = true;
                    } else {
                        _this.boolean_admin = false;
                    }
                });
                // console.log(this);
            },
            //切换操作类型：出入库
            //  1.获取本次暂存器里内容：（1）空--不操作    （2）非空--判断是什么操作，根据操作存储数据在localststorage（分操作）
            // 2.清除这个暂存器，把localststorage里有的数据写入（分操作）
            //点击选择资产类型    （1）是否这个添加的资产已经在暂存器存在？存在拒绝添加，不存在就添加--
            //再次切换类型。类似逻辑
            //点击提交，是否提交？ 是--弹窗消失--（成功或者失败，--成功就清除本地local-清除暂存器。。失败的话提示失败，保留 本地和暂存器数据） 不提交--弹窗消失..


            chooseStoreHouse(){
                // console.log(this.key_do);
                this.goods.length = 0 ;
                this.items.length = 0 ;
                // console.log(this.boolean_admin)
                // console.log(this.key_do) in/out
                let _this = this;
                let path = window.location.href.match(/(\S*)admin/)[1];
                if(this.key_do['0']==="reclaim"){

                }
                if(this.key_do['0'] === 'out' || this.key_do['0'] === 'in'){
                    this.$http.post(path + "admin/api/category", {
                        asset_type: "MaintainComponent"
                    }).then(function(response) {
                        // console.log(response)
                        // _this.consumble_type = response.data.data;
                        _this.items.length = 0;
                        _this.items.push(response.data.data['0']);
                        // console.log(response.data.data);
                    });
                }
            },
            chooseItems(value){
                //获取到是哪个物品，编号唯一   value.reverse()[0]
                let _this = this;
                let path = window.location.href;
                path = path.match(/(\S*)admin/)[1];
                if(this.user_id ==='admin'){
                    if(this.key_do['0']==='in'){
                        _this.onOff = true;
                        this.$http.post(path + "admin/api/goods", {
                        asset_type: "MaintainComponent",
                        parent: value[0],
                        category_uuid: value[1],
                        asset_uuid: value[2]
                        }).then(function(res){
                            let resID = res.data.data[0].asset_uuid;
                            _this.goods.every((e)=>{
                                if(e[0].asset_uuid != resID){
                                    return _this.onOff = true;
                                }else{
                                    return _this.onOff = false;
                                }
                            })
                            // console.log(_this.onOff)
                            if(_this.onOff){
                                res.data.data[0].num_self_initialization = 0 ;
                                _this.goods.push(res.data.data);
                            }else{
                                alert('请不要重复添加')
                            }
                        })
                    }else if(this.key_do['0'] ==='out'){
                        _this.onOff = true;
                        this.$http.post(path + "admin/api/goods", {
                        asset_type: "MaintainComponent",
                        parent: value[0],
                        category_uuid: value[1],
                        asset_uuid: value[2]
                        }).then(function(res){
                            // console.log(_this.goods)//池子
                            // console.log(res.data.data)//鱼
                            let resID = res.data.data[0].asset_uuid;
                            _this.goods.every((e)=>{
                                if(e[0].asset_uuid != resID){
                                    return _this.onOff = true;
                                }else{
                                    return _this.onOff = false;
                                }
                            })

                            if(_this.onOff){
                                if(res.data.data[0].store_num<res.data.data[0].predict_num){
                                    alert('现库存总量数量小于预计库存总量，请补充库存！')
                                }
                                res.data.data[0].num_self_initialization = 0 ;
                                _this.goods.push(res.data.data);
                            }else{
                                alert('请不要重复添加')
                            }

                        })

                    }
                }
            },
            handleChange(value){
                // console.log(value);
                // console.log(this.goods)
            },
            submit:function(){
                let _this = this;
                let path = window.location.href;
                path = path.match(/(\S*)admin/)[1];
                if(this.goods.length == 0){
                    alert('请添加项目！')
                }else{
                    if(!this.user_id){
                    alert('非管理员禁止提交！')
                    }else{
                        if(!this.storehouse_id){
                            alert('请选择要操作的仓库！')
                        }else{
                            function checkNum(num) {
                                return num == 0
                            }
                            let arr = [];
                            this.goods.forEach((e)=>{
                                arr.push(e[0].num_self_initialization)
                            })
                            if(arr.every((e)=>{return e!=0})){
                                //规则
                                // console.log(this.key_do);
                                // this.$http.post(path + "admin/api/asset-order", {
                                //     user_uuid:this.user_id,
                                //     storehouse_uuid:this.storehouse_id,
                                //     type_uuid:'MaintainComponent',
                                //     type:_this.key_do,
                                //     detail:[
                                //         {
                                //             asset_uuid:'maintainComponent-00001',
                                //             title:'八字阀',
                                //             original_amount:'store_num',  85
                                //             amount:'num_self_initialization',  1
                                //             current_amount:'',  85 - 1 或者  +1
                                //             description:'车辆配件',
                                //         }
                                //     ]
                                // })
                                // console.log(this.goods);
                                //拼 API-----应该可以用解构的吧？数组中的对象取一部分，重新命名其中一部分
                                let arr_submit_in = [];
                                this.goods.forEach((e,i)=>{
                                    let obj = {};
                                    obj.asset_uuid = e[0].asset_uuid;
                                    obj.title = e[0].title;
                                    obj.original_amount = e[0].store_num;
                                    obj.amount = e[0].num_self_initialization;
                                    obj.current_amount = e[0].store_num +e[0].num_self_initialization;
                                    obj.description = e[0].description;
                                    arr_submit_in.push(obj);
                                })
                                let arr_submit_out = [];
                                this.goods.forEach((e,i)=>{
                                    let obj = {};
                                    obj.asset_uuid = e[0].asset_uuid;
                                    obj.title = e[0].title;
                                    obj.original_amount = e[0].store_num;
                                    obj.amount = e[0].num_self_initialization;
                                    obj.current_amount = e[0].store_num - e[0].num_self_initialization;
                                    obj.description = e[0].description;
                                    arr_submit_out.push(obj);
                                })
                                //拼法
                                // // let obj = {};
                                // // obj.asset_uuid = this.goods[0][0].asset_uuid;
                                // // obj.title = this.goods[0][0].title;
                                // // obj.original_amount = this.goods[0][0].store_num;
                                // // obj.amount = this.goods[0][0].num_self_initialization;
                                // // obj.current_amount = this.goods[0][0].store_num +this.goods[0][0].num_self_initialization;
                                // // obj.description = this.goods[0][0].description;

                                // console.log(arr_submit)
                                // this.goods.forEach((e,i)=>{
                                //     // console.log(e[0].asset_uuid);
                                //     // arr_submit[i].asset_uuid = e[0].asset_uuid
                                // })
                                // // console.log(arr_submit)
                                if(this.key_do['0'] == 'in'){
                                    this.$http.post(path + "admin/api/asset-order", {
                                    user_uuid:this.user_id,
                                    storehouse_uuid:this.storehouse_id,
                                    type_uuid:'MaintainComponent',
                                    asset_type:'MaintainComponent',
                                    type:'in',
                                    detail:arr_submit_in
                                    }).then(function(res) {
                                        // console.log(res.data)
                                        if(res.data.success){
                                            _this.goods= [];
                                            _this.onOff = true;
                                            alert('提交成功！')
                                        }else{
                                            alert('提交失败！')
                                        }
                                    })
                                }else if(this.key_do['0'] == 'out'){
                                    this.$http.post(path + "admin/api/asset-order", {
                                    user_uuid:this.user_id,
                                    storehouse_uuid:this.storehouse_id,
                                    type_uuid:'MaintainComponent',
                                    asset_type:'MaintainComponent',
                                    type:'out',
                                    detail:arr_submit_out
                                    }).then(function(res) {
                                        if(res.data.success){
                                            _this.goods = [];
                                            _this.onOff = true;
                                            alert('提交成功！')
                                        }else{
                                            alert('提交失败！')
                                        }
                                    })
                                }
                            }else{
                                alert('一项或多项选择数量为零！')
                            }
                        }
                    }
                }
            },
            // delect(aaa){
            //     console.log(aaa.target)
            // }
        },
        watch:{

        },
    };

</script>

<style>
    .searchList {
        position: absolute;
        right: 144px;
        top: 149px;
        width: 210px;
    }
    .portlet-title {
        min-height: 48px;
        padding: 0;
        border-bottom: 1px solid #eee;
        margin-bottom: 10px;
    }
    .text-right {
        margin-left: 30px;
    }
    .caption {
        line-height: 18px;
        font-size: 18px;
        color: #32c5d2 !important;
        display: inline-block;
        padding: 10px 0;
        float: left;
    }
    .fa-list {
        font-size: 15px;
        font-weight: 300;
        color: #32c5d2;
        float: left;
        display: inline-block;
        margin-right: 5px;
        margin-top: 2px;
    }
    .actions {
        float: right;
        display: inline-block;
        padding: 6px 0 14px;
    }
    .actions>a {
        padding: 4px 10px;
        font-size: 13px;
        line-height: 1.5;
    }
    .btn.btn-outline.green {
        border-color: #32c5d2;
        color: #32c5d2;
    }
    .form-control,
    .btn {
        box-shadow: none !important;
    }
    .form-control {
        outline: 0 !important;
        border: 1px solid #c2cad8;
        color: #4d6b8a !important;
    }
    .form-inline input:focus {
        border-color: #4d6b8a;
    }
    .btn.green:not(.btn-outline) {
        color: #fff;
        background-color: #32c5d2;
        border-color: #32c5d2;
    }
    .btn:not(.btn-sm):not(.btn-lg) {
        line-height: 1.44;
    }
    #index ul li {
        width: 200px;
        border: 1px #000 solid;
    }
    #index ul li a {
        display: block;
        font-size: 12px;
        line-height: 40px;
        color: #000;
        font-weight: bold;
        text-align: left;
        padding-left: 4px;
    }
    ul.open {
        display: block;
    }
    ul.on {
        display: block;
    }
    .test {
        display: none;
    }
    .btn{
        margin-left: 10px;
    }
</style>

