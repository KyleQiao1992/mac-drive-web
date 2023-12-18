<template>
  <div class="change-file-model">
    <!-- Global file search -->
    <el-input
      v-if="fileType === 0"
      class="select-file-input"
      v-model="searchFile.fileName"
      placeholder="Search by file name"
      size="mini"
      maxlength="255"
      :clearable="true"
      @change="handleSearchInputChange"
      @clear="$emit('getTableDataByType')"
      @keyup.enter.native="handleSearchInputChange(searchFile.fileName)"
    >
      <i
        slot="prefix"
        class="el-input__icon el-icon-search"
        title="Click to search"
        @click="handleSearchInputChange(searchFile.fileName)"
      ></i>
    </el-input>

    <el-divider direction="vertical" class="split-line"></el-divider>
    <!-- File display mode -->
    <el-radio-group v-model="currentshowModel" size="mini">
      <el-radio-button :label="0">
        <i class="el-icon-tickets"></i> List
      </el-radio-button>
      <el-radio-button :label="1">
        <i class="el-icon-s-grid"></i> Grid
      </el-radio-button>
      <el-radio-button :label="2" v-if="fileType === 1">
        <i class="el-icon-date"></i> Timeline
      </el-radio-button>
    </el-radio-group>
  </div>
</template>

<script>
export default {
  name: "ShowModel",
  props: {
    // File type
    fileType: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      // File search data
      searchFile: {
        fileName: ""
      }
    };
  },
  computed: {
    // View mode - 0 List | 1 Grid | 2 Timeline
    currentshowModel: {
      get() {
        return this.$store.getters.showModel;
      },
      set(val) {
        this.$store.commit("changeShowModel", val); // Change view mode
      }
    }
  },
  methods: {
    // Search input box search event
    handleSearchInputChange(value) {
      if (value === "") {
        this.$emit("getTableDataByType");
      } else {
        this.$emit("getSearchFileList", value);
      }
    }
  }
};
</script>

<style lang="stylus" scoped>
.change-file-model {
  margin-right: 5px;
  display: flex;
  padding: 0;
}

.operation-menu-wrapper.file-type-6 {
  margin: 8px 0;
  justify-content: flex-end;
}

.select-file-input {
  margin-right: 3px;
  width: 200px;
}

.split-line {
  margin: 5px;
}

.img-text-wrapper {
  display: flex;
  align-items: center;

  img {
    margin-right: 4px;
    height: 24px;
  }
}
</style>
