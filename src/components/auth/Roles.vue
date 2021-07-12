<template>
    <div>
        <!-- 面包屑导航 -->
        <el-breadcrumb separator=">">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>

            <el-table :data="roleList" border stripe>
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bdBottom', index==0 ? 'bdTop' : '', 'vcenter']" v-for="(item, index) in scope.row.children" :key="index">
                            <!-- 一级权限 -->
                            <el-col :span="5">
                                <el-tag>{{item.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="19">
                                <el-row :class="[index1 == 0 ? '' : 'bdTop', 'vcenter']" v-for="(item1, index1) in item.children" :key="index1">
                                    <el-col :span="6">
                                        <el-tag type="success">{{item1.authName}}</el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag  v-for="(item2, index2) in item1.children" :key="index2" type="warning" closable @close="removeRight(scope.row, item2.id)">{{item2.authName}}</el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>

                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色名称" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showRoleInfo(scope.row.id)">编辑</el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteRole(scope.row.id)" >删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>

            <!-- 添加角色对话框 -->
            <el-dialog
                title="添加角色"
                :visible.sync="addDialogVisible"
                width="40%"
                @close="addDialogClose">
                <el-form :model="addForm" :rules="Rules" ref="addFormRef" label-width="80px" class="demo-ruleForm">
                    <el-form-item label="角色名称">
                        <el-input v-model="addForm.roleName"></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述">
                        <el-input v-model="addForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="addDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="addRole">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 修改角色对话框 -->
            <el-dialog
                title="修改角色信息"
                :visible.sync="editDialogVisible"
                width="40%"
                @close="editDialogClose">
                <el-form :model="editForm" :rules="Rules" ref="editFormRef" label-width="80px" class="demo-ruleForm">
                    <el-form-item label="角色名称">
                        <el-input v-model="editForm.roleName"></el-input>
                    </el-form-item>
                    <el-form-item label="角色描述">
                        <el-input v-model="editForm.roleDesc"></el-input>
                    </el-form-item>
                </el-form>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="editDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="editRole">确 定</el-button>
                </span>
            </el-dialog>

            <!-- 分配角色对话框 -->
            <el-dialog
                title="分配角色"
                :visible.sync="setRightDialogVisible"
                width="50%"
                >
                <el-tree :data="rightList" :props="treeProps" ref="treeRef" node-key="id" :default-checked-keys="treeKeys" show-checkbox default-expand-all></el-tree>
                <span slot="footer" class="dialog-footer">
                    <el-button @click="setRightDialogVisible = false">取 消</el-button>
                    <el-button type="primary" @click="setRight">确 定</el-button>
                </span>
            </el-dialog>
        </el-card>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                roleList: [],
                addDialogVisible: false,
                addForm: {
                    roleName:'',
                    roleDesc:''
                },
                Rules:{
                    // 验证用户名是否合法
                    rolename: [
                        { required: true, message: '请输入角色', trigger: 'blur' },
                        { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
                    ],
                },
                editDialogVisible: false,
                editForm: {
                    roleName:'',
                    roleDesc:''
                },
                setRightDialogVisible: false,
                rightList: [],
                treeProps: {
                    label: 'authName',
                    children: 'children'
                },
                treeKeys:[],
                // 当前分配权限的id
                roleId:''
            }
        },
        created () {
            this.getRoleList();
        },
        methods: {
            async getRoleList() {
                const {data : res} = await this.$http.get('roles');
                if(res.meta.status !== 200){
                    return this.$message.error("获取用户列表失败！");
                }
                this.roleList = res.data;
            },
            addDialogClose(){
                this.$refs.addFormRef.resetFields()
            },
            addRole(){
                this.$refs.addFormRef.validate(async valid => {
                    if(!valid) return
                    const {data:res} = await this.$http.post('roles', this.addForm);
                    if(res.meta.status !== 201){
                        return this.$message.error('添加角色失败！');
                    }
                    this.$message.success('添加角色成功！');
                    this.addDialogVisible = false;
                    this.getRoleList();
                })
            },
            async showRoleInfo(id){
                const {data:res} = await this.$http.get("roles/"+id);
                if(res.meta.status !== 200){
                    return this.$message.error("获取角色信息失败！");
                }
                this.editForm = res.data;
                this.editDialogVisible = true;
            },
            editDialogClose(){
                this.$refs.editFormRef.resetFields()
            },
            editRole(){
                this.$refs.editFormRef.validate(async valid => {
                    if(!valid) return 
                    const {data:res} = await this.$http.put('roles/'+this.editForm.roleId, {
                        roleName: this.editForm.roleName,
                        roleDesc: this.editForm.roleDesc
                    });
                    console.log(res);
                    if(res.meta.status !== 200){
                        return this.$message.error("编辑角色信息失败！")
                    }
                    this.editDialogVisible = false;
                    this.getRoleList();
                    this.$message.success("编辑角色信息成功！")
                })
            },
            async deleteRole(id){
                // 弹出提示框
                const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => err);
                // console.log(confirmResult); // cancel | confirm
                if(confirmResult == 'cancel'){
                    return this.$message.info("取消删除")
                }

                const {data:res} = await this.$http.delete('roles/'+id);
                if(res.meta.status !== 200){
                    return this.$message.error('删除角色失败！');
                }
                this.$message.success('删除角色成功！');
                this.getRoleList();
            },
            async removeRight(role, rightId){
                console.log(role);
                const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(err => err);
                if(confirmResult == 'cancel'){
                    return this.$message.info("取消删除")
                }
                // console.log(confirmResult);
                const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
                if(res.meta.status!==200) return this.$message.error('删除权限失败！');
                role.children = res.data;
            },
            async showRightDialog(role){
                this.roleId = role.id;
                const {data:res} = await this.$http.get("rights/tree");
                this.rightList = res.data;
                this.getShowKeys(role ,this.treeKeys);
                this.setRightDialogVisible = true;
            },
            // 获取当前角色的所有三级权限
            getShowKeys(node, arr){
                if(!node.children){
                    return arr.push(node.id)
                }
                node.children.forEach(item => this.getShowKeys(item, arr))
            },
            // 分配权限
            async setRight(){
                const keys = [...this.$refs.treeRef.getCheckedKeys(),
                ...this.$refs.treeRef.getHalfCheckedKeys()]
                const idStr = keys.join(',');
                // console.log(idStr);
                const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr});
                
                if(res.meta.status !== 200) return this.$message.error("更新权限失败！")
                this.$message.success("更新权限成功！")
                
                this.getRoleList();
                this.setRightDialogVisible = false;
                
            }
        },
    }
</script>

<style lang="less" scoped>
.el-tag{
  margin: 7px;
}

.bdBottom{
    border-bottom: 1px solid #eee;
}

.bdTop{
    border-top: 1px solid #eee;
}
.vcenter{
    display: flex;
    align-items: center;
}
</style>