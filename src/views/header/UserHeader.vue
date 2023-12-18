<template>
  <div class="el_header">
    <!-- Header left side icons li -->
    <ul class="header_left_ul">
      <li title="Refresh" @click="goF5">
        <i class="el-icon-refresh" />
      </li>
    </ul>
    <!-- Header right side icons li -->
    <ul class="header_right_ul">
      <!--      <li title="Online Chat" @click="goChat">-->
      <!--        <i class="el-icon-bell" />-->
      <!--      </li>-->

      <li class="throwLiStyle" style="width: 135px">
        <el-dropdown @command="handleCommand">
          <div class="el-dropdown-link">
            <!-- User avatar, src is the default not loaded image path -->
            <el-avatar
              :size="30"
              src=""
              @error="errorHandler"
              class="user_Face"
            >
              <img :src="user.userFace" />
            </el-avatar>
            <!-- Username -->
            <span>{{ user.name }}</span>
          </div>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="userInfo" icon="el-icon-s-custom"
            >Personal Center</el-dropdown-item
            >
            <el-dropdown-item
              command="exit"
              icon="el-icon-switch-button"
              divided
            >Logout</el-dropdown-item
            >
          </el-dropdown-menu>
        </el-dropdown>
      </li>
      <li title="More">
        <i class="el-icon-s-operation" />
      </li>
    </ul>
  </div>
</template>


<script>
import axios from "axios";

export default {
  name: "UserHeader",
  props: {
    user: {
      type: Object,
      required: true
    }
  },
  data() {
    return {};
  },
  methods: {
    /* User information dropdown list click */
    handleCommand(command) {
      switch (command) {
        case "userInfo":
          this.$router.push("/userInfo");
          break;
        case "setting":
          // Additional functionality can be implemented here for settings
          break;
        case "exit":
          this.$confirm("This action will log you out, do you want to continue?", "Warning", {
            confirmButtonText: "Confirm",
            cancelButtonText: "Cancel",
            type: "warning"
          })
            .then(() => {
              // Perform logout
              axios.post("/api/logout");
              // Clear data in sessionStorage
              window.sessionStorage.clear();
              // Clear menu information in Vuex
              this.$store.commit("initRoutes", []);
              // Redirect to login page
              this.$router.replace("/");
            })
            .catch(() => {
              this.$message({
                type: "info",
                message: "Operation cancelled"
              });
            });
          break;
      }
    },
    /* User avatar load failed */
    errorHandler() {
      return true;
    },
    // Redirect to chat window
    goChat() {
      this.$router.push("/chat");
    },
    // Force update the component (commonly used to refresh)
    goF5() {
      this.$forceUpdate();
    }
  },
  computed: {
    // Additional computed properties can be added here
  }
};
</script>


<style lang="less" scoped>
.el-menu-vertical-demo:not(.el-menu--collapse) {
  width: 250px;
}

/*主容器*/
.body_container {
  height: 100%;
  background-color: #eef0f4;
  overflow: hidden;
}
.el-container {
  height: 100%;
}

/*头部*/
.el_header {
  display: flex;
  height: 40px;
  /*align-items: center;*/
  /*去除头部内边距多余的空格*/
  justify-content: space-between;
  //border-bottom: 1px #F5F5F5 solid;
}

//头部图标ul li
.el_header ul {
  margin: 0;
  padding: 0;
  list-style: none;
}
.el_header ul li {
  float: left;
  width: 50px;
  text-align: center;
  line-height: 60px;
}

/*鼠标移至头部icon*/
.el_header ul li:hover {
  background-color: #f7f7f7;
}

/*显示的头像*/
.el-dropdown-link {
  cursor: pointer;
}

/*用户头像*/
.user_Face {
  margin-right: 10px;
  width: 30px;
  height: 32px;
  background-color: #fff;
  vertical-align: middle;
}
</style>
