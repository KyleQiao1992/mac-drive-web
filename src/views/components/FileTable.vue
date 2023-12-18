<template>
  <el-table
    class="file-table"
    :data="tableData"
    height="calc(100vh - 232px)"
    :row-style="{ height: '50px' }"
    :cell-style="{ padding: '0px' }"
    style="width: 100%"
    v-loading="loading"
    @selection-change="handleSelectRow"
  >
    <!-- Checkbox Column -->
    <el-table-column
      type="selection"
      width="50"
      align="center"
    ></el-table-column>
    <el-table-column label prop="isDir" width="50" align="center">
      <template slot-scope="scope">
        <img :src="setFileImg(scope.row)" style="width: 30px" />
      </template>
    </el-table-column>
    <el-table-column prop="fileName" label="File Name">
      <template slot-scope="scope">
        <div style="cursor: pointer" @click="handleFileNameClick(scope.row)">
          {{
            scope.row.file_type
              ? `${scope.row.file_name}.${scope.row.file_type}`
              : `${scope.row.file_name}`
          }}
        </div>
      </template>
    </el-table-column>
    <el-table-column prop="extendName" label="Type" width="100">
      <template slot-scope="scope">
        <span>
          {{ scope.row.file_type ? scope.row.file_type : "Folder" }}
        </span>
      </template>
    </el-table-column>
    <el-table-column prop="file_size" label="Size" width="100">
      <template slot-scope="scope">
        <span>
          {{ calculateFileSize(scope.row.file_size) }}
        </span>
      </template>
    </el-table-column>
    <el-table-column prop="file_date" label="Upload Date" width="180">
    </el-table-column>
    <el-table-column
      prop="file_url"
      label="Original Path"
      width="180"
      v-if="fileType === 8"
    >
    </el-table-column>
    <el-table-column
      prop="file_url"
      label="Share Link"
      width="250"
      v-if="fileType === 9"
    >
    </el-table-column>
    <el-table-column
      prop="file_url"
      label="Share Date"
      width="180"
      v-if="fileType === 9"
    >
    </el-table-column>
    <el-table-column
      prop="file_url"
      label="Expiration Date"
      width="180"
      v-if="fileType === 9"
    >
    </el-table-column>
    <el-table-column
      prop="file_del_date"
      label="Deletion Date"
      width="180"
      v-if="fileType === 8"
    >
    </el-table-column>
    <!-- Operation Column - Custom Table Header -->
    <el-table-column :width="operaColumnIsFold ? 200 : 100">
      <!-- Custom Table Header -->
      <template slot="header">
        <span>Operation</span>
        <i
          class="el-icon-circle-plus"
          title="Expand"
          @click="operaColumnIsFold = true"
        ></i>
        <i
          class="el-icon-remove"
          title="Collapse"
          @click="operaColumnIsFold = false"
        ></i>
      </template>
      <template slot-scope="scope">
        <!-- Buttons when Operation Column is Expanded -->
        <div class="opera-unfold" v-if="operaColumnIsFold">
          <el-button
            type="text"
            size="small"
            v-if="fileType < 7"
            @click.native="handleClickDelete(scope.row)"
          >Delete</el-button
          >
          <el-button
            type="text"
            size="small"
            v-if="fileType === 8"
            @click.native="handleClickDelete(scope.row)"
          >Restore</el-button
          >
          <el-button
            type="text"
            size="small"
            v-if="fileType < 7"
            @click.native="handleClickMove(scope.row)"
          >Move</el-button
          >
          <el-button
            type="text"
            v-if="fileType === 8"
            size="small"
            @click.native="handleClickDelete(scope.row)"
          >Permanently Delete</el-button
          >
          <el-button
            type="text"
            size="small"
            v-if="fileType < 7"
            @click.native="handleClickRename(scope.row)"
          >Rename</el-button
          >
          <el-button type="text" size="small">
            <a
              target="_blank"
              style="display: block; color: inherit; text-decoration-line: none"
              :href="getDownloadPath(scope.row)"
              v-if="fileType < 7"
            >Download</a
            >
          </el-button>
        </div>
        <!-- Buttons when Operation Column is Collapsed -->
        <el-dropdown trigger="click" v-else>
          <el-button size="mini">
            Operation
            <i class="el-icon-arrow-down el-icon--right"></i>
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item
              @click.native="handleClickDelete(scope.row)"
              v-if="fileType < 7"
            >Delete</el-dropdown-item
            >
            <el-dropdown-item
              @click.native="handleClickMove(scope.row)"
              v-if="fileType < 7"
            >Move</el-dropdown-item
            >
            <el-dropdown-item
              @click.native="handleClickRename(scope.row)"
              v-if="fileType < 7"
            >Rename</el-dropdown-item
            >
            <el-dropdown-item
              @click.native="handleClickRename(scope.row)"
              v-if="fileType === 8"
            >File Restore</el-dropdown-item
            >
            <el-dropdown-item
              @click.native="handleClickRename(scope.row)"
              v-if="fileType === 8"
            >Permanently Delete</el-dropdown-item
            >
            <el-dropdown-item>
              <a
                target="_blank"
                :href="getDownloadPath(scope.row)"
                style="display: block; color: inherit; text-decoration-line: none"
                v-if="fileType < 7"
              >Download</a
              >
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </template>
    </el-table-column>
  </el-table>
</template>

<script>
import { deleteFile, renameFile } from "@/request/file";
export default {
  name: "FileTable",
  props: {
    // Table data
    tableData: {
      type: Array,
      required: true
    },
    // Table loading status
    loading: {
      type: Boolean,
      required: true
    },
    // File type
    fileType: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      operaColumnIsFold: sessionStorage.getItem("operaColumnIsFold") || false, // Table operation column - whether it's collapsed
      // File image map
      fileImgMap: {
        csv: require("../../assets/images/file/file_csv.png"),
        doc: require("../../assets/images/file/file_word.svg"),
        excel: require("../../assets/images/file/file_excel.svg"),
        exe: require("../../assets/images/file/file_exe.png"),
        gif: require("../../assets/images/file/file_gif.png"),
        html: require("../../assets/images/file/file_html.png"),
        json: require("../../assets/images/file/file_json.png"),
        mp3: require("../../assets/images/file/file_music.png"),
        flac: require("../../assets/images/file/file_music.png"),
        other: require("../../assets/images/file/file_unknown.png"),
        pdf: require("../../assets/images/file/file_pdf.png"),
        ppt: require("../../assets/images/file/file_ppt.svg"),
        rar: require("../../assets/images/file/file_rar.png"),
        svg: require("../../assets/images/file/file_svg.png"),
        fold: require("../../assets/images/file/dir.png"),
        txt: require("../../assets/images/file/file_txt.png"),
        zip: require("../../assets/images/file/file_zip.png"),
        wav: require("../../assets/images/file/file_music.png"),
        mp4: require("../../assets/images/file/file_avi.png")
      }
    };
  },
  computed: {
    // Table displayed columns
    selectedColumnList() {
      return this.$store.getters.selectedColumnList;
    }
  },
  watch: {
    // Listen to changes in the collapse status and store it in sessionStorage to maintain the setting after page refresh
    operaColumnIsFold(newValue) {
      sessionStorage.setItem("operaColumnIsFold", newValue);
    }
  },
  created() {
    this.operaColumnIsFold =
      sessionStorage.getItem("operaColumnIsFold") === "true"; // Read saved status
  },
  methods: {
    // Set file image based on file extension
    setFileImg(row) {
      if (row._directory) {
        // Folder
        return this.fileImgMap.fold;
      } else if (["jpg", "png", "jpeg", "gif"].includes(row.file_type)) {
        // Image types, show thumbnail directly
        return this.getThumbnailPath(row.id);
      } else {
        const fileTypeMap = {
          pptx: "ppt",
          doc: "word",
          docx: "doc",
          xls: "excel",
          xlsx: "excel"
        };
        // Recognizable file types
        return (
          this.fileImgMap[row.file_type] ||
          this.fileImgMap[fileTypeMap[row.file_type]] ||
          this.fileImgMap.other
        );
      }
    },
    // File name click event
    handleFileNameClick(row) {
      // If it is a directory, then go into the directory
      if (row._directory) {
        this.$router.push({
          query: {
            filePath: `${row.file_url}${row.file_name}/`,
            fileType: 0
          }
        });
      } else if (["jpg", "png", "jpeg"].includes(row.file_type)) {
        // Trigger image online viewing
        let data = {
          imgReviewVisible: true,
          imgReviewList: [
            {
              fileUrl: this.getThumbnailPath(row.id),
              downloadLink: this.getThumbnailPath(row.id),
              fileName: row.file_name,
              extendName: row.file_type
            }
          ],
          activeIndex: 0
        };
        this.$store.commit("setImgReviewData", data);
      } else if (["mp3", "flac", "wav"].includes(row.file_type)) {
        // Trigger audio
        let data = {
          audioReviewVisible: true,
          audioReviewList: [
            {
              fileUrl: this.getThumbnailPath(row.id),
              downloadLink: this.getThumbnailPath(row.id),
              filename: row.file_name + "." + row.file_type
            }
          ],
          activeIndex: 0
        };
        this.$store.commit("setAudioReviewData", data);
      } else if (["mp4", "wmv"].includes(row.file_type)) {
        // Video
        let data = {
          visible: true,
          url: this.getThumbnailPath(row.id),
          filename: row.file_name + "." + row.file_type
        };
        this.$store.commit("setVideoReviewData", data);
      } else if (["txt", "js"].includes(row.file_type)) {
        // Text files
        let data = {
          codeReviewVisible: true,
          fileId: row.id,
          filename: row.file_name + "." + row.file_type
        };
        this.$store.commit("setCodeReviewData", data);
      } else if (
        ["ppt", "pptx", "doc", "docx", "xls", "xlsx"].includes(row.file_type)
      ) {
        // Office files
        window.open(this.getDownloadFile(row.id), "_blank");
      }
    },
    // Calculate file size
    calculateFileSize(size) {
      const B = 1024;
      const KB = Math.pow(1024, 2);
      const MB = Math.pow(1024, 3);
      const GB = Math.pow(1024, 4);
      if (!size) {
        return " ";
      } else if (size < KB) {
        return (size / B).toFixed(0) + "KB";
      } else if (size < MB) {
        return (size / KB).toFixed(1) + "MB";
      } else if (size < GB) {
        return (size / MB).toFixed(2) + "GB";
      } else {
        return (size / GB).toFixed(3) + "TB";
      }
    },
    // Table row selection event, selection is the selected table row data
    handleSelectRow(selection) {
      this.$emit("handleSelectFile", true, selection); // true/false for batch move/single file operation; selection is the selected table row data
    },
    // Delete button - Click event
    handleClickDelete(row) {
      this.$confirm(
        "This operation will delete the file, but you can still find it in the recycle bin. Do you want to continue?",
        "Warning",
        {
          confirmButtonText: "Confirm",
          cancelButtonText: "Cancel",
          type: "warning"
        }
      )
        .then(() => {
          deleteFile(row).then(res => {
            console.log(res);
            this.$emit("getTableData"); // Refresh file list
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Deletion cancelled"
          });
        });
    },
    // Move button - Click event
    handleClickMove(row) {
      this.$emit("handleSelectFile", false, row); // true/false for batch move/single file operation; row is the current row file data
      this.$emit("handleMoveFile", true); // true/false to open/close the move file dialog
    },
    // Rename button - Click event
    handleClickRename(row) {
      const fileName = row.file_name;
      this.$prompt("Please enter a new file name", "Prompt", {
        confirmButtonText: "Confirm",
        cancelButtonText: "Cancel",
        inputValue: fileName,
        inputPattern: /\S/, // File name can't be empty
        inputErrorMessage: "Please enter a file name",
        closeOnClickModal: false
      })
        .then(({ value }) => {
          renameFile({
            ...row,
            file_name: value
          }).then(res => {
            console.log(res);
            this.$emit("getTableData"); // Refresh file list
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Cancelled"
          });
        });
    },
    // Download button
    getDownloadPath(row) {
      return this.getDownloadFile(row.id);
    }
  }
};
</script>

<style lang="stylus" scoped>
@import '~@/assets/style/mixins.styl';

.file-table {

  // 调整滚动条样式
  >>> .el-table__body-wrapper {
    setScrollbar(8px, #EBEEF5, #909399);
  }
}

// 表格操作列-表头图标样式调整
.el-icon-circle-plus, .el-icon-remove {
  margin-left: 8px;
  cursor: pointer;
  font-size: 16px;

  &:hover {
    opacity: 0.5;
  }
}
</style>
