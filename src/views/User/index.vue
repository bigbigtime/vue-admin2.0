<template>
    <div>
        <el-row>
            <el-col :span="20">
                <div class="label-wrap">
                    <label>关键字：</label>
                    <div class="warp-content">
                        <el-row :gutter="16">
                            <el-col :span="3">
                                <SelectVue :config="configOption" :selectData.sync="selectData" />
                            </el-col>
                            <el-col :span="5">
                                <el-input v-model="key_word" placeholder="请输入搜索的关键字"></el-input>
                            </el-col>
                            <el-col :span="15">
                                <el-button type="danger" @click="search">搜索</el-button>
                            </el-col>
                        </el-row>
                    </div>
                </div>
            </el-col>
            <el-col :span="4">
                <el-button type="danger" class="pull-right" @click="handlerAdd">添加用户</el-button>
            </el-col>
        </el-row>
        <div class="black-space-30"></div>
        <TableVue ref="userTable" :config="configTable" :tableRow.sync="tableRow">
            <!--插槽-->
            <template v-slot:status="slotData">
                <el-switch @change="handlerSwitch(slotData.data)" v-model="slotData.data.status" active-value="2"  inactive-value="1" active-color="#13ce66" inactive-color="#ff4949"></el-switch>
            </template>
            <template v-slot:operation="slotData">
                <el-button size="small" type="danger" @click="handlerDel(slotData.data)">删除</el-button>
                <el-button size="small" type="success" @click="handlerEdit(slotData.data)">编辑</el-button>
            </template>
            <template v-slot:tableFooterLeft>
                <el-button size="small" @click="handlerBatchDel()">批量删除</el-button>
            </template>
            <!--插槽-->
        </TableVue>
        <!--子组件-->
        <DialogAdd :flag.sync="dialog_add" :editData="editData" @componentCallback="componentCallbackFun" />
    </div>
</template>
<script>
import { reactive, ref, watch, onMounted, provide } from '@vue/composition-api';
import { UserDel, UserActives } from "@/api/user";
// 组件
import SelectVue from "@c/Select";
import TableVue from "@c/Table";
import DialogAdd from "./dialog/add";
// 3.0抽离的方法
import { global } from "@/utils/global_V3.0";
export default {
    name: 'userIndex',
    components: { SelectVue, TableVue, DialogAdd },
    data(){
        return {
            userTable: null,
            tableRow: {},
            cityPickerData: {},
            dialog_add: false,
            dialog_edit: false,
            editData: {},
            configOption: {
                init: ["truename", "phone"]
            },
            // 下接菜单的数据
            selectData: {},
            // 搜索关键字
            key_word: "",
            // 阻止状态
            updateUserStatusFlag: false,
            // table组件配置参数
            configTable: {
                // 表头
                tHead: [
                    { 
                        label: "邮箱/用户名",
                        field: "username",
                        width: 200
                    },
                    { 
                        label: "真实姓名",
                        field: "truename",
                        width: 120
                    },
                    { 
                        label: "手机号",
                        field: "phone"
                    },
                    { 
                        label: "地区",
                        field: "region"
                    },
                    { 
                        label: "角色",
                        field: "role"
                    },
                    { 
                        label: "禁启用状态",
                        field: "status",
                        columnType: "slot",
                        slotName: "status"
                    },
                    { 
                        label: "操作",
                        columnType: "slot",
                        slotName: "operation"
                    }
                ],
                // 请求接口URL
                requestData: {
                    url: "getUserList",
                    data: {
                        pageNumber: 1,
                        pageSize: 10
                    }
                }
            }
        }
    },
    methods: {
        componentCallbackFun(params){ params.function && this[params.function](params.data); },
        /**
         * 搜索
         */
        search(){
            let requesttData = {
                [this.selectData.value] : this.key_word
            }
            this.$refs.userTable.paramsLoadData(requesttData);
        },
        /**
         * 批量删除
         */
        handlerBatchDel(){
            let userId = this.tableRow.idItem
            if(!userId || userId.length === 0) {
                this.$message({
                    message: "请勾选需要删除的用户！！",
                    type: "error"
                })
                return false;
            }
            this.confirm({
                content: "确认删除当前信息，确认后将无法恢复！！",
                tip: "警告",
                fn: this.userDelete
            })
        },
        /**
         * 删除用户
         */
        userDelete(){
            UserDel({ id: this.tableRow.idItem }).then(response => {
                // 其中一种写法
                // refs.userTable.refreshData()
                // 第二种写法
                this.refreshData()
            })
        },
        /**
         * 刷新data
         */
        refreshData(){
            this.$refs.userTable.refreshData();
        },
        /**
         * 单条数据删除
         */
        handlerDel(params){
            this.tableRow.idItem = [params.id];
            this.confirm({
                content: "确认删除当前信息，确认后将无法恢复！！",
                tip: "警告",
                fn: this.userDelete
            })
        },
        /**
         * 添加用户
         */
        handlerAdd(){
            this.dialog_add = true;
            // 子组件赋值
            this.editData = Object.assign({});
        },
        /**
         * 编辑
         */
        handlerEdit(params){
            this.dialog_add = true;
            // 子组件赋值
            this.editData = Object.assign({}, params);
        },
        /**
         * 修改用户状态
         */
        handlerSwitch(params){
            if(this.updateUserStatusFlag) { return false }
            this.updateUserStatusFlag = true
            UserActives({ id: params.id, status: params.status }).then(response => {
                this.$message({
                    message: response.data.message,
                    type: "success"
                })
                this.updateUserStatusFlag = !this.updateUserStatusFlag
            }).catch(error => {
                this.updateUserStatusFlag = !this.updateUserStatusFlag
            })
        }
    }
}
</script>
<style lang="scss" scoped>
@import "../../styles/config.scss";
.label-wrap {
    @include labelDom(left, 60, 40);
}
</style>