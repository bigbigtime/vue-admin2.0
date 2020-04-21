<template>
    <div>
        <el-row :gutter="14">
            <el-col :span="4">
                <div class="label-wrap category">
                    <label for="">分类：</label>
                    <div class="warp-content">
                        <el-select v-model="category_value" placeholder="请选择" style="width: 100%;">
                            <el-option
                                v-for="item in options.category"
                                :key="item.id"
                                :label="item.category_name"
                                :value="item.id">
                            </el-option>
                        </el-select>
                    </div>
                </div>
            </el-col>
            <el-col :span="7">
                <div class="label-wrap date">
                    <label for="">日期：&nbsp;&nbsp;</label>
                    <div class="warp-content">
                        <el-date-picker
                            style="width: 100%;"
                            v-model="date_value"
                            type="datetimerange"
                            format="yyyy 年 MM 月 dd 日"
                            value-format="yyyy-MM-dd HH:mm:ss"
                            align="right"
                            start-placeholder="开始日期"
                            end-placeholder="结束日期"
                            :default-time="['12:00:00', '08:00:00']">
                        </el-date-picker>
                    </div>
                </div>
            </el-col>
            <el-col :span="3">
                <div class="label-wrap key-work">
                    <label for="">关键字：&nbsp;&nbsp;</label>
                    <div class="warp-content">
                        <SelectVue :config="data.configOption" :selectData.sync="selectData"/>
                    </div>
                </div>
            </el-col>
            <el-col :span="3">
                <el-input v-model="search_keyWork" placeholder="请输入内容" style="width: 100%;"></el-input>
            </el-col>
            <el-col :span="2">
                <el-button type="danger" style="width: 100%;" @click="search()">搜索</el-button>
            </el-col>
            <el-col :span="3">&nbsp;</el-col>
            <el-col :span="2">
                <el-button type="danger" class="pull-right" style="width: 100%;" @click="dialog_info = true"  v-if="btnPerm('info:add')">新增</el-button>
            </el-col>
        </el-row>
        <!-- 表格数据 -->
        <div class="black-space-30"></div>
        <TableVue ref="infoTable" :config="configTable" :tableRow.sync="tableRow">
            <template v-slot:operation="slotData">
                <el-button type="danger" size="mini" v-btnPerm="'info:del'" class="hiden-button">自定义指令测试</el-button>
                <el-button type="danger" size="mini" @click="deleteItem(slotData.data.id)" v-btnPerm="'info:del'" class="hiden-button">删除</el-button>
                <el-button type="success" size="mini" @click="editInfo(scope.row.id)" v-btnPerm="'info:edit'" class="hiden-button">编辑</el-button>
                <el-button type="success" size="mini" @click="detailed(scope.row)" v-btnPerm="'info:detailed'" class="hiden-button">编辑详情</el-button>
            </template>
        </TableVue>
        <!--新增弹窗-->
        <DialogInfo :flag.sync="dialog_info" :category="options.category" />
        <!--修必弹窗-->
        <DialogEditInfo :flag.sync="dialog_info_edit" :id="infoId" :category="options.category" />
    </div>
</template>
<script>
import { GetCategory, DeleteInfo } from "@/api/news";
import DialogInfo from "./dialog/info";
import DialogEditInfo from "./dialog/edit";
import { timestampToTime } from "@/utils/common";
// 组件
import SelectVue from "@c/Select";
import TableVue from "@c/Table";
export default {
    name: 'infoIndex',
    components: { DialogInfo, DialogEditInfo, SelectVue, TableVue },
    data(){
        return {
            // table组件配置参数
            configTable: {
                // 表头
                tHead: [
                    { 
                        label: "标题",
                        field: "title",
                        width: 600
                    },
                    { 
                        label: "类型",
                        field: "categoryId",
                        width: 120
                    },
                    { 
                        label: "日期",
                        field: "createDate",
                        width: 237
                    },
                    { 
                        label: "操作",
                        columnType: "slot",
                        slotName: "operation"
                    }
                ],
                // 请求接口URL
                requestData: {
                    url: "getInfoList",
                    data: {
                        pageNumber: 1,
                        pageSize: 10
                    }
                }
            },
            tableRow: {},
            data: {
                configOption: {
                    init: ["id", "title"]
                }
            },
            dialog_info: false,
            dialog_info_edit: false,
            search_key: 'id',
            // 下接菜单的数据
            selectData: {},
            category_value: '',
            date_value: '',
            search_keyWork: '',
            total: 0,
            deleteInfoId: '',
            infoId: '',
            // loading
            loadingData: false,
            options: {
                category: []
            },
            // 搜索关键字
            search_option: [
                { value: "id", label: "ID"},
                { value: "title", label: "标题"},
            ],
            // 页码
            page: {
                pageNumber: 1,
                pageSize: 10
            },
            // 表格数据
            table_data: []
        }
    },
    mounted(){
        // 获取分类
        this.getInfoCategory();
    },
    methods: {
        /**
         * 搜索
         */
        search(){
            let requesttData = {
                [this.selectData.value] : this.search_keyWork
            }
            this.$refs.infoTable.paramsLoadData(requesttData);
        },
        editInfo(id){
            infoId.value = id;
            dialog_info_edit.value = true;
        },
        /**
         * 详情页
         */
        detailed(data){
            // 预先存值
            // this.$store.commit("infoDetailed/SET_ID", data.id);
            // this.$store.commit("infoDetailed/SET_TITLE", data.title);
            this.$store.commit("infoDetailed/UPDATE_STATE_VALUE", {
                id: {
                    value: data.id,
                    sessionKey: "infoId",
                    session: true
                },
                title: {
                    value: data.title,
                    sessionKey: "infoTitle",
                    session: true
                }
            });
            // 跳转页面
            this.$router.push({
                name: "InfoDetailed",
                params: {
                    id: data.id, 
                    title: data.title
                }
            })
        },
        /**
         * 删除数据
         */
        deleteItem(id){
            this.deleteInfoId = [id];
            this.confirm({
                content: "确认删除当前信息，确认后将无法恢复！！",
                tip: "警告",
                fn: this.confirmDelete
            })
        },
        deleteAll(){
            if(!deleteInfoId.value || deleteInfoId.value.length == 0) {
                this.$message({
                    message: "请选择要删除的数据！！",
                    type: "error"
                })
                return false;
            }
            this.confirm({
                content: "确认删除选择的数据，确认后将无法恢复！",
                tip: "警告",
                fn: this.confirmDelete
            })
        },
      
        confirmDelete(value){
            DeleteInfo({id: this.deleteInfoId}).then(response => {
                this.deleteInfoId = '';
                this.$message({
                    message: '删除成功！！',
                    type: 'success'
                })
                // 刷新数据
                this.$refs.infoTable.refreshData();
            })
        },
        getInfoCategory(){
            this.$store.dispatch('common/getInfoCategory').then(response => {
                this.options.category = response
            })
        },
        toData(row, column, cellValue, index){
            return timestampToTime(row.createDate);
        },
        toCategory(row){
            // 调用一个函数，返回一个新的值，替换原始值
            let categoryId = row.categoryId;
            let categoryData = this.options.category.filter(item => item.id == categoryId)[0];
            if(!categoryData) { return false; }
            return categoryData.category_name;
        },
        handleSelectionChange(val){
            let id = val.map(item => item.id);
            deleteInfoId.value = id
        }
    }
}
</script>
<style lang="scss" scoped>
@import "../../styles/config.scss";
.label-wrap {
    &.category { @include labelDom(left, 60, 40); }
    &.date { @include labelDom(right, 93, 40); }
    &.key-work { @include labelDom(right, 99, 40); }
}
</style>
<style>
button.hiden-button { display: none; }
button.show-button { display: inline-block; }
</style>