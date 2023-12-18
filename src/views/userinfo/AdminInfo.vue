<template>
  <!-- Personal Center -->
  <div style="overflow:hidden;">
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>{{ admin.name }}</span
        ><span style="font-size: 12px;color: #808080;margin-left: 10px"
      >({{ admin.username }})</span
      >
        <el-button
          @click="handleEdit"
          style="float: right; padding: 3px 0"
          type="text"
          icon="el-icon-edit"
          title="Edit"
        ></el-button>
      </div>
      <div>
        <!-- User Avatar -->
        <div class="user_avatar">
          <el-upload
            action="/admin/upUserFace"
            :headers="headers"
            :show-file-list="true"
            :on-success="onSuccess"
          >
            <el-avatar
              :size="85"
              :src="admin.userFace"
              title="Click to change user avatar"
            >
              <img :src="admin.userFace" class="user_face" />
            </el-avatar>
          </el-upload>
        </div>

        <!-- User Detailed Information -->
        <ul class="user_info_ul">
          <li>
            Nickname: <el-tag size="small">{{ admin.name }}</el-tag>
          </li>
          <li>
            Email: <el-tag size="small">{{ admin.email }}</el-tag>
          </li>
          <li>
            Storage Size: <el-tag size="small">{{ admin.disksize }}G</el-tag>
          </li>
        </ul>
      </div>
      <el-button
        @click="updatePwd"
        type="danger"
        size="small"
        style="float: right"
      >Change Password</el-button
      >
    </el-card>

    <!-- Edit User Information Dialog -->
    <el-dialog
      title="Edit User Information"
      :visible.sync="UserDialogVisible"
      width="30%"
    >
      <div>
        <el-form
          :model="updateAdmin"
          :rules="rules"
          ref="ruleForm"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="Account:" prop="username">
            <el-input
              v-model="updateAdmin.username"
              size="small"
              disabled="disabled"
            ></el-input>
          </el-form-item>
          <el-form-item label="Nickname:" prop="name">
            <el-input v-model="updateAdmin.name" size="small"></el-input>
          </el-form-item>
          <el-form-item label="Email:" prop="age">
            <el-input
              v-model="updateAdmin.email"
              size="small"
              disabled="disabled"
            ></el-input>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="UserDialogVisible = false" size="small"
        >Cancel</el-button
        >
        <el-button type="primary" @click="userDialogSubmit" size="small"
        >Confirm</el-button
        >
      </span>
    </el-dialog>

    <!-- Change User Password Dialog -->
    <el-dialog title="Change Password" :visible.sync="UserPassword" width="30%">
      <div>
        <el-form
          :model="updatePassword"
          :rules="rulesPassword"
          ref="ruleForm"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="Old Password:" prop="oldPassword" type="password">
            <el-input
              v-model="updatePassword.oldPassword"
              type="password"
              size="small"
            ></el-input>
          </el-form-item>
          <el-form-item label="New Password:" prop="newPassword" type="password">
            <el-input
              v-model="updatePassword.newPassword"
              type="password"
              size="small"
            ></el-input>
          </el-form-item>
          <el-form-item label="Confirm:" prop="agPassword" type="password">
            <el-input
              size="small"
              v-model="updatePassword.agPassword"
              type="password"
            ></el-input>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button type="primary" @click="UpdatePasswordButton" size="small"
        >Confirm</el-button
        >
        <el-button @click="UserPassword = false" size="small">Cancel</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "AdminInfo",
  data() {
    return {
      headers: {
        Authorization: window.sessionStorage.getItem("tokenStr")
      },
      imageUrl: "",
      admin: {},
      updateAdmin: {},
      updatePassword: {
        oldPassword: "",
        newPassword: "",
        agPassword: ""
      },
      UserDialogVisible: false,
      UserPassword: false,
      rules: {
        name: [{ required: true, message: "Please enter a nickname", trigger: "change" }]
      },
      rulesPassword: {
        oldPassword: [{ required: true, message: "Please enter the old password" }],
        newPassword: [{ required: true, message: "Please enter a new password" }],
        agPassword: [{ required: true, message: "Please confirm the password" }]
      }
    };
  },
  mounted() {
    this.getAdminInfo();
  },
  methods: {
    // Initialize data
    getAdminInfo() {
      axios.get("/api/admin/info").then(resp => {
        if (resp) {
          this.admin = resp;
        }
      });
    },
    // Edit user information
    handleEdit() {
      axios.get("/api/admin/info").then(resp => {
        if (resp) {
          this.updateAdmin = resp;
          this.UserDialogVisible = true;
        }
      });
    },
    // Submit user information
    userDialogSubmit() {
      this.$refs["ruleForm"].validate(valid => {
        if (valid) {
          axios.put("/api/admin/update", this.updateAdmin).then(resp => {
            if (resp) {
              this.UserDialogVisible = false;
              this.getAdminInfo();
            }
          });
        }
      });
    },
    // Change password dialog
    updatePwd() {
      this.UserPassword = true;
    },
    // Submit change password
    UpdatePasswordButton() {
      axios
        .put(
          "/api/admin/upPassword?oldPassword=" +
          this.updatePassword.oldPassword +
          "&password=" +
          this.updatePassword.newPassword
        )
        .then(resp => {
          if (resp.code === 200) {
            this.UserDialogVisible = false;
            this.getAdminInfo();
          }
        });
    },
    // Submit image successfully
    onSuccess() {
      this.getAdminInfo();
    }
  }
};
</script>


<style scoped>
/*用户信息卡片*/
.box-card {
  width: 600px;
  margin: 30px auto;
  padding: 10px;
}

/*展示用户具体信息ul*/
.user_info_ul {
  list-style: none;
}
.user_info_ul li {
  margin-bottom: 20px;
}

.user_avatar {
  cursor: pointer;
  display: flex;
  justify-content: center;
}
/*用户头像*/
.user_face {
}
</style>
