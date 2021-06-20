<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!-- 添加用户对话框 -->
      <el-dialog
        title="添加角色"
        :visible.sync="addDialogVisible"
        width="50%"
        @close="addDialogClosed"
      >
        <!-- 内容主体区 -->
        <el-form
          :model="addForm"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="80px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <!-- 底部区 -->
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="addDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="addUser">确 定</el-button>
          </span>
        </template>
      </el-dialog>
      <!-- 修改用户对话框 -->
      <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%">
        <el-form ref="editFormRef" :model="editForm" label-width="70px">
          <el-form-item label="角色名">
            <el-input v-model="editForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述">
            <el-input v-model="editForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <template #footer>
          <span class="dialog-footer">
            <el-button @click="editDialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="editRolesInfo">确 定</el-button>
          </span>
        </template>
      </el-dialog>
      <!-- 角色列表区 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-right"></i>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <!-- for循环渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                      >{{ item3.authName }}</el-tag
                    >
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-dalete"
              @click="removeRolesById(scope.row.id)"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分配权限 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
      
    >
      <!-- 树形控件 -->
      <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref=treeRef ></el-tree>
      <template #footer>
        <span class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRights"
            >确 定</el-button
          >
        </span>
      </template>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 所有角色列表
      rolelist: [],
      //   控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      //   添加用户表单数据
      addForm: {
        roleName: "",
        roleDesc: "",
      },
      //   添加表单验证规则
      addFormRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" },
        ],
      },
      // 控制修改用户对话框的显示与隐藏
      editDialogVisible: false,
      // 查询到的用户信息对象
      editForm: {},
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible:false,
      // 所有权限数据
      rightslist:[],
      // 树形控件属性绑定对象
      treeProps:{
        label:'authName',
        children:'children'
      },
      // 默认选中节点id值
      defKeys:[],
      // 当前即将分配权限的角色id
      roleId:''
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    // 获取所有角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败");
      }
      this.rolelist = res.data;
      console.log(this.rolelist);
    },
    // 监听添加用户对话框的关闭
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    // 点击按钮添加成员
    addUser() {
      this.$refs.addFormRef.validate(async (valid) => {
        if (!valid) return;
        // 可以发起请求
        const { data: res } = await this.$http.post("roles", this.addForm);
        if (res.meta.status !== 201) {
          this.$message.error("添加失败");
        }
        this.$message.success("添加成功");
        this.addDialogVisible = false;
        this.getRolesList();
      });
    },
    // 展示编辑用户的对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("查询失败");
      }
      this.editForm = res.data;
      this.editDialogVisible = true;
    },
    // 修改用户提交
    async editRolesInfo() {
      const { data: res } = await this.$http.put("roles/" + this.editForm.id, {
        roleName: this.editForm.roleName,
        roleDesc: this.editForm.roleDesc,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("请求失败,应该是接口坏了。。。");
      }
      this.editDialogVisible = false;
      this.getRolesList();
      this.$message.success("更新用户成功");
    },
    // 根据id删除用户
    async removeRolesById(id) {
      // 弹框询问
      const confirmResult = await this.$confirm(
        "此操作将永久删除角色, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      // console.log(confirmResult)
      if (confirmResult !== "confirm") {
        return this.$message.info("已经取消删除");
      }
      const { data: res } = await this.$http.delete("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("删除失败");
      }
      this.$message.success("删除成功");
      this.getRolesList();
    },
    // 根据id删除权限
    async removeRightById(role, rightId) {
      // 弹框提示是否删除
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("取消了删除");
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除失败");
      }
      // this.getRolesList()
      role.children = res.data;
    },
    // 展示分配权限对话框
    async showSetRightDialog(role) {
      this.roleId=role.id
      // 获取所有权限数据
      const{data:res}=await this.$http.get('rights/tree')
      if(res.meta.status!==200){
        return this.$message.error('获取所有权限数据失败')
      }
      // 把获取到的权限数据保存到data中
      this.rightslist=res.data
      console.log(this.rightslist)
      // 递归获取三级节点id
      this.getLeafKeys(role,this.defKeys)
      this.setRightDialogVisible=true
    },
    // 通过递归形式，获取角色下所有三级权限id
    getLeafKeys(node,arr){
      // 如果当前node节点不包含children属性，则是三级节点
      if(!node.children){
        return arr.push(node.id)
      }
      node.children.forEach(item=>
      this.getLeafKeys(item,arr))
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed(){
      this.defKeys=[]
    },
    // 点击为角色分配权限
    async allotRights(){
      const keys=[
        ...this.$refs.treeRef.
        getCheckedKeys(),
        ...this.$refs.treeRef.
        getHalfCheckedKeys()
      ]
      const idStr=keys.join(',')
      const{data:res}=await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
      if(res.meta.status!==200){
        return this.$message.error('分配权限失败')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible=false
    }
  },
};
</script>
<style scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: felx;
  align-items: center;
}
</style>