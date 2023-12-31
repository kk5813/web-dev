<template>
  <div class="global-container mt20">
    <div>
      <SearchTool
        :tableData="tableData"
        :rules="searchRule1"
        v-model="showTable"
        title="专家团队查询"
      >
        <el-button
          @click="newGroup.dialogFlag = true"
          type="success"
          size="small"
          >添加专家团队</el-button
        >
      </SearchTool>
    </div>

    <el-divider></el-divider>
    <div class="heightControl mt20">
      <ListCard
        @discard="discard"
        v-for="item in $utils.pageSlice(showTable, currentPage, pageSize)"
        :key="item.id"
        :cardInfo="item"
        @details="details"
      ></ListCard>
    </div>
    <div class="clearFloat mt20">
      <el-pagination
        style="float: right"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[5, 10, 20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="showTable.length"
      >
      </el-pagination>
    </div>

    <!-- 添加医疗机构对话框 -->
    <el-dialog
      title="添加机构"
      :visible.sync="newGroup.dialogFlag"
      width="700px"
    >
      <div class="dialog">
        <el-form
          label-position="left"
          ref="addForm"
          :show-message="false"
          :inline="false"
          size="small"
          :model="newGroup.info"
        >
          <div class="basicInfo">
            <div class="pic">
              <el-upload
                class="avatar-uploader"
                action="/api/upload"
                :show-file-list="false"
                :on-success="handleAvatarSuccess"
                :before-upload="beforeAvatarUpload"
                :headers="uploadToken"
              >
                <img
                  style="width: 178px; height: 178px; overflow: hidden"
                  v-if="newGroup.info.groupImage"
                  :src="newGroup.info.groupImage"
                  class="avatar"
                />
                <div v-else style="width: 190px; height: 190px">
                  <i class="el-icon-plus avatar-uploader-icon"></i>
                </div>
              </el-upload>
            </div>

            <div class="info">
              <el-form-item prop="groupName" label="团队名称">
                <el-input
                  v-model="newGroup.info.groupName"
                  style="width: 300px"
                ></el-input>
              </el-form-item>
              <el-form-item prop="groupTel" label="团队擅长">
                <el-input
                  v-model="newGroup.info.groupShanchang"
                  style="width: 300px"
                ></el-input>
              </el-form-item>

              <el-form-item prop="groupTel" label="联系方式">
                <el-input
                  v-model="newGroup.info.groupTel"
                  style="width: 300px"
                ></el-input>
              </el-form-item>
            </div>
          </div>
          <div class="introduction">
            <div><span>机构简介</span></div>
            <div>
              <el-form-item prop="groupName">
                <el-input
                  type="textarea"
                  :rows="2"
                  v-model="newGroup.info.groupIntroduction"
                ></el-input>
              </el-form-item>
            </div>
          </div>
        </el-form>

        <span slot="footer" class="dialog-footer clearFloat">
          <el-button
            size="small"
            style="margin-top: 20px; float: right"
            type="primary"
            @click="createGroup"
            >确 定</el-button
          >
        </span>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import ListCard from "../components/ListCard.vue";
import {
  getExpertGroupsInfo,
  addExpertGroupsInfo,
  delExpertGroupsInfo,
} from "@api/manage/expertManage.js";
export default {
  components: {
    ListCard,
  },
  data() {
    return {
      uploadToken: {
        Authorization: localStorage.getItem("token"),
      },
      tableData: [],
      currentPage: 1,
      pageSize: 5,
      showTable: [],
      searchRule1: [
        {
          label: "团队名称",
          value: "name",
          type: "input",
        },
      ],
      newGroup: {
        dialogFlag: false,
        info: {
          groupName: "第三人民医院神经科1",
          groupShanchang: "神经损伤、心理干预",
          groupIntroduction:
            "成都市第三人民医院是一家成成都市第三人民医院是一家成",
          groupImage:
            "https://www.nowhealth.top:3000/readload?url=chatfile/784bf7fb71b9d1f436ec70a81af80279.png",
          groupTel: 123,
        },
      },
    };
  },
  methods: {
    handleAvatarSuccess(res, file) {
      console.log(res);
      this.newGroup.info.Image = res.readloadurl;
    },
    beforeAvatarUpload(file) {
      const isJPG = file.type === "image/jpeg";
      const isLt2M = file.size / 1024 / 1024 < 2;

      if (!isJPG) {
        this.$message.error("上传头像图片只能是 JPG 格式!");
      }
      if (!isLt2M) {
        this.$message.error("上传头像图片大小不能超过 2MB!");
      }
      return isJPG && isLt2M;
    },

    handleSizeChange(val) {
      this.pageSize = val;
    },
    handleCurrentChange(val) {
      this.currentPage = val;
    },
    createGroup() {
      addExpertGroupsInfo(this.newGroup.info).then((res) => {
        if (res) {
          this.$message.success("创建成功");
          this.getInfo();
          this.newGroup.dialogFlag = false;
        } else {
          this.$message.error("创建失败");
        }
      });
    },
    details(val) {
      this.$router.push({
        path: "/test1/groupdetails",
        query: {
          id: val,
        },
      });
    },
    discard(id) {
      this.$confirm("此操作将解散医疗机构，是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          delExpertGroupsInfo(id).then((res) => {
            if (res) {
              this.getInfo();
              this.$message.success("成功解散");
            } else {
              this.$message.error("解散失败");
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消",
          });
        });
    },
    getInfo() {
      getExpertGroupsInfo().then((res) => {
        this.tableData = res.map((item) => {
          return {
            id: item.ExpertID,
            img: item.ExpertImage,
            name: item.ExpertName,
            introduction: item.ExpertIntroduction,
          };
        });
      });
    },
  },
  created() {
    this.getInfo();
  },
};
</script>
<style scoped lang="scss">
.heightControl {
  max-height: 800px;
  overflow: auto;
}
.dialog {
  width: 100%;
  .basicInfo {
    height: 200px;
    padding-left: 20px;
    display: flex;
    .pic {
      width: 200px;
      flex-shrink: 0;
      overflow: hidden;
      img {
        width: 100%;
        height: 100%;
      }
    }
    .info {
      padding-left: 20px;
      width: 100%;
      p {
        font-size: 18px;
        margin-bottom: 10px;
        .lable {
          font-weight: bolder;
          color: #1c7e7c;
        }
      }
    }
  }
  .introduction {
    padding-left: 20px;
    padding-top: 20px;
    width: 100%;

    .lable {
      font-size: 18px;
      font-weight: bolder;
      color: #1c7e7c;
    }
  }
}
</style>


<style>
.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}
</style>