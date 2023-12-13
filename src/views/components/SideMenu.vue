<template>
  <!-- collapse attribute: controls the menu collapse and expansion -->
  <el-menu
      class="side-menu"
      :default-active="activeIndex"
      :router="true"
      :collapse="isCollapse"
      background-color="#fff"
      text-color="#000"
      active-text-color="#2578b5"
  >
    <div class="fold-wrapper">
      <!-- click event: toggle the menu collapse status when clicked -->
      <i
          class="el-icon-s-unfold"
          v-if="isCollapse"
          title="Expand"
          @click="isCollapse = false"
      ></i>
      <i
          class="el-icon-s-fold"
          v-else
          title="Collapse"
          @click="isCollapse = true"
      ></i>
    </div>
    <el-submenu index="myFile" class="my-file">
      <template slot="title">
        <!-- Icons are from the Element UI official icon library https://element.eleme.cn/#/zh-CN/component/icon -->
        <i class="el-icon-files"></i>
        <span slot="title">My Files</span>
      </template>
      <el-menu-item
          index="0"
          :route="{ name: 'index', query: { fileType: 0, filePath: '/' } }"
      >
        <i class="el-icon-menu"></i>
        <span slot="title">All</span>
      </el-menu-item>
      <el-menu-item index="1" :route="{ name: 'index', query: { fileType: 1 } }">
        <i class="el-icon-picture-outline"></i>
        <span slot="title">Images</span>
      </el-menu-item>
      <el-menu-item index="2" :route="{ name: 'index', query: { fileType: 2 } }">
        <i class="el-icon-document"></i>
        <span slot="title">Documents</span>
      </el-menu-item>
      <el-menu-item index="3" :route="{ name: 'index', query: { fileType: 3 } }">
        <i class="el-icon-video-camera"></i>
        <span slot="title">Videos</span>
      </el-menu-item>
      <el-menu-item index="4" :route="{ name: 'index', query: { fileType: 4 } }">
        <i class="el-icon-headset"></i>
        <span slot="title">Music</span>
      </el-menu-item>
      <el-menu-item index="5" :route="{ name: 'index', query: { fileType: 5 } }">
        <i class="el-icon-receiving"></i>
        <span slot="title">Compressed</span>
      </el-menu-item>

      <el-menu-item index="6" :route="{ name: 'index', query: { fileType: 6 } }">
        <i class="el-icon-takeaway-box"></i>
        <span slot="title">Others</span>
      </el-menu-item>
    </el-submenu>

    <el-menu-item index="8" :route="{ name: 'index', query: { fileType: 8 } }">
      <i class="el-icon-delete"></i>
      <span slot="title">Recycle Bin</span>
    </el-menu-item>

    <el-menu-item
        index="9"
        :route="{ name: 'index', query: { fileType: 9, filePath: '/' } }"
    >
      <i class="el-icon-share"></i>
      <span slot="title">My Share</span>
    </el-menu-item>

    <div class="storage-wrapper" :class="{ fold: isCollapse }">
      <el-progress
          :percentage="storagePercentage"
          :color="storageColor"
          :show-text="false"
          :type="isCollapse ? 'circle' : 'line'"
          :width="32"
          :stroke-width="isCollapse ? 4 : 6"
          stroke-linecap="square"
      ></el-progress>
      <div class="text" v-show="!isCollapse">
        <span class="label">Storage</span>
        <span
        >{{ storageValue | storageTrans }} /
          {{ storageMaxValue | storageTrans(true) }}
        </span>
      </div>
      <div class="text" v-show="isCollapse">
        <!--<span></span>-->
      </div>
    </div>
  </el-menu>
</template>


<script>
export default {
  name: "SideMenu",
  props: {
    storageValue: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      isCollapse: false, // Controls menu collapse and expansion
      storageMaxValue: Math.pow(1024, 3) * 100, // Default storage capacity, 100GB
      // Custom progress bar color, different percentages have different colors
      storageColor: [
        { color: "#89ea59", percentage: 50 },
        { color: "#eea740", percentage: 80 },
        { color: "#f35050", percentage: 100 }
      ]
    };
  },
  computed: {
    // The index of the currently active menu
    activeIndex() {
      return String(this.$route.query.fileType); // Get the file type included in the current route parameter
    },
    // Storage percentage
    storagePercentage() {
      return (this.storageValue / this.storageMaxValue) * 100;
    }
  },
  watch: {
    // Listen for changes in the collapse state, store in sessionStorage to maintain the state after page refresh
    isCollapse(newValue) {
      sessionStorage.setItem("isCollapse", newValue);
    }
  },
  filters: {
    // Calculate space usage
    storageTrans(size, status) {
      const B = 1024;
      const KB = Math.pow(1024, 2);
      const MB = Math.pow(1024, 3);
      const GB = Math.pow(1024, 4);
      if (status) {
        // Truncate to integer part
        if (!size) {
          return 0 + "KB";
        } else if (size < KB) {
          return (size / B).toFixed(0) + "KB";
        } else if (size < MB) {
          return (size / KB).toFixed(0) + "MB";
        } else if (size < GB) {
          return (size / MB).toFixed(0) + "GB";
        } else {
          return (size / GB).toFixed(0) + "TB";
        }
      } else {
        if (!size) {
          return 0 + "KB";
        } else if (size < KB) {
          return (size / B).toFixed(0) + "KB";
        } else if (size < MB) {
          return (size / KB).toFixed(2) + "MB";
        } else if (size < GB) {
          return (size / MB).toFixed(3) + "GB";
        } else {
          return (size / GB).toFixed(4) + "TB";
        }
      }
    }
  },
  created() {
    this.isCollapse = sessionStorage.getItem("isCollapse") === "true"; // Retrieve saved state
  }
};
</script>


<style lang="stylus" scoped>
@import '~@/assets/style/mixins.styl';

.side-menu {
  // 高度设置为屏幕高度减去顶部导航栏的高度
  height: 100vh;
  overflow: auto;
  // 调整滚动条样式
  setScrollbar(6px, #909399, #EBEEF5);

  // 折叠图标调整样式
  .fold-wrapper {
    height: 103px;
    line-height: 56px;
    padding: 0 20px;
    text-align: right;
    color: #343434;
    font-size: 24px;

    .el-icon-s-unfold, .el-icon-s-fold {
      cursor: pointer;

      &:hover {
        opacity: 0.5;
      }
    }
  }

  // 存储空间展示区
  .storage-wrapper {
    position: absolute;
    bottom: 0;
    left: 0;
    box-sizing: border-box;
    width: 100%;
    padding: 16px;
    z-index: 2;
    color: #224270;
    opacity: 0.8;

    .text {
      margin-top: 8px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      font-size: 14px;
    }
  }
}

// 对展开状态下的菜单设置宽度
.side-menu:not(.el-menu--collapse) {
  width: 220px;
}
</style>
