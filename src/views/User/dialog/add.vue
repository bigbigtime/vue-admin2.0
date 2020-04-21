<template>
    <el-dialog title="新增" :visible.sync="dialog_info_flag" @close="close" width="700px" @opened="openDialog">
        <el-form :model="form" :rules="rules" ref="addInfoForm">
            <el-form-item label="用户名：" :label-width="formLabelWidth" prop="username">
                <el-input v-model="form.username" placeholder="请输入邮箱"></el-input>
            </el-form-item>
            <el-form-item label="密码：" :label-width="formLabelWidth" prop="password">
                <el-input type="password" v-model="form.password" placeholder="请输入6~20数字+字母"></el-input>
            </el-form-item>
            <el-form-item label="姓名：" :label-width="formLabelWidth" prop="truename">
                <el-input v-model="form.truename" placeholder="请输入真实姓名"></el-input>
            </el-form-item>
            <el-form-item label="手机号：" :label-width="formLabelWidth" prop="phone">
                <el-input v-model.number="form.phone" placeholder="请输入手机号"></el-input>
            </el-form-item>
            <el-form-item label="地区：" :label-width="formLabelWidth" prop="region">
                <CityPicker :cityPickerData.sync="cityPickerData" />
            </el-form-item>
            <el-form-item label="是否启用：" :label-width="formLabelWidth" prop="status">
                <el-radio v-model="form.status" label="1">禁用</el-radio>
                <el-radio v-model="form.status" label="2">启用</el-radio>
            </el-form-item>
            <el-form-item label="角色：" :label-width="formLabelWidth" prop="role">
                <el-checkbox-group v-model="form.role">
                    <el-checkbox v-for="item in roleItem" :key="item.role" :label="item.role">{{ item.name }}</el-checkbox>
                </el-checkbox-group>
            </el-form-item>
            <el-form-item label="按钮：" :label-width="formLabelWidth">
                <template v-if="btnPerm.length > 0">
                    <div v-for="item in btnPerm" :key="item.id">
                        <h4>{{ item.name }}</h4>
                        <template v-if="item.perm && item.perm.length > 0">
                            <el-checkbox-group v-model="form.btnPerm">
                                <el-checkbox v-for="buttons in item.perm" :key="buttons.value" :label="buttons.value">{{ buttons.name }}</el-checkbox>
                            </el-checkbox-group>
                        </template>
                    </div>
                </template>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="close">取消</el-button>
            <el-button type="danger" :loading="submitLoading" @click="submit('addInfoForm')">确定</el-button>
        </div>
    </el-dialog>
</template>
<script>
import sha1 from 'js-sha1';
import { GetRole, GetSystem, GetPermButton, UserAdd, UserEdit} from "@/api/user";
// 过滤
import { stripscript, validatePass, validateEmail } from '@/utils/validate';
// 组件
import CityPicker from "@c/CityPicker"
export default {
    name: 'dialogInfo',
    components: { CityPicker },
    data(){
        // 验证用户名
        let validateUsername = (rule, value, callback) => {
            if (value === '') {
                callback(new Error('请输入用户名'));
            } else if(validateEmail(value)){
                callback(new Error('用户名格式有误'));
            } else {
                callback(); //true
            }
        };
        // 验证密码
        let validatePassword = (rule, value, callback) => {
            /**
             * 业务逻辑
             * 1、编辑状态：
             *    需要检验：data.form.id存在并且，密码不为空时
             *    不需要检验：data.form.id存在并且，密码为空时
             * 
             * 2、添加状态
             *    需要检验：data.form.id不存在
             */
            if(this.form.id && !value) {
                callback();
            }
            if((this.form.id && value) || !this.form.id) {
                // 过滤后的数据
                if(value) {
                    this.form.password = stripscript(value);
                    value = this.form.password;
                }
                if (value === '') {
                    callback(new Error("请输入密码"));
                } else if (validatePass(value)) {
                    callback(new Error("密码为6至20位数字+字母"));
                } else {
                    callback();
                }
            }
        };
        return {
            dialog_info_flag: false,  // 弹窗标记
            cityPickerData: {},
            formLabelWidth: '90px',
            form: {
                username: "",
                truename: "",
                password: "",
                phone: "",
                region: "",
                status: "2",
                role: [],
                btnPerm: []
            },
            rules: {
                username: [
                    { validator: validateUsername, trigger: 'blur' }
                ],
                password: [
                    { validator: validatePassword, trigger: 'blur' }
                ],
                role: [
                    { required: true, message: "请选择角色", trigger: 'change' }
                ]
            },
            // 角色选择
            roleItem: [],
            // 按钮权限
            btnPerm: [],
            // 按钮加载
            submitLoading: false
        }
    },
    methods: {
        /**
         * 请求角色
         */
        getRole(){
            // if(data.roleItem.length > 0 && data.btnPerm.length > 0) { return false }
            if(this.roleItem.length === 0) {
                GetRole().then(response => {
                    this.roleItem = response.data.data
                })
            }
            if(this.btnPerm.length === 0) {
                GetPermButton().then(response => {
                    this.btnPerm = response.data.data
                })
            }
        },
        /**
         * 弹窗打开，动画结束时
         */
        openDialog(){
            // 角色请求
            this.getRole()
            // 初始值处理
            let editData = this.editData;
            if(editData.id) { // 编辑
                editData.role = editData.role ? editData.role.split(',') : []; // 数组
                editData.btnPerm = editData.btnPerm ? editData.btnPerm.split(',') : []; // 数组
                // 循环JSON对象
                for(let key in editData) {
                    this.form[key] = editData[key]
                }
            }else{ // 添加
                this.form.id && delete this.form.id
            }
        },
        close(){
            this.dialog_info_flag = false;
            this.resetForm();
            this.$emit("update:flag", false);
        },
        resetForm(){
            this.cityPickerData = {}
            this.$refs.addInfoForm.resetFields();
           
        },
        submit(formName){
            this.$refs[formName].validate((valid) => {
                // 表单验证通过
                if (valid) {
                    // 数据处理
                    let requestData = Object.assign({}, this.form); //
                    requestData.role = requestData.role.join();  // 数组转字符串，默认以，号隔开
                    requestData.btnPerm = requestData.btnPerm.join();  // 数组转字符串，默认以，号隔开
                    requestData.region = JSON.stringify(this.cityPickerData);
                    // 添加状态：需要密码，并且加密码
                    // 编辑状态：值存在，需要密码，并且加密码；否删除
                    if(requestData.id) {
                        if(requestData.password) {
                            requestData.password = sha1(requestData.password)
                        }else{
                            delete requestData.password
                        }
                        this.userEdit(requestData)
                    }else{
                        requestData.password = sha1(requestData.password);
                        this.userAdd(requestData)
                    }
                    
                } else {
                    return false;
                }
            })
        },
        userAdd(requestData){
            UserAdd(requestData).then(response => {
                let data = response.data
                this.$message({
                    message: data.message,
                    type: "success"
                })
                this.close();
                // 
                this.$emit('componentCallback', {
                    function: 'refreshData'
                })
            })
        },
        userEdit(requestData){
            UserEdit(requestData).then(response => {
                let data = response.data
                this.$message({
                    message: data.message,
                    type: "success"
                })
                this.close();
                this.$emit('componentCallback', {
                    function: 'refreshData'
                })
            })
        }
    },
    props: {
        flag: {
            type: Boolean,
            default: false
        },
        editData: {
            type: Object,
            default: () => {}
        }
    },
    watch: {
        flag(newValue, oldValue){
            this.dialog_info_flag = newValue;
        },
        editData: {
            handler(newValue, oldValue){
                // 三四层数据 
                // {
                //     phone: '1111',
                //     aaaa: {
                //         name: '绑三',
                //         bbbb: {
                //             name: '李四',
                //             cccc: {

                //             }
                //         }
                //     }
                // }
                console.log('newValue')
                console.log(newValue)
            }
        }
    }
}
</script>
<style scoped>

</style>