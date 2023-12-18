<template>
  <div class="home">
    <!-- Left menu - Distinguish file types -->
    <SideMenu class="home-left" :storageValue="storageValue"></SideMenu>
    <!-- Right content area -->
    <div class="home-right">
      <UserHeader style="height: 40px" :user="user"></UserHeader>
      <!-- Breadcrumb navigation bar -->
      <BreadCrumb :fileType="fileType" style="height: 30px"></BreadCrumb>
      <router-view />
      <div class="right-main" v-if="this.$route.path === '/index'">
        <div class="operation-wrapper">
          <OperationMenu
            :fileType="fileType"
            :filePath="filePath"
            @getTableData="getFileData"
            @handleUploadFile="handleUploadFile"
            @handleSelectFile="setOperationFile"
            @handleMoveFile="setMoveFileDialog"
            :operationFileList="operationFileList"
          ></OperationMenu>
          <FileUploader
            ref="globalUploader"
            @getTableData="getFileData"
          ></FileUploader>
          <!-- Viewing mode switch component, pass fileType to the child component -->
          <ShowModel :fileType="fileType"></ShowModel>
        </div>
        <!-- File table display area -->
        <FileTable
          v-if="showModel === 0"
          :tableData="tableData"
          :fileType="fileType"
          :loading="loading"
          @getTableData="getFileData"
          @handleSelectFile="setOperationFile"
          @handleMoveFile="setMoveFileDialog"
        ></FileTable>
        <FileGrid
          v-if="showModel === 1"
          :tableData="tableData"
          :loading="loading"
        ></FileGrid>
        <!-- Pagination component -->
        <FilePagination
          style="position:absolute; bottom:0;padding-bottom: 20px"
          :pageData="pageData"
          @changePageData="changePageData"
        ></FilePagination>
        <!-- Move file modal dialog -->
        <MoveFileDialog
          :dialogMoveFile="dialogMoveFile"
          @setSelectFilePath="setSelectFilePath"
          @confirmMoveFile="confirmMoveFile"
          @handleMoveFile="setMoveFileDialog"
        ></MoveFileDialog>
        <!-- Streaming file online viewing components -->
        <ImgReview></ImgReview>
        <AudioReview></AudioReview>
        <VideoPlayer></VideoPlayer>
        <CodeReview></CodeReview>
      </div>
    </div>
  </div>
</template>


<script>
import SideMenu from "./components/SideMenu.vue"; // Import the left menu component
import BreadCrumb from "./components/BreadCrumb.vue"; // Import the breadcrumb navigation bar
import FileTable from "./components/FileTable.vue"; // Import the file table display area
import FilePagination from "./components/FilePagination.vue"; // Import the pagination component
import OperationMenu from "./components/OperationMenu.vue"; // Import file creation component
import ShowModel from "./components/ShowModel.vue"; // Import viewing mode switching component
import FileGrid from "./components/FileGrid.vue"; // Import grid component
import MoveFileDialog from "./components/MoveFileDialog"; // Import move file component
import ImgReview from "@/components/ImgReview"; // Image viewing
import UserHeader from "@/views/header/UserHeader"; // Header
import FileUploader from "./components/FileUploader.vue"; // File upload component
import AudioReview from "@/components/AudioReview"; // Audio playback
import VideoPlayer from "@/components/VideoPlayer"; // Video playback
import CodeReview from "@/components/CodeReview"; // Code playback

import axios from "axios";

import {
  batchMoveFile,
  getFileByName,
  getFileListByPath,
  getFileListByType,
  getFileStorage,
  getFileTree,
  moveFile
} from "@/request/file.js";

export default {
  name: "Home",
  components: {
    VideoPlayer,
    UserHeader,
    SideMenu,
    BreadCrumb,
    FileTable,
    FilePagination,
    OperationMenu,
    ShowModel,
    FileGrid,
    FileUploader,
    MoveFileDialog,
    ImgReview,
    AudioReview,
    CodeReview
  },
  data() {
    return {
      user: {}, // User information
      storageValue: 0, // Storage space usage size
      loading: false,
      tableData: [], // File list
      pageData: {
        currentPage: 1, // Page number
        pageCount: 10, // Number of items per page
        total: 0 // Total count
      },
      // Data for the move file modal dialog
      dialogMoveFile: {
        visible: false, // Whether the dialog is displayed
        fileTree: [] // Directory tree
      },
      isBatch: false, // Whether it is a batch operation
      operationFile: {}, // Information of the file operated on individually
      operationFileList: [], // Information of the files operated on in batch
      selectFilePath: "" // Target path
    };
  },
  computed: {
    // File type selected in the left menu
    fileType() {
      return this.$route.query.fileType
        ? Number(this.$route.query.fileType)
        : 0;
    },
    // Viewing mode
    showModel() {
      return this.$store.getters.showModel;
    },
    // Current path
    filePath() {
      return this.$route.query.filePath ? this.$route.query.filePath : "/";
    }
  },
  watch: {
    fileType() {
      this.getFileData(); // Get file list
    },
    filePath() {
      // When selecting all in the left menu, and the file path changes, get the file list again
      if (this.fileType === 0) {
        this.getFileData(); // Get file list
      }
    }
  },
  mounted() {
    this.getFileData(); // Get file list
    this.created();
  },
  methods: {
    // Upload file button click event
    handleUploadFile() {
      // Trigger the event to open the file upload window in the child component
      this.$refs.globalUploader.triggerSelectFileClick();
    },
    // Get user information
    created() {
      axios.get("/api/admin/info").then(resp => {
        // console.log(resp);
        this.user = resp;
        // console.log(this.user);
      });
    },
    // Get file list
    getFileData() {
      this.loading = true; // Open table loading status
      if (this.fileType === 0) {
        // When 'All' is selected in the left menu, get the file list by path
        this.loading = false;
        this.getFileDataByPath();
      } else {
        // When a type other than 'All' is selected in the left menu, get the file list by type
        this.getFileDataByType();
      }
      this.getStorageValue(); // Get file storage space
    },
    // Get file storage space
    getStorageValue() {
      getFileStorage().then(res => {
        this.storageValue = res.data ? Number(res.data) : 0;
      });
    },
    // Get file list by path
    getFileDataByPath() {
      getFileListByPath({
        filePath: this.filePath, // Pass the current path
        currentPage: this.pageData.currentPage,
        pageCount: this.pageData.pageCount
      }).then(
        res => {
          // console.log(res);
          this.loading = false; // Close table loading status
          this.tableData = res.data.list; // Assign table data
          this.pageData.total = res.data.total; // Assign total count to pagination component
        },
        error => {
          this.loading = false;
          console.log(error);
        }
      );
    },
    // Get file list by type
    getFileDataByType() {
      getFileListByType({
        fileType: this.fileType, // File type
        currentPage: this.pageData.currentPage, // Current page number
        pageCount: this.pageData.pageCount // Number of items per page
      }).then(
        res => {
          this.loading = false; // Close table loading status
          this.tableData = res.data.list; // Assign table data
          this.pageData.total = res.data.total; // Assign total count to pagination component
        },
        error => {
          this.loading = false;
          console.log(error);
        }
      );
    },
    // Fuzzy search to get file list
    getFileByName() {
      getFileByName({
        fileName: "7", // File name
        currentPage: this.pageData.currentPage, // Current page number
        pageCount: this.pageData.pageCount // Number of items per page
      }).then(
        res => {
          this.loading = false; // Close table loading status
          this.tableData = res.data.list; // Assign table data
          this.pageData.total = res.data.total; // Assign total count to pagination component
        },
        error => {
          this.loading = false;
          console.log(error);
        }
      );
    },
    // Pagination component - When page number or items per page change
    changePageData(pageData) {
      this.pageData.currentPage = pageData.currentPage; // Assign page number
      this.pageData.pageCount = pageData.pageCount; // Assign number of items per page
      this.getFileData(); // Get file list
    },
    /**
     * Set file information for moving files
     * @param {Boolean} isBatch Whether it is a batch move, true for batch, false for individual file operation
     * @param {Object | Array} file Information of the file to move, Object for individual operation, Array for batch operation
     */
    setOperationFile(isBatch, file) {
      this.isBatch = isBatch; // Save the operation type
      if (isBatch) {
        this.operationFileList = file; // Batch operation files
      } else {
        this.operationFile = file; // Individual operation file
      }
    },
    /**
     * Set data related to the move file dialog
     * @param {Boolean} visible Open/Close move file modal dialog
     */
    setMoveFileDialog(visible) {
      this.dialogMoveFile.visible = visible; // Open the dialog
      if (visible) {
        // When opening the dialog, get the folder directory tree
        getFileTree().then(res => {
          if (res.code === 200) {
            this.dialogMoveFile.fileTree = [res.data];
          } else {
            this.$message.error(res.code);
          }
        });
      }
    },
    // Set the target path for moving files
    setSelectFilePath(selectFilePath) {
      this.selectFilePath = selectFilePath;
    },
    // Move file modal dialog - Confirm button event
    confirmMoveFile() {
      if (this.isBatch) {
        // Batch move
        batchMoveFile({
          filePath: this.selectFilePath,
          files: JSON.stringify(this.operationFileList)
        }).then(res => {
          if (res.code === 200) {
            this.getFileData(); // Refresh file list
            this.dialogMoveFile.visible = false; // Close dialog
            this.operationFileList = [];
          } else {
            this.$message.error(res.message);
          }
        });
      } else {
        // Single file move
        moveFile({
          id: this.operationFile.id, // File ID
          file_name: this.operationFile.file_name, // File name
          file_url: this.selectFilePath // Target path
        }).then(res => {
          // console.log(res);
          if (res.code === 200) {
            this.getFileData(); // Refresh file list
            this.dialogMoveFile.visible = false; // Close dialog
          } else {
            this.$message.error(res.message);
          }
        });
      }
    }
  }
};
</script>


<style lang="stylus" scoped>
.home {
  // 使用flex布局 菜单居左固定宽度 右侧内容区域自适应宽度
  background-color: #eef0f4;
  display: flex;
  margin: 0;
  padding: 0;
  font-family: Arial, Helvetica, sans-serif;

  .home-left {
    box-sizing: border-box;
  }

  .home-right {
    box-sizing: border-box;
    width: calc(100vh - 220px);
    padding-left: 20px;
    padding-right: 20px;
    flex: 1;

    .operation-wrapper {
      padding-top 10px
      margin-bottom: 16px;
      display: flex;
      align-items: center;
      justify-content: space-between;

      // 左侧菜单按钮组 样式调整
      >>> .operation-menu-wrapper {
        flex: 1;
      }
    }

    .right-main {
      border-top-right-radius 15px
      border-top-left-radius 15px
      padding-right 10px
      padding-left 10px
      background-color white
      height calc(100vh - 130px)
    }
  }
}
</style>
