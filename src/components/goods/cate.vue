<template>
    <div>

        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>权限列表</el-breadcrumb-item>
        </el-breadcrumb>

        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>

            <tree-table 
                border 
                :show-index="true" 
                :data="cateList" 
                :columns="columns" 
                :selection-type="false" 
                :expand-type="false">
                <template slot="isok" slot-scope="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen"></i>
                    <i class="el-icon-error" v-else style="color: red"></i>
                </template>

                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag size="mini" v-else-if="scope.row.cat_level === 1" type="success">二级</el-tag>
                    <el-tag size="mini" v-else type="warning">三级</el-tag>
                </template>
                
                <template slot="opt" slot-scope="scope">
                    <el-button type="primary" size="mini" icon="el-icon-edit" @click="showEditCateDialog(scope.row)">编辑</el-button>
                    <el-button type="danger"  size="mini" icon="el-icon-delete" @click="RemoveCateDialog(scope.row)">删除</el-button>
                </template>
            </tree-table>

            <el-pagination  
                @size-change="handleSizeChange" 
                @current-change="handleCurrentChange"
                :current-page="queryInfo.pagenum"
                :page-sizes="[3, 5, 10, 15]"
                :page-size="queryInfo.pagesize"
                layout="total, sizes, prev, pager, next, jumper"
                :total="total">
            </el-pagination>
        </el-card>

        <el-dialog
            title="添加分类"
            :visible.sync="addCateDialogVisible"
            width="50%"
            @close="addCateDialogColsed">

            <el-form 
                :model="addCateForm" 
                :rules="addCateFormRules" 
                ref="addCateFormRef" 
                label-width="100px">

                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>

                <el-form-item label="父级分类：">
                    <el-cascader
                        expand-trigger="hover"
                        :options="parentCateList"
                        v-model="selectedCateId"
                        :props="cateProps"
                        @change="handleChange" 
                        clearable
                        change-on-select>
                    </el-cascader>
                </el-form-item>

            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addCate">确 定</el-button>
            </span>
        </el-dialog>

        <el-dialog
            title="提示"
            :visible.sync="editCateDialogVisible"
            width="30%">

            <el-form :model="editRuleForm" :rules="addCateFormRules" ref="editRuleFormRef" label-width="100px">
                <el-form-item label="活动名称" prop="cat_name">
                    <el-input v-model="editRuleForm.cat_name"></el-input>
                </el-form-item>
            </el-form>

            <span slot="footer" class="dialog-footer">
                <el-button @click="editCateDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editCate">确 定</el-button>
            </span>
        </el-dialog>

    </div>
</template>

<script>
export default {
    data() {
        return {
            queryInfo: {
                type: 3,
                pagenum: 1,
                pagesize: 5
            },
            cateList: [],
            total: 0,
            columns: [
                {
                    label: '分类名称',
                    prop: 'cat_name',
                },
                {
                    label: '是否有效',
                    type: 'template',
                    template: 'isok',
                },
                {
                    label: '排序',
                    type: 'template',
                    template: 'order',
                },
                {
                    label: '操作',
                    type: 'template',
                    template: 'opt',
                }
            ],
            addCateDialogVisible: false,
            addCateForm: {
                cat_name: '',
                cat_pid: 0,
                cat_level: 0
            },
            addCateFormRules: {
                cat_name: [
                    {required: true, message: '请输入分类名称', trigger: 'blur'}
                ]
            },
            parentCateList: [],
            cateProps: {
                value: 'cat_id',
                label: 'cat_name',
                children: 'children'
            },
            selectedCateId: [],
            editCateDialogVisible: false,
            editRuleForm: {
                cat_name: '',
                cat_id: '',
            }
        }
    },
    created() {
        this.getCateList()
    },
    methods: {
        async getCateList() {
            const {data: res} = await this.$http.get('categories', {params: this.queryInfo})
            if(res.meta.status !== 200) {
                return this.$message.error("获取商品分类失败！");
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
            this.getParentCateList()
            this.addCateDialogVisible = true
        },
        async getParentCateList() {
            const {data: res} = await this.$http.get('categories',{params: {type: 2}})
            if(res.meta.status !== 200) {
                return this.$message.error("获取父级分类失败！");
            }
            this.parentCateList = res.data
        },
        handleChange() {
            if(this.selectedCateId.length > 0) {
                this.addCateForm.cat_pid = this.selectedCateId[this.selectedCateId.length - 1]
                this.addCateForm.cat_level = this.selectedCateId.length
            } else {
                this.addCateForm.cat_pid = 0
                this.addCateForm.cat_level = 0
            }      
        },
        addCate() {
            if (!this.addCateForm.cat_name.trim()) {
                this.$message.error("请输入分类名称！")
                return
            }
            this.$refs.addCateFormRef.validate(async valid => {
                if(!valid) return
                const {data: res} = await this.$http.post('categories', this.addCateForm)
                if(res.meta.status !== 201) {
                    return this.$message.error("添加分类失败！");
                }
                this.$message.success("添加分类成功！")
                this.getCateList()
                this.addCateDialogVisible = false
            })
        },
        addCateDialogColsed() {
            this.$refs.addCateFormRef.resetFields()
            this.selectedCateId = []
            this.addCateForm.cat_pid = 0
            this.addCateForm.cat_level = 0
        },
        showEditCateDialog(row) {
            this.editRuleForm.cat_name = row.cat_name
            this.editRuleForm.cat_id = row.cat_id
            this.editCateDialogVisible = true
        },
        async editCate() {
            if (!this.editRuleForm.cat_name.trim()) {
                this.$message.error("请输入分类名称！")
                return
            }
            const {data: res} = await this.$http.put('categories/' + this.editRuleForm.cat_id, {
                cat_name: this.editRuleForm.cat_name
            })
            if(res.meta.status !== 200) {
                return this.$message.error("编辑分类失败！");
            }
            this.$message.success("编辑分类成功！")
            this.getCateList()
            this.editCateDialogVisible = false
        },
        async RemoveCateDialog(row) {
            const Res = await this.$confirm('此操作将永久删除该分类, 是否继续?', '提示', {
                confirmButtonText: '确定',
                cancelButtonText: '取消',
                type: 'warning'
            })
            .catch(() => {
                this.$message({
                    type: 'info',
                    message: '已取消删除'
                });          
            });
            if(Res == 'confirm') {
                const {data: res} = await this.$http.delete('categories/' + row.cat_id)
                if(res.meta.status !== 200) {
                    return this.$message.error('删除失败！')
                }
                this.$message.success('删除成功！')
                this.getCateList()
            }

        },
    }
}
</script>

<style lang="less" scoped>
.zk-table {
    margin: 15px 0;
}
.el-cascader {
    width: 100%;
}

</style>