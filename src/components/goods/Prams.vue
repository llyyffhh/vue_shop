<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品参数</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card>
            <el-alert
                    title="注意：只允许为第三级分类设置相关参数"
                    type="warning"
                    :closable="false" show-icon>
            </el-alert>
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <el-cascader filterable :options="cateList" v-model="selectedKeys" @change="handleChange"
                                 :props="cateProps"></el-cascader>
                </el-col>
            </el-row>
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">
                        添加参数
                    </el-button>
                    <el-table :data="manyTableData" border stripe>
                        <el-table-column type="expand">
                            <template v-slot="scopeProps">
                                <el-tag v-for="(item, i) in scopeProps.row.attr_vals" :key="i" closable @close="handleClose(i,scopeProps.row)">
                                    {{item}}
                                </el-tag>
                                <el-input class="input-new-tag" v-if="scopeProps.row.inputVisible" v-model="scopeProps.row.inputValue" ref="saveTagInput"
                                          size="small"
                                          @keyup.enter.native="handleInputConfirm(scopeProps.row)"
                                          @blur="handleInputConfirm(scopeProps.row)"></el-input>
                                <el-button v-else size="small" @click="showInput(scopeProps.row)">新增</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="参数名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template v-slot="scopeProps">
                                <el-button type="primary" icon="el-icon-edit" size="mini"
                                           @click="showEditDialog(scopeProps.row.attr_id)">编辑
                                </el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scopeProps.row.attr_id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" size="mini" :disabled="isBtnDisabled" @click="addDialogVisible=true">
                        添加属性
                    </el-button>
                    <el-table :data="onlyTableData" border stripe>
                        <el-table-column type="expand">
                            <template v-slot="scopeProps">
                                <el-tag v-for="(item, i) in scopeProps.row.attr_vals" :key="i" closable @close="handleClose(i,scopeProps.row)">
                                    {{item}}
                                </el-tag>
                                <el-input class="input-new-tag" v-if="scopeProps.row.inputVisible" v-model="scopeProps.row.inputValue" ref="saveTagInput"
                                          size="small"
                                          @keyup.enter.native="handleInputConfirm(scopeProps.row)"
                                          @blur="handleInputConfirm(scopeProps.row)"></el-input>
                                <el-button v-else size="small" @click="showInput(scopeProps.row)">新增</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index"></el-table-column>
                        <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                        <el-table-column label="操作">
                            <template v-slot="scopeProps">
                                <el-button type="primary" icon="el-icon-edit" size="mini"
                                           @click="showEditDialog(scopeProps.row.attr_id)">编辑
                                </el-button>
                                <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeParams(scopeProps.row.id)">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>
        <el-dialog :title="'添加'+titleText" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="addForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="addParams">确 定</el-button>
            </span>
        </el-dialog>

        <el-dialog :title="'修改'+titleText" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
                <el-form-item :label="titleText" prop="attr_name">
                    <el-input v-model="editForm.attr_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="editParams">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Prams",
        data() {
            return {
                cateList: [],
                cateProps: {
                    expandTrigger: true,
                    value: 'cat_id',
                    label: "cat_name",
                    children: 'children'
                },
                selectedKeys: [],
                activeName: 'many',
                manyTableData: [],
                onlyTableData: [],
                addDialogVisible: false,
                editDialogVisible: false,
                addForm: {
                    attr_name: ''
                },
                editForm: {
                    attr_name: '',
                    attr_id : ''
                },
                addFormRules: {
                    attr_name: [{
                        required: true, message: "请输入参数名称", trigger: "blur"
                    }]
                },
                editFormRules: {
                    attr_name: [{
                        required: true, message: "请输入参数名称", trigger: "blur"
                    }]
                },
                inputVisible : false,
                inputValue : ''
            }
        },
        created() {
            this.getCateList()
        },
        computed: {
            isBtnDisabled() {
                if (this.selectedKeys.length !== 3) {
                    return true
                }
                return false
            },
            cateId() {
                if (this.selectedKeys.length === 3) {
                    return this.selectedKeys[2]
                }
                return null
            },
            titleText() {
                if (this.activeName === "many") {
                    return "动态参数"
                }
                return "静态属性"
            }
        },
        methods: {
            async getCateList() {
                const {data: res} = await this.$http.get("/categories")
                if (res.meta.status != 200) {
                    return this.$message.error('获取商品分类失败')
                }
                this.cateList = res.data
            },
            handleChange() {
                this.getParamsDate()
            },
            async getParamsDate() {
                if (this.selectedKeys.length != 3) {
                    this.selectedKeys = []
                    this.manyTableData = []
                    this.onlyTableData = []
                    return
                }
                const {data: res} = await this.$http.get(`categories/${this.cateId}/attributes`, {params: {sel: this.activeName}})
                if (res.meta.status != 200) {
                    return this.$message.error('参数列表获取失败')
                }
                res.data.forEach(function (item) {
                    item.attr_vals = item.attr_vals ? item.attr_vals.split(' '):[]
                    item.inputVisible = false
                    item.inputValue = ''
                })
                console.log(res.data)
                if (this.activeName === "many") {
                    this.manyTableData = res.data
                } else {
                    this.onlyTableData = res.data
                }
            },
            handleTabClick() {
                this.getParamsDate()
            },
            addDialogClosed() {
                this.$refs.addFormRef.resetFields()
            },
            addParams() {
                this.$refs.addFormRef.validate(async valid => {
                    if (!valid) return
                    const {data: res} = await this.$http.post(`categories/${this.cateId}/attributes`,
                        {
                            attr_name: this.addForm.attr_name,
                            attr_sel: this.activeName
                        })
                    if (res.meta.status != 201) {
                        return this.$message.error(res.meta.msg)
                    }
                    this.$message.success('添加参数成功')
                    this.addDialogVisible = false
                    this.getParamsDate()
                })
            },
            async showEditDialog(attr_id) {
                const {data: res} = await this.$http.get(`/categories/${this.cateId}/attributes/${attr_id}`,
                    {params: {attr_sel: this.activeName}})
                if (res.meta.status !== 200) {
                    return this.$message.error('获取参数信息失败')
                }
                this.editForm = res.data
                this.editDialogVisible = true
            },
            editDialogClosed() {
                this.$refs.addFormRef.resetFields()
            },
            editParams() {
                this.$refs.editFormRef.validate(async valid => {
                    if (!valid) return
                    const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
                        {
                            attr_name: this.editForm.attr_name,
                            attr_sel: this.activeName
                        })
                    if (res.meta.status != 200) {
                        return this.$message.error(res.meta.msg)
                    }
                    this.$message.success('修改参数成功')
                    this.editDialogVisible = false
                    this.getParamsDate()
                })
            },
            async removeParams(attr_id) {
                const result = await this.$confirm("此操作将永久删除该属性，是否继续？","提示",{
                    confirmButtonText : '确定',
                    cancelButtonText:'取消',
                    type : 'warning'
                }).catch(err=>error)
                if (result !== "confirm") {
                    return this.$message.info('已取消删除')
                }
                const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
                if (res.meta.status !== 200) {
                    return this.$message.error('删除参数失败')
                }
                this.$message.success('删除参数成功！')
                this.getParamsDate()
            },
            async handleInputConfirm(row) {
                if (row.inputValue.trim().length === 0) {
                    row.inputValue = ''
                    row.inputVisible = false
                    return
                }
                row.attr_vals.push(row.inputValue.trim())
                row.inputValue = ''
                row.inputVisible = false
                const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
                    {attr_name:row.attr_name,
                          attr_sel : row.attr_sel,
                          attr_vals: row.attr_vals.join(' ')})
                if (res.meta.status !== 200) {
                    return this.$message.error('修改参数失败')
                }
                this.$message.success('修改参数成功');

            },
            async saveAttrVals(row) {
                const {data: res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
                    {attr_name:row.attr_name,
                        attr_sel : row.attr_sel,
                        attr_vals: row.attr_vals.join(' ')})
                if (res.meta.status !== 200) {
                    return this.$message.error('修改参数失败')
                }
                this.$message.success('修改参数成功');
            },
            showInput(row) {
                row.inputVisible = true
                this.$nextTick(_=>{
                    this.$refs.saveTagInput.$refs.input.focus()
                })
            },
            handleClose(i,row) {
                row.attr_vals.splice(i,1)
                this.saveAttrVals(row)
            }
        }
    }
</script>

<style lang="less" scoped>
    .cat_opt {
        margin: 15px 0;
    }

    .el-table {
        margin-top: 15px;
    }
    .el-tag {
        margin : 10px;
    }
    .input-new-tag {
        width: 120px;
    }

</style>
