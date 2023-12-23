<template>
  <div id="global-uploader">
    <!-- Upload component -->
    <uploader
      class="uploader-box"
      ref="uploader"
      :options="options"
      :autoStart="false"
      @file-added="handleFileAdded"
      @file-progress="onFileProgress"
      @file-success="handleFileSuccess"
      @file-error="handleFileError"
    >
      <uploader-unsupport></uploader-unsupport>
      <!-- Select file button -->
      <uploader-btn class="select-file-btn" :attrs="attrs" ref="uploadBtn"
      >Select File
      </uploader-btn
      >
      <!-- Panel for files being uploaded -->
      <uploader-list v-show="panelShow">
        <div class="file-panel" slot-scope="props">
          <div class="panel-title">
            <span class="text">Upload List</span>

            <div class="operate-btn-wrapper">
              <el-button
                type="text"
                title="Close Window"
                icon="el-icon-close"
                @click="handleClosePanel"
              >
              </el-button>
            </div>
          </div>
          <!-- List of files being uploaded -->
          <ul class="file-list">
            <li class="file-item" v-for="file in props.fileList" :key="file.id">
              <uploader-file
                ref="files"
                :class="`file_${file.id}`"
                :file="file"
                :list="true"
              ></uploader-file>
            </li>
            <div class="no-file" v-if="!props.fileList.length">
              <i class="icon-empty-file"></i> No files waiting to be uploaded
            </div>
          </ul>
        </div>
      </uploader-list>
    </uploader>
  </div>
</template>


<script>
import SparkMD5 from "spark-md5";
import { getMultipartUpload, getUpFile } from "@/request/file";

export default {
  name: "FileUploader",
  data() {
    return {
      options: {
        // The target upload URL, which can be a string or a function. If it's a function, it will be passed with Uploader.File instance,
        // current chunk Uploader.Chunk, and whether it is in test mode. The default value is '/'.
        target: function(file, chunkFile, mode) {
          // Entering this method before each chunk upload
          console.log("Entering target");
          console.log("File name: " + file.name);
          console.log("Current chunk sequence number: " + chunkFile.offset);
          console.log("Retrieved chunk upload URL:");
          console.log(file.chunkUrlData);
          const key = "chunk_" + chunkFile.offset; // Key for retrieving chunk link URL
          return file.chunkUrlData[key];
        },
        // Sends a GET request to the server for each chunk to check if it already exists. If implemented on the server side,
        // this will allow continuing uploads after a browser crash or even computer restart. (Default: true)
        testChunks: true,
        // Chunks are divided according to this value. The size of the last chunk will be greater than or equal to this value, but less than twice this value.
        // See this Issue #51, default 1*1024*1024.
        chunkSize: 5 * 1024 * 1024,
        // Forces all chunks to be less than or equal to chunkSize. Otherwise, the last chunk will be greater than or equal to chunkSize. (Default: false)
        forceChunkSize: true,
        // Additional parameters included in multipart POST with data. This can be an object or a function. If it is a function,
        // it will be passed an Uploader.File, an Uploader.Chunk object, and an isTest boolean value (default value {}：)
        query: function(file, chunkFile, mode) {
          const data = { partNumber: chunkFile.offset + 1 };
          return data;
        },
        uploadMethod: "PUT",
        // The method used for uploading, options are multipart or octet, default multipart, refer to multipart vs octet.
        // MiniO chunking can't use form
        method: "octet",
        // Handles request parameters, default function (params) {return params}, usually used to modify parameter names or delete parameters.
        processParams: function(params) {
          return {};
        }
        // headers: {
        //  'Content-Type': 'binary/octet-stream'
        // }
      },
      attrs: {
        accept: "*" // Acceptable file types
      },
      panelShow: false // Whether the file upload panel is shown
    };
  },
  computed: {
    // Global upload component uploader instance
    uploader() {
      return this.$refs.uploader.uploader;
    },
    // Current path
    filePath() {
      return this.$route.query.filePath ? this.$route.query.filePath : "/";
    },
    // Whether the file upload event is triggered
    startUploadFile() {
      return this.$store.state.file.startUploadFile;
    }
  },
  methods: {
    // Trigger the click event of the select file button
    triggerSelectFileClick() {
      this.$refs.uploadBtn.$el.click(); // Trigger the click event of the select file button
    },

    // File added callback function
    handleFileAdded(file) {
      this.panelShow = true; // Show file upload panel
      this.calculateFileMD5(file); // Calculate file MD5 value
      // Calculate MD5
      // Get chunk upload link
      // eslint-disable-next-line no-unused-vars
      const res = this.getChunkUploadUrl(file);
      console.log("File added to check if chunk URL is retrieved");
      console.log(file.chunkUrlData);
    },
    async getChunkUploadUrl(file) {
      // Requesting the server for each chunk's upload link
      console.log(file);
      console.log("Retrieving chunk upload link");
      const fileName = file.name; // File name
      const fileSize = file.size; // File size
      const chunkSize = file.chunks.length; // Number of chunks
      // Request backend to return upload link for each chunk
      // eslint-disable-next-line no-unused-vars
      const res = await getUpFile({
        filename: fileName,
        chunkNumber: chunkSize,
        filePath: this.filePath,
        uploadTime: new Date().getTime(),
        fileSize: fileSize,
        identifier: file.uniqueIdentifier
      })
        .then(function(response) {
          console.log("Retrieved uploadId:" + response.uploadId);
          console.log("Retrieved chunk upload link collection:");
          file.chunkUrlData = response;
          console.log(file.chunkUrlData);
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    /**
     * File upload process callback function
     * @param rootFile The root Uploader.File object of the successfully uploaded file, which should contain or equal the successfully uploaded file
     * @param file The current successful Uploader.File object itself
     * @param chunk The Uploader.Chunk instance, which is the last chunk instance of the file, if you want to get the response code, chunk.xhr.status is it
     */
    onFileProgress(rootFile, file, chunk) {
      // Print information during file upload
      console.log(
        `Uploading ${file.name}, chunk: ${chunk.startByte /
        1024 /
        1024} ~ ${chunk.endByte / 1024 / 1024}`
      );
    },
    /**
     * File upload success callback function
     * @param rootFile
     * @param file
     */
    handleFileSuccess(rootFile, file) {
      console.log("Single file uploaded successfully", arguments);
      // Call backend to merge files
      const fileName = file.name; // File name
      const uploadId = file.chunkUrlData.uploadId; // uploadId
      console.log();
      getMultipartUpload({
        FileName: fileName,
        uploadId: uploadId
      })
        .then(function(response) {
          console.log(response);
        })
        .catch(function(error) {
          console.log(error);
        });
      this.$emit("getTableData");
      console.log("Merge complete");
    },

    // File upload failure callback function
    handleFileError(rootFile, file, responseStr) {
      this.$message({
        message: JSON.parse(responseStr).message,
        type: "error"
      });
    },

    // Calculate file MD5 value
    calculateFileMD5(file) {
      let fileReader = new FileReader();
      let blobSlice =
        File.prototype.slice ||
        File.prototype.mozSlice ||
        File.prototype.webkitSlice;
      let currentChunk = 0;
      const chunkSize = 5 * 1024 * 1024;
      let chunks = Math.ceil(file.size / chunkSize);
      let spark = new SparkMD5.ArrayBuffer();
      file.pause(); // Pause file upload
      loadNext();
      fileReader.onload = e => {
        spark.append(e.target.result);
        if (currentChunk < chunks) {
          currentChunk++;
          loadNext();
        } else {
          let md5 = spark.end();
          this.calculateFileMD5End(md5, file);
          // console.log(
          //   `MD5 calculation complete: ${file.name} \nMD5: ${md5} \nChunks: ${chunks} Size: ${
          //     file.size
          //   } Time: ${new Date().getTime() - time} ms`
          // );
        }
      };
      fileReader.onerror = function() {
        this.$notify({
          title: "Upload error",
          message: `File ${file.name} read error, please check the file`,
          type: "error",
          duration: 2000
        });
        file.cancel(); // Cancel upload and remove from file list.
      };

      function loadNext() {
        let start = currentChunk * chunkSize;
        let end =
          start + chunkSize >= file.size ? file.size : start + chunkSize;
        fileReader.readAsArrayBuffer(blobSlice.call(file.file, start, end));
      }
    },
    // File MD5 calculation complete
    calculateFileMD5End(md5, file) {
      file.uniqueIdentifier = md5;
      file.resume(); // Continue file upload
    },

    // Close upload panel
    handleClosePanel() {
      this.uploader.cancel(); // Cancel file upload
      this.panelShow = false;

      this.$emit("getTableData");
    }
  }
};
</script>


<style lang="stylus" scoped>
#global-uploader {
  position: fixed;
  z-index: 20;
  right: 15px;
  bottom: 15px;

  .uploader-box {
    width: 520px;
  }

  .file-panel {
    background-color: #fff;
    border: 1px solid #e2e2e2;
    border-radius: 7px 7px 0 0;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.2);

    .panel-title {
      display: flex;
      height: 40px;
      line-height: 40px;
      padding: 0 16px;
      border-bottom: 1px solid #ddd;

      .text {
        padding-left: 0;
        margin-bottom: 0;
        font-size: 16px;
        color: #303133;
      }

      .operate-btn-wrapper {
        flex: 1;
        text-align: right;

        >>> .el-button {
          *:hover {
            opacity: 0.5;
          }

          i[class^=el-icon-] {
            color: #000;
          }
        }
      }
    }

    .file-list {
      position: relative;
      height: 240px;
      overflow-x: hidden;
      overflow-y: auto;
      background-color: #fff;
      font-size: 14px;

      .file-item {
        background-color: #fff;

        >>> .uploader-file-icon {
          display: none;
        }
      }
    }
  }

  .no-file {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    font-size: 16px;
  }

  /deep/.uploader-file-actions > span {
    margin-right: 6px;
  }
}

/* 隐藏上传按钮 */
.select-file-btn {
  display: none;
}
</style>
