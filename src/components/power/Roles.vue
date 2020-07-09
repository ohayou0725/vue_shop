<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" round @click="addDialogVisible=true">添加角色</el-button>
                </el-col>
            </el-row>

            <el-table :data="roleList" border stripe>
                <el-table-column type="expand">
                    <template v-slot="scopeProps">
                        <el-row v-for="(item1, i1) in scopeProps.row.children" :key="item1.id"
                                :class="['vcenter','bd-bottom',i1 === 0 ? 'bd-top':'']">
                            <el-col :span="5">
                                <el-tag closable @close="removeRightById(scopeProps.row,item1.id)"> {{item1.authName}}
                                </el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="19">
                                <el-row :class="[i2 === 0 ? '': 'bd-top','vcenter']"
                                        v-for="(item2, i2) in item1.children" :key="item2.id">
                                    <el-col :span="6">
                                        <el-tag type="success" closable
                                                @close="removeRightById(scopeProps.row,item2.id)">{{item2.authName}}
                                        </el-tag>
                                        <i class="el-icon-caret-right"></i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag type="warning" v-for="(item3, i) in item2.children" :key="item3.id"
                                                closable @close="removeRightById(scopeProps.row,item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type="index"></el-table-column>
                <el-table-column label="角色" prop="roleName"></el-table-column>
                <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
                <el-table-column label="操作" width="300px">
                    <template v-slot="scopeProps">
                        <el-button type="primary" icon="el-icon-edit" size="mini" round
                                   @click="showEditDialog(scopeProps.row.id)">编辑
                        </el-button>
                        <el-button type="danger" icon="el-icon-delete" size="mini" round
                                   @click="removeRoleByRoleId(scopeProps.row.id)">删除
                        </el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" round @click="showSetRightsDialog(scopeProps.row)">
                            分配权限
                        </el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <el-dialog title="添加角色" :visible.sync="addDialogVisible" ref="addRoleRef" width="50%" @close="addDialogClosed">
            <!-- 内容主题区域 -->
            <el-form :model="addForm" :rules="addRoleRules" ref="addFormRef" label-width="70px">
                <el-form-item label="角色名" prop="roleName">
                    <el-input v-model="addForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                    <el-input v-model="addForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false" around>取 消</el-button>
        <el-button type="primary" @click="addRole" around>确 定</el-button>
      </span>
        </el-dialog>

        <el-dialog title="修改角色" :visible.sync="editDialogVisible" ref="editRoleRef" width="50%">
            <!-- 内容主题区域 -->
            <el-form :model="editForm" :rules="addRoleRules" ref="editFormRef" label-width="70px">
                <el-form-item label="角色名" prop="roleName" size="small">
                    <el-input v-model="editForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc" size="small">
                    <el-input v-model="editForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <!-- 底部区域 -->
            <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false" around>取 消</el-button>
        <el-button type="primary" @click="editRole()" around>确 定</el-button>
      </span>
        </el-dialog>
        <!--        分配权限-->
        <el-dialog title="分配权限" :visible.sync="setRightsDialogVisable" width="50%" @close="setRightDialogClosed">
            <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all
            :default-checked-keys="defaultKeys" ref="treeRef"></el-tree>

            <span slot="footer" class="dialog-footer">
            <el-button @click= "setRightsDialogVisable = false" around>取 消</el-button>
            <el-button type="primary" around @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Roles",
        data() {
            return {
                roleList: [],
                addDialogVisible: false,
                editDialogVisible: false,
                addForm: {
                    roleName: '',
                    roleDesc: ''
                },
                editForm: {},
                addRoleRules: {
                    roleName: [
                        {required: true, message: "请输入角色名", trigger: "blur"},
                    ],
                    roleDesc: [
                        {required: true, message: "请输入角色名", trigger: "blur"},
                    ]
                },
                setRightsDialogVisable : false,
                rightsList:[],
                treeProps : {
                    label : "authName",
                    children :'children'
                },
                defaultKeys : [],
                roleId : ''

            }
        },
        created() {
            this.getRolesList()
        },
        methods: {
            async getRolesList() {
                const {data: res} = await this.$http.get('/roles')
                if (res.meta.status != 200) {
                    return this.$message.error("获取角色列表失败")
                }
                this.roleList = res.data
            },
            addRole() {
                this.$refs.addFormRef.validate(async (valid) => {
                    if (!valid) return
                    const {data: res} = await this.$http.post('/roles', this.addForm)
                    if (res.meta.status != 201) {
                        return this.$message.error(res.meta.msg)
                    }
                    this.addDialogVisible = !this.addDialogVisible
                    this.$message.success("添加角色成功")
                    this.getRolesList()
                })
            },
            addDialogClosed() {
                this.$refs.addRoleRef.resetFields()
            },

            async showEditDialog(id) {
                const {data: res} = await this.$http.get(`/roles/${id}`)
                if (res.meta.status != 200) {
                    return this.$message.error(res.meta.msg)
                }
                this.editForm = res.data
                this.editDialogVisible = !this.editDialogVisible
            },

            editRole(id) {
                this.$refs.editFormRef.validate(async (valid) => {
                    const {data: res} = await this.$http.put(`/roles/${this.editForm.roleId}`, this.editForm)
                    if (res.meta.status != 200) {
                        return this.$message.error(res.meta.msg)
                    }
                    this.editDialogVisible = !this.editDialogVisible
                    this.$message.success("修改角色成功!")
                    this.getRolesList()
                })
            },
            async removeRoleByRoleId(id) {
                const result = await this.$confirm("此操作将永久删除该权限，是否继续？", "提示", {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(error => {
                    return error
                })
                if (result != "confirm") {
                    return this.$message.info("已经取消删除")
                }
                const {data: res} = await this.$http.delete(`/roles/${id}`)
                if (res.meta.status != 200) {
                    return this.$message.error(res.meta.msg)
                }
                this.$message.success("删除角色成功!")
                this.getRolesList()
            },
            async removeRightById(role, rightId) {
                const result = await this.$confirm("此操作将永久删除该角色，是否继续？", "提示", {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning'
                }).catch(error => {
                    return error
                })
                if (result != "confirm") {
                    return this.$message.info("取消了删除")
                }
                const {data: res} = await this.$http.delete(`/roles/${role.id}/rights/${rightId}`)
                if (res.meta.status !== 200) {
                    return this.$message.error(res.meta.msg)
                }
                role.children = res.data
            },
            async showSetRightsDialog(role){
                this.roleId = role.id
                const {data : res} = await this.$http.get(`rights/tree`)
                if (res.meta.status != 200) {
                    return this.$message.error(res.meta.msg)
                }
                this.rightsList = res.data
                this.getLeafKeys(role,this.defaultKeys)
                this.setRightsDialogVisable = true
            },
            getLeafKeys(node,arr) {
                if (!node.children) {
                    return arr.push(node.id)
                }
                node.children.forEach(item =>{
                    this.getLeafKeys(item,arr)
                })
            },
            setRightDialogClosed() {
                this.defaultKeys = []
            },
            async allotRights() {
                const keys = [...this.$refs.treeRef.getCheckedKeys(),...this.$refs.treeRef.getHalfCheckedKeys()]
                const idStr = keys.join(",")
                const {data:res} = await this.$http.post(`/roles/${this.roleId}/rights`,{rids:idStr})
                if (res.meta.status != 200) {
                    return this.$message.error(res.meta.msg)
                }
                this.$message.success("分配权限成功")
                this.getRolesList()
                this.setRightsDialogVisable = false
            }
        }
    }
</script>

<style lang="less" scoped>
    .el-row {
        margin-bottom: 15px;
    }

    .el-tag {
        margin: 7px;
    }

    .bd-top {
        border-top: 1px solid #eee;
    }

    .bd-bottom {
        border-bottom: 1px solid #eee;
    }

    .vcenter {
        display: flex;
        align-items: center;
    }
</style>
