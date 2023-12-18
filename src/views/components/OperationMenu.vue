<template>
  <div class="operation-menu-wrapper">
    <!-- Button group -->
    <el-button-group class="operate-group">
      <el-dropdown class="create-drop" trigger="hover" v-if="fileType < 7">
        <el-button
          size="mini"
          type="primary"
          :disabled="fileType !== 0"
          icon="el-icon-plus"
        >Create<i class="el-icon-arrow-down el-icon--right"></i
        ></el-button>
        <el-dropdown-menu slot="dropdown">
          <el-dropdown-item @click.native="addFolderDialog.visible = true">
            <div class="img-text-wrapper">
              <img src="../../assets/images/file/dir.png" class="imgD" />
              Create Folder
            </div>
          </el-dropdown-item>
          <hr color="#eee" style="width: 120px" />
          <el-dropdown-item>
            <div class="img-text-wrapper">
              <img
                src="../../assets/images/file/file_word.svg"
                class="imgS"
              />Word Document
            </div>
          </el-dropdown-item>
          <el-dropdown-item>
            <div class="img-text-wrapper">
              <img
                src="../../assets/images/file/file_excel.svg"
                class="imgS"
              />Excel Spreadsheet
            </div>
          </el-dropdown-item>
          <el-dropdown-item>
            <div class="img-text-wrapper">
              <img
                src="../../assets/images/file/file_ppt.svg"
                class="imgS"
              />PPT Presentation
            </div>
          </el-dropdown-item>
        </el-dropdown-menu>
      </el-dropdown>
      <el-button
        size="mini"
        type="primary"
        icon="el-icon-upload2"
        @click="handleUploadFileClick()"
        :disabled="fileType !== 0"
        v-if="fileType < 7"
      >Upload File
      </el-button
      >

      <el-button
        size="mini"
        type="primary"
        icon="el-icon-upload2"
        @click="handleEmptyTrash()"
        v-if="fileType === 8"
      >Empty Recycle Bin
      </el-button
      >
      <el-button
        size="mini"
        type="primary"
        icon="el-icon-upload2"
        @click="handleRestoreTrash()"
        v-if="fileType === 8"
      >Restore Recycle Bin
      </el-button
      >

      <template v-if="operationFileList.length && fileType < 7">
        <el-button
          size="mini"
          type="primary"
          icon="el-icon-delete"
          @click="handleDeleteFileClick()"
        >Delete
        </el-button
        >
        <el-button
          size="mini"
          type="primary"
          icon="el-icon-rank"
          :disabled="fileType !== 0"
          @click="handleMoveFileClick()"
        >Move
        </el-button
        >
        <el-button
          size="mini"
          type="primary"
          icon="el-icon-download"
          @click="handleDownloadFileClick()"
        >Download
        </el-button
        >
      </template>
      <template v-if="operationFileList.length && fileType === 8">
        <el-button
          size="mini"
          type="primary"
          icon="el-icon-delete"
          @click="handleEmptyFilesBatch()"
        >Permanently Delete
        </el-button
        >
        <el-button
          size="mini"
          type="primary"
          icon="el-icon-delete"
          @click="handRestoreFilesBatch()"
        >Batch Restore
        </el-button
        >
      </template>
    </el-button-group>

    <!-- Dialog - Create Folder -->
    <el-dialog
      title="Create Folder"
      width="600px"
      :visible.sync="addFolderDialog.visible"
      @closed="$refs.addFolderForm.resetFields()"
    >
      <el-form
        :model="addFolderForm"
        :rules="addFolderRules"
        label-position="top"
        ref="addFolderForm"
      >
        <el-form-item label="Folder Name" prop="name">
          <el-input v-model="addFolderForm.name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addFolderDialog.visible = false">Cancel</el-button>
        <el-button
          type="primary"
          :loading="addFolderDialog.loading"
          @click="handleAddFolderSubmit('addFolderForm')"
        >Submit</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {
  batchDeleteFile,
  batchEmptyFile,
  batchRestoreFile,
  getEmptyTrash,
  getRestoreTrash,
  makerDir
} from "@/request/file";

export default {
  name: "OperationMenu",
  props: {
    // File type
    fileType: {
      type: Number,
      required: true
    },
    // File path
    filePath: {
      type: String,
      required: true
    },
    // Selected items in table rows
    operationFileList: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      // Create folder dialog data
      addFolderDialog: {
        visible: false, // Dialog visibility status
        loading: false
      },
      // Create folder form
      addFolderForm: {
        name: ""
      },
      // Create folder form validation rules
      addFolderRules: {
        name: [
          {
            required: true,
            message: "Please enter folder name",
            trigger: "blur, change"
          }
        ]
      }
    };
  },
  methods: {
    // Create folder dialog - Submit button
    handleAddFolderSubmit() {
      this.$refs.addFolderForm.validate(valid => {
        if (valid) {
          this.addFolderDialog.loading = true; // Enable loading state for the dialog's confirm button
          // Form validation passed - Call the create folder interface
          makerDir({
            fileName: this.addFolderForm.name,
            filePath: this.filePath // File path
            // IsDir: 1 // Whether it is a folder: 1 for folder, 0 for file
          }).then(
            resp => {
              this.addFolderDialog.loading = false; // Disable loading state for the dialog's confirm button
              if (resp.code === 200) {
                this.addFolderDialog.visible = false; // Close the dialog
                this.$emit("getTableData"); // Refresh file list
              } else {
                this.$message.warning(resp.message);
              }
            },
            error => {
              this.addFolderDialog.loading = false; // Disable loading state for the dialog's confirm button
              console.log(error);
            }
          );
        } else {
          return false;
        }
      });
    },
    // Upload file button - Click event
    handleUploadFileClick() {
      this.$emit("handleUploadFile", true);
    },
    // Batch delete file button
    handleDeleteFileClick() {
      this.$confirm(
        "This operation will delete the selected files, but you can still find them in the recycle bin. Do you want to continue?",
        "Warning",
        {
          confirmButtonText: "Confirm",
          cancelButtonText: "Cancel",
          type: "warning"
        }
      )
        .then(() => {
          batchDeleteFile({
            files: JSON.stringify(this.operationFileList),
            filePath: "/"
          }).then(res => {
            if (res.code === 200) {
              this.$emit("getTableData"); // Refresh file list
            } else {
              this.$message.error("Failed" + res.message);
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Deletion cancelled"
          });
        });
    },
    // Move file button - Click event
    handleMoveFileClick() {
      this.$emit("handleSelectFile", true, this.operationFileList);
      this.$emit("handleMoveFile", true); // Open/close the move file dialog
    },
    // Empty recycle bin
    handleEmptyTrash() {
      this.$confirm(
        "This operation will empty the recycle bin. Do you want to continue?",
        "Warning",
        {
          confirmButtonText: "Confirm",
          cancelButtonText: "Cancel",
          type: "warning"
        }
      )
        .then(() => {
          getEmptyTrash({}).then(res => {
            if (res.code === 200) {
              this.$emit("getTableData"); // Refresh file list
            } else {
              this.$message.error("Failed" + res.message);
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Deletion cancelled"
          });
        });
    },
    // Restore recycle bin
    handleRestoreTrash() {
      this.$confirm(
        "This operation will restore the recycle bin. Do you want to continue?",
        "Warning",
        {
          confirmButtonText: "Confirm",
          cancelButtonText: "Cancel",
          type: "warning"
        }
      )
        .then(() => {
          getRestoreTrash({}).then(res => {
            if (res.code === 200) {
              this.$emit("getTableData"); // Refresh file list
            } else {
              this.$message.error("Failed" + res.message);
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Deletion cancelled"
          });
        });
    },
    // Permanently delete
    handleEmptyFilesBatch() {
      this.$confirm(
        "This operation will permanently delete the selected files. Do you want to continue?",
        "Warning",
        {
          confirmButtonText: "Confirm",
          cancelButtonText: "Cancel",
          type: "warning"
        }
      )
        .then(() => {
          batchEmptyFile({
            files: JSON.stringify(this.operationFileList),
            filePath: "/"
          }).then(res => {
            if (res.code === 200) {
              this.$emit("getTableData"); // Refresh file list
            } else {
              this.$message.error("Failed" + res.message);
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "Deletion cancelled"
          });
        });
    },
    // Batch restore
    handRestoreFilesBatch() {
      batchRestoreFile({
        files: JSON.stringify(this.operationFileList),
        filePath: "/"
      }).then(res => {
        if (res.code === 200) {
          this.$emit("getTableData"); // Refresh file list
        } else {
          this.$message.error("Failed" + res.message);
        }
      });
    }
  }
};
</script>

<style>
.img-text-wrapper {
  display: flex;
  align-items: center;
}

.imgD {
  margin-right: 4px;
  height: 30px;
}

.imgS {
  margin-right: 4px;
  height: 23px;
}

.create-drop {
  float: left;
}
</style>
