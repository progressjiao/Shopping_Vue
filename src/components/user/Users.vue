<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator=">">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>用户管理</el-breadcrumb-item>
            <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card class="box-card">
            <el-row :gutter="20">
                <el-col :span = "8">
                    <el-input placeholder="请输入内容" class="input-with-select" v-model="paramsList.query" clearable @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span = "4">
                    <el-button type="primary" @click="addDialogVisible = true">添加用户</el-button>
                </el-col>
            </el-row>
            <!-- 表格显示用户列表 -->
            <el-table border
                :data="userList"
                stripe
                style="width: 100%">
                <el-table-column type="index"></el-table-column>
                <el-table-column prop="username" label="姓名" width="180"></el-table-column>
                <el-table-column prop="email" label="邮箱" width="180"></el-table-column>
                <el-table-column prop="mobile" label="电话" width="180"></el-table-column>
                <el-table-column prop="role_name" label="角色" width="180"></el-table-column>
                <el-table-column label="状态" width="120px">
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="handleStateChange(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180px">
                    <template slot-scope="scope">
                        <el-tooltip class="item" effect="dark" content="编辑信息" placement="top" :enterable="false">
                            <el-button type="primary" icon="el-icon-edit" size="mini" @click="showUserInfo(scope.row.id)"></el-button>
                        </el-tooltip>
                        <el-tooltip class="item" effect="dark" content="删除用户" placement="top" :enterable="false">
                            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteUser(scope.row.id)"></el-button>
                        </el-tooltip>
                        <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRoleDialog(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页 -->
            <el-pagination
                @size-change="handleSizeChange"
                @current-change="handleCurrentChange"
                :current-page="paramsList.pagenum"
                :page-sizes="[1, 2, 5, 10]"
                :page-size="paramsList.pageSize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>

            <!-- 添加用户对话框 -->
            <el-dialog
                title="添加用户"
                :visible.sync="addDialogVisible"
                width="40%"
                @close="addDialogClose">
                <el-form :model="addForm" :rules="Rules" ref="addFormRef" label-width="70px" class="demo-ruleForm">
                    <el-form-item label="用户名" prop="username">
                        <el-input v-model="addForm.username"></el-input>
                    </el-form-item>
                    <el-form-item label="密码" prop="password">
                        <el-input v-model="addForm.password"></el-input>
                    </el-form-item>
                    <el-form-item label="邮箱" prop="email">
                        <el-input v-model="addForm.email"></el-input>
                    </el-form-item>
                    <el-form-item label="手机" prop="mobile">
                        <el-input v-model="addForm.mobile"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="addDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addUser">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 修改用户对话框 -->
            <el-dialog
                title="修改用户信息"
                :visible.sync="editDialogVisible"
                width="40%"
                @close="editDialogClose">
                <el-form :model="editForm" :rules="Rules" ref="editFormRef" label-width="70px" class="demo-ruleForm">
                    <el-form-item label="用户名" disabled>
                        <el-input v-model="editForm.username"></el-input>
                    </el-form-item>
                    <el-form-item label="邮箱" prop="email">
                        <el-input v-model="editForm.email"></el-input>
                    </el-form-item>
                    <el-form-item label="手机" prop="mobile">
                        <el-input v-model="editForm.mobile"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="editDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editUser">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 分配角色对话框 -->
            <el-dialog
                title="分配角色"
                :visible.sync="setRoleDialogVisible"
                width="50%"
                @close="setRoleDialogClose">
                <div>
                    <p>当前用户：{{userInfo.username}}</p>
                    <p>当前角色：{{userInfo.role_name}}</p>
                    <p>分配新角色：
                        <el-select v-model="selectedRoleId" placeholder="请选择">
                            <el-option
                            v-for="item in roleList"
                            :key="item.id"
                            :label="item.roleName"
                            :value="item.id">
                            </el-option>
                        </el-select>
                    </p>
                </div>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="setRoleDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="setRole()">确 定</el-button>
                </span>
            </el-dialog>
        </el-card>
    </div>
</template>

<script>
export default {
    data() {
        var checkEmail = (rule, value, cb) => {
            const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
            if(regEmail.test(value)) {
                return cb()
            }
            cb(new Error('请输入合法邮箱！'))
        }
        var checkMobile = (rule, value, cb) => {
            const regMobile = /^(0|86|17951)?(13[0-9]|15[0-9]|17[678]|18[0-9]|14[57])[0-9]{8}$/
            if(regMobile.test(value)) {
                return cb()
            }
            cb(new Error('请输入合法手机号！'))
        }
        return {
            paramsList: {
                query: '',
                pagenum: 1,
                pagesize: 10
            },
            userList: [],
            total: 0,
            // 添加用户对话框的显示与隐藏
            addDialogVisible: false,
            // 添加用户表单
            addForm: {
                username:"",
                password:"",
                email:"",
                mobile:""
            },
            Rules:{
                // 验证用户名是否合法
                username: [
                    { required: true, message: '请输入登录名称', trigger: 'blur' },
                    { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                ],
                // 验证密码是否合法
                password: [
                    { required: true, message: '请输入登录密码', trigger: 'blur' },
                    { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
                ],
                email: [
                    { required: true, message: '请输入邮箱', trigger: 'blur' },
                    { validator: checkEmail, trigger: 'blur'}
                    
                ],
                mobile: [
                    { required: true, message: '请输入手机号', trigger: 'blur' },
                    { validator: checkMobile, trigger: 'blur'}
                ]
            },
            editDialogVisible: false,
            editForm: {
                username:"",
                email:"",
                mobile:""
            },
            setRoleDialogVisible: false,
            // 需要分配角色的用户信息（展示在对话框）
            userInfo: {},
            // 角色列表 
            roleList:[],
            selectedRoleId:''
        }
    },
    created () {
        this.getUserList();
    },
    methods: {
        async getUserList() {
            const {data : res} = await this.$http.get('users', { params: this.paramsList})
            if(res.meta.status !== 200) return this.$message.error("获取用户列表失败");
            this.userList = res.data.users;
            this.total = res.data.total;
        },
        // 每一页展示的条数
        handleSizeChange(newSize){
            this.paramsList.pagesize = newSize;
            this.getUserList();
            // 
        },
        // 监听当前页改变
        handleCurrentChange(currentPage){
            this.paramsList.pagenum = currentPage;
            this.getUserList();
        },
        // 监听状态改变
        async handleStateChange(userInfo) {
            console.log(userInfo);
            const { data: res} = await this.$http.put(`users/${userInfo.id}/state/${userInfo.mg_state}`)
            console.log(res);
            if(res.meta.status !== 200){
                userInfo.mg_state = !userInfo.mg_state;
                return this.$message.error("更新用户状态失败！");
            }
            this.$message.success("更新用户状态成功！")
        },
        // 添加用户对话框关闭的时候重置表单
        addDialogClose(){
            this.$refs.addFormRef.resetFields()
        },
        // 添加用户请求
        addUser(){
            this.$refs.addFormRef.validate(async valid => {
                if(!valid) return
                const {data:res} = await this.$http.post('users', this.addForm);
                if(res.meta.status !== 201){
                    return this.$message.error('添加用户失败！');
                }
                this.$message.success('添加用户成功！');
                this.addDialogVisible = false; // 关闭添加用户对话框
                this.getUserList(); // 请求展示添加完之后的用户列表
            })
        },
        // 根据id查询用户信息展示在编辑信息表单
        async showUserInfo(id){
            const {data:res} = await this.$http.get('users/'+id);
            if(res.meta.status !== 200) return  this.$message.error("查询用户信息失败！");
            this.editForm = res.data;
            this.editDialogVisible = true;
        },
        // 编辑用户对话框关闭的时候清空校验结果
        editDialogClose(){
            this.$refs.editFormRef.resetFields()
        },
        // 发起请求，修改用户数据
        editUser(){
            this.$refs.editFormRef.validate(async valid => {
                if(!valid) return
                const {data:res} = await this.$http.put('users/'+this.editForm.id, {
                    email : this.editForm.email,
                    mobile: this.editForm.mobile
                });
                if(res.meta.status !== 200){
                    return this.$message.error('修改用户信息失败！');
                }

                this.editDialogVisible = false;
                this.getUserList();
                this.$message.success('修改用户信息成功！');
            })
        },
        // 删除用户
        async deleteUser(id){
            // 弹出提示框
            const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            }).catch(err => err);
            // console.log(confirmResult); // cancel | confirm
            if(confirmResult == 'cancel'){
                return this.$message.info("取消删除")
            }
            
            const {data:res} = await this.$http.delete('users/'+id);
            if(res.meta.status !== 200){
                    return this.$message.error('删除用户失败！');
            }
            this.$message.success('删除用户成功！');
            this.getUserList();
        },
        async showSetRoleDialog(userInfo){
            this.userInfo = userInfo;
            const {data:res} = await this.$http.get('roles');
            // console.log(res.data);
            if(res.meta.status !== 200) return this.$message.error('获取角色列表失败');
            this.roleList = res.data;

            this.setRoleDialogVisible = true;
        },
        async setRole(){
            if(!this.selectedRoleId){
                return this.$message.error("请选择一个角色！");
            }
            console.log(this.userInfo.id, this.selectedRoleId);
            const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`, {rid: this.selectedRoleId});
            console.log(res.data);
            if(res.meta.status !== 200){
                return this.$message.error('分配角色失败！')
            }
            this.$message.success('分配角色成功！');
            this.getUserList();
            setRoleDialogVisible = false
        },
        setRoleDialogClose(){
            this.selectedRoleId = '',
            this.userInfo = {}
        }
    },
}
</script>

<style lang="less" scoped>

</style>