<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>
            <tree-table class="treeTable"
                        :data="cateList"
                        :columns="columns"
                        :selection-type="false"
                        :expand-type="false"
                        show-index
                        index-text="#"
                        border
                        :show-row-hover="false">
                <template slot="isOk" slot-scope="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color:darkgreen;"></i>
                    <i class="el-icon-error" v-else></i>
                </template>
                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0 ">一级</el-tag>
                    <el-tag type="success" size="mini" v-else-if="scope.row.cat_level ===1">二级</el-tag>
                    <el-tag type="warning" size="mini" v-else>三级</el-tag>
                </template>
                <template slot="opt" slot-scope="scope">
                    <el-button type="primary" icon="el-icon-edit" size="mini" round>编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" round>删除</el-button>
                </template>
            </tree-table>
            <el-pagination
                    @size-change="handleSizeChange"
                    @current-change="handleCurrentChange"
                    :current-page="queryInfo.pagenum"
                    :page-sizes="[5, 10, 15, 20]"
                    :page-size="queryInfo.pagesize"
                    layout="total, sizes, prev, pager, next, jumper"
                    :total="total">
            </el-pagination>
        </el-card>
        <!--        添加分类对话框-->
        <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
                <el-form-item label="分类名称:" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类:">
                    <el-cascader clearable
                                 v-model="selectedKeys"
                                 :options="parentCateList"
                                 :props="cascaderProps"
                                 @change="parentCateChange">
                    </el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Cate",
        data() {
            return {
                queryInfo: {
                    type: 3,
                    pagenum: 1,
                    pagesize: 5
                },
                cateList: [],
                total: 0,
                columns: [{
                    label: "分类名称",
                    prop: "cat_name"
                }, {
                    label: "是否有效",
                    type: 'template',
                    template: "isOk"
                }, {

                    label: "排序",
                    type: 'template',
                    template: "order"
                }, {

                    label: "操作",
                    type: 'template',
                    template: "opt"
                }],
                addCateDialogVisible: false,
                addCateForm: {
                    cat_name: '',
                    cat_pid: 0,
                    cat_level: 0
                },
                addCateFormRules: {
                    cat_name: [
                        {required: true, message: "请输入分类名称", trigger: 'blur'}
                    ]
                },
                parentCateList: [],
                cascaderProps: {
                    expandTrigger: 'hover',
                    checkStrictly: true,
                    value: 'cat_id',
                    label: 'cat_name',
                    children: 'children',
                },
                selectedKeys: []

            }
        },
        created() {
            this.getCateList()
        },
        methods: {
            async getCateList() {
                const {data: res} = await this.$http.get('/categories', {params: this.queryInfo})
                if (res.meta.status != 200) {
                    return this.$message.error("获取商品分类失败")
                }
                this.cateList = res.data.result
                this.total = res.data.total
            },
            handleSizeChange(newSize) {
                this.queryInfo.pagesize = newSize
                this.getCateList()
            },
            handleCurrentChange(newPage) {
                this.queryInfo.pagenum = newPage
                this.getCateList()
            },
            showAddCateDialog() {
                this.getParentList()
                this.addCateDialogVisible = true
            },
            async getParentList() {
                const {data: res} = await this.$http.get('categories', {params: {type: 2}})
                if (res.meta.status != 200) {
                    return this.$message.error(res.data.msg)
                }
                this.parentCateList = res.data
            },
            parentCateChange() {
                if (this.selectedKeys.length > 0) {
                    this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
                    this.addCateForm.cat_levle = this.selectedKeys.length
                    return
                } else {
                    this.addCateForm.cat_pid = 0
                    this.addCateForm.cat_levle = 0
                    return
                }

            },
            addCate() {
                this.$refs.addCateFormRef.validate(async valid => {
                    if (!valid) return
                    const {data : res} = await this.$http.post('categories', this.addCateForm)
                    console.log(res)
                    if (res.meta.status != 201) {
                        return this.$message.error("添加失败")
                    }
                    this.$message.success("添加分类成功")
                    this.getCateList()
                    this.addCateDialogVisible = false
                })
            },
            addCateDialogClosed() {
                this.$refs.addCateFormRef.resetFields()
                this.selectedKeys = []
            }

        }
    }
</script>

<style lang="less" scoped>
    .el-row {
        margin-bottom: 15px;
    }

    .el-cascader {
        width: 100%;
    }
</style>
