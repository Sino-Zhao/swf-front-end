<template>
  <div>
    <div id="title">购买服务</div>
    <el-card>
      <el-table :data="serviceList.slice((currentPage-1)*pageSize,currentPage*pageSize)" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="服务名称" prop="name"></el-table-column>
        <el-table-column label="最大并发实例数" prop="limit"></el-table-column>
        <el-table-column label="服务级别" prop="level"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-tooltip effect="dark" content="购买服务" placement="top-start" :enterable="false">
                <!-- 购买服务按钮 -->
                <el-button type="primary" @click="editDialogVisible = true">购买服务</el-button>
              </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页组件 -->
      <el-pagination 
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[1, 5, 10, 20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="serviceList.length">
      </el-pagination>
    </el-card>

    <!-- 购买服务表单 -->
    <el-dialog title="购买服务" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form label-width="70px" ref="importFormRef">
        <el-form-item label="流程名称" prop="processName">
            <el-input v-model="processName"></el-input>
        </el-form-item>
      
        <el-form-item label="上传文件" prop="uploadFile">
          <el-upload
            class="upload-demo"
            ref="upload"
            action="#"
            :http-request="submitUpload"
            :before-upload="beforeUpload"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :name="fileName"
            :limit="1"
            :file-list="fileList"
            :auto-upload="false">
            <el-button slot="trigger" size="small" type="primary">部署流程</el-button>
            <!-- <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传到服务器</el-button> -->
            <div slot="tip" class="el-upload__tip">只能上传bpmn/zip文件</div>
          </el-upload>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="submitImportForm">确 认</el-button>
      </span>
    </el-dialog>

  </div>
</template>
<script>
import Cookie from "js-cookie";
export default {
  created() {
    this.getServiceList();
  },
  data() {
    return {
      processName: "",
      fileName: "processName",
      fileList: [],
      editDialogVisible: false,
      username: Cookie.get('username'),
      serviceList: [],
      currentPage: 1,
      pageSize: 10,
    };
  },
  methods: {
    async getServiceList() {
      const {data:res} = await this.$http.get("services");
      if (res.flag !== 200) return this.$message.error("获取服务失败！");
      this.serviceList = res.services;
    },
    async submitUpload(param) {
      // console.log(param.file)
      const params = new FormData()
      params.append('username',this.username)
      // console.log(params.get('username'))
      params.append('processFile',param.file)
      // console.log(params.get('processFile'))
      params.append('processName',this.processName)
      // console.log(params.get('processName'))
      // const {data:res} = await this.$http.post("processDefinition/uploadStreamAndDeployment?processName=" + this.processName + "&username=" + this.username, filelist);
      const {data:res} = await this.$http.post("processDefinition/uploadStreamAndDeployment", params);
      if (res.msg != "success") {
        return this.$message.error("部署流程定义失败！");
      }
      this.editDialogVisible = false;
      return this.$message.success("部署流程定义成功！");
    },
    beforeUpload(file) {},
    handleRemove(file, fileList) { console.log(file, fileList); },
    handlePreview(file) { console.log(file); },
    showEditDialog() { this.editDialogVisible = true; },
    editDialogClosed() { this.editDialogVisible = false;},
    async submitImportForm() {
      this.$refs.upload.submit()
    },
    handleSizeChange(val) {
      this.currentPage = 1;
      this.pageSize = val;
    },
    handleCurrentChange(val) {
      this.currentPage = val;
    },
  }
}
</script>
<style scoped>
#title{
  font-family: "Helvetica Neue",Helvetica,"PingFang SC","Hiragino Sans GB","Microsoft YaHei","微软雅黑",Arial,sans-serif;
  text-align: center;
  font-size: 2em;
  margin-bottom: 0.5em;
}
</style>