<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card class="box-card">
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
            <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" round @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="姓名" prop="username"></el-table-column>
        <el-table-column label="邮箱" prop="email"></el-table-column>
        <el-table-column label="电话" prop="mobile"></el-table-column>
        <el-table-column label="角色" prop="role_name"></el-table-column>
        <el-table-column label="状态">
          <template v-slot="scopeProps">
            <el-switch v-model="scopeProps.row.mg_state" @change="userStateChanged(scopeProps.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template v-slot="scopeProps">
            <!-- 编辑 -->
            <el-tooltip effect="dark" content="编辑用户" placement="top" :enterable="false">
              <el-button
                type="primary"
                icon="el-icon-edit"
                size="mini"
                round
                @click="showEditDialog(scopeProps.row.id)"
              ></el-button>
            </el-tooltip>
            <!-- 删除 -->
            <el-tooltip effect="dark" content="删除用户" placement="top" :enterable="false">
              <el-button type="danger" icon="el-icon-delete" size="mini" round @click="removeUserById(scopeProps.row.id)"></el-button>
            </el-tooltip>
            <!-- 分配角色 -->
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini" round @click="setRole(scopeProps.row)"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加用户的对话框 -->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <!-- 内容主题区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password" type="password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
        <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
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
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="分配角色" :visible.sync="setRoleDialogVisible" width="50%" @close="setRoleDialogClosed">
      <div>
        <p>当前的用户:{{userInfo.username}}</p>
        <p>当前的角色:{{userInfo.role_name}}</p>
        <p>分配新角色:
          <el-select v-model='selectedRoleId' placeholder="请选择">
            <el-option v-for="item in rolesList" :key="item.id" :label="item.roleName" :value="item.id"></el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    //验证邮箱
    var checkEamil = (rule, value, cb) => {
      const regEmail = /\w[-\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\.)+[A-Za-z]{2,14}/;
      if (regEmail.test(value)) {
        return cb();
      }

      cb(new Error("请输入合法的邮箱"));
    };

    //验证手机号
    var checkMobile = (rule, value, cb) => {
      const regMobile = /0?(13|14|15|18|17)[0-9]{9}/;
      if (regMobile.test(value)) {
        return cb();
      }

      cb(new Error("请输入正确的手机号"));
    };
    return {
      //获取用户列表对象
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 5
      },
      userList: [],
      total: 0,
      addDialogVisible: false,
      //   添加用户的表单数据
      addForm: {
        username: "",
        password: "",
        email: "",
        mobile: ""
      },
      addFormRules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          {
            min: 3,
            max: 10,
            message: "用户名长度在3到10个字符之间",
            trigger: "blur"
          }
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          {
            min: 6,
            max: 15,
            message: "用户名长度在6到15个字符之间",
            trigger: "blur"
          }
        ],
        email: [
          { required: true, message: "请输入邮箱", trigger: "blur" },
          { validator: checkEamil, trigger: "blur" }
        ],
        mobile: [
          { required: true, message: "请输入手机号", trigger: "blur" },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editDialogVisible : false,

      selectedRoleId : '',
      editForm : {},
      userInfo : {},
      rolesList :{},
    
      editFormRules : {
          email : [
              {required : true , message : '请输入用户邮箱',trigger: 'blur'},
              {validator: checkEamil, trigger:'blur'}
          ],
          mobile : [
              {required : true , message : '请输入用户手机号',trigger: 'blur'},
              {validator: checkMobile, trigger:'blur'}
          ]
      },
      setRoleDialogVisible : false
    };
  },
  created() {
    this.getUserList();
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get("users", {
        params: this.queryInfo
      });
      if (res.meta.status != 200) return this.$message.error(res.meta.msg);
      this.userList = res.data.users;
      this.total = res.data.total;
    },
    //监听pagesize
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getUserList();
    },

    //监听页码值改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getUserList();
    },
    //监听switch状态改变
    async userStateChanged(userInfo) {
      const { data: res } = await this.$http.put(
        `users/${userInfo.id}/state/${userInfo.mg_state}`
      );
      if (res.meta.status != 200) {
        userInfo.mg_state = !userInfo.mg_state;
        return this.$message.error("更新用户状态失败");
      }
      this.$message.success("更新用户状态成功");
    },
    //监听用户对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return;
        const { data: res } = await this.$http.post("users", this.addForm);
        if (res.meta.status != 201) {
          return this.$message(res.meta.msg);
        }
        this.$message.success("添加用户成功");
        this.addDialogVisible = false;
        this.getUserList();
      });
    },
    async showEditDialog(id) {
        const {data:res} = await this.$http.get(`/users/${id}`)
        if (res.meta.status != 200) {
            return this.$message.errro(res.meta.msg)
        }
        this.editForm = res.data
        this.editDialogVisible = true
    },

    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    editUserInfo() {
      this.$refs.editFormRef.validate(async (valid)=>{
        if(!valid) return
        const {data: res} = await this.$http.put(`users/${this.editForm.id}`,{
          email :this.editForm.email,
          mobile : this.editForm.mobile
        })
        if (res.meta.status != 200) {
          return this.$message.error(res.meta.msg)
        }

        this.editDialogVisible = false
        this.getUserList()
        this.$message.success("更新用户信息成功")
      })
    },
    async removeUserById(id) {
      const result = await this.$confirm("此操作将永久删除该用户，是否继续？","提示",{
        confirmButtonText : '确定',
        cancelButtonText:'取消',
        type : 'warning'
      }).catch(err => {

      })
      if (result !== "confirm") {
        return this.$message.info('已经取消删除')
      }
      const {data:res} = await this.$http.delete(`/users/${id}`)
      if (res.meta.status != 200) {
        return this.$message.error(res.meta.msg)
      }
      this.$message.success("删除用户成功！")
      this.getUserList()
    },
    async setRole(userInfo) {
      this.userInfo = userInfo
      const {data:res} = await this.$http.get('roles')
      if (res.meta.status !=200) {
        return this.$message.error("获取角色列表失败")
      }
      this.rolesList = res.data
      this.setRoleDialogVisible = true
    },
    async saveRoleInfo() {
        if (!this.selectedRoleId) {
          return this.$message.error("请选择分配的角色")
        }
        const {data:res} = await this.$http.put(`users/${this.userInfo.id}/role`,{rid:this.selectedRoleId})
        if (res.meta.status != 200) {
          return this.$message.error(res.meta.msg)
        }
        this.$message.success("更新角色成功")
        this.getUserList()
        this.setRoleDialogVisible = false
    },
    setRoleDialogClosed() {
      this.setRoleDialogVisible = ''
      this.userInfo = {}
    }

  }
};
</script>

<style lang="less" scoped>
.el-table {
  margin-top: 15px;
  font-size: 12px;
}
</style>
