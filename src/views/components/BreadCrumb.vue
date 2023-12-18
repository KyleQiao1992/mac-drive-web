<template>
  <!-- Breadcrumb -->
  <div class="bread-crumb-wrapper">
    <div class="current-path">Current Location:</div>
    <!-- When viewing files by type -->
    <el-breadcrumb class="bread-crumb" v-if="fileType" separator="/">
      <el-breadcrumb-item>{{ fileTypeMap[fileType] }}</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- When viewing files by path -->
    <el-breadcrumb class="bread-crumb" v-else separator="/">
      <!-- When clicking on a level in the breadcrumb navigation bar, change the route -->
      <el-breadcrumb-item
        v-for="(item, index) in breadCrumbList"
        :key="index"
        :to="{ query: { fileType: 0, filePath :  item.path } }"
      > {{ item.name }}
      </el-breadcrumb-item
      >
    </el-breadcrumb>
  </div>
</template>

<script>
export default {
  name: "BreadCrumb",
  props: {
    fileType: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      fileTypeMap: {
        1: "Image",
        2: "Documents",
        3: "Videos",
        4: "Music",
        5: "Compressed",
        6: "Others",
        8: "Recycle Bin",
        9: "My Shares"
      }
    };
  },
  computed: {
    // When viewing by path, the breadcrumb data split from the current path
    breadCrumbList() {
      let path = this.$route.query.filePath;
      let pathList = path ? path.split("/").slice(1, -1) : []; // Path list, take the second to the second-to-last, as the first and last array items are empty strings
      let res = [{ name: "All Files", path: "/" }];
      pathList.forEach((element, index) => {
        res.push({
          name: element,
          path: `${res[index].path}${element}/`
        });
      });
      return res;
    }
  }
};
</script>

<style lang="stylus" scoped>
.bread-crumb-wrapper {
  height: 32px;
  line-height: 32px;
  display: flex;
  align-items: center;
}
</style>
