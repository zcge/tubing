<template>
<div style="padding:20px;">
  <el-tabs v-model="tabName" type="card">
    <el-tab-pane label="兴趣组编辑" name="first">
      <div class="box">
        <el-form ref="form" :model="form" label-width="80px">
          <el-form-item label="名称">
            <el-input v-model="form.groupName"></el-input>
          </el-form-item>
          <el-form-item label="头像">
            <div class="upload">
              <img :src="form.avatarUrl" />
            </div>
            <el-upload class="upload-demo" :action="actionUrl" :on-success="successUpload" :file-list="fileList">
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSubmit">修改</el-button>
            <el-button @click="onCancle">取消</el-button>
          </el-form-item>
        </el-form>
      </div>
    </el-tab-pane>
    <el-tab-pane label="兴趣组成员" name="second">
      <div style="margin:20px;">
        <el-row>
          <el-col :span="16">
            <el-form :inline="true">
              <el-form-item>
                <el-input v-model="form_add.members" auto-complete="off" placeholder="多个账户用@分割"></el-input>
              </el-form-item>
              <el-form-item>
                <el-button type="primary" @click="()=>submitForm()">添加</el-button>
              </el-form-item>
            </el-form>
          </el-col>
        </el-row>
        <el-table :data="list" style="width: 100%">
          <el-table-column prop="username" label="源平台账号">
            <template slot-scope="scope">
              <a href="javascript:void(0);" type="text" @click="()=>gotoConetent(scope.row.username)">{{scope.row.username}}</a>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="180">
            <template slot-scope="scope">
                    <el-button type="text" @click="()=>remove(scope.row)">删除</el-button>
</template>
            </el-table-column>
        </el-table>
    </div>
    <div style="margin:20px;" v-if="pageInfo.total">
        <el-pagination layout="sizes,prev, pager, next,jumper" :current-page="pageInfo.page" @size-change="handleSizeChange" :page-sizes="[10, 20, 30, 40,50]" :total="pageInfo.total" :page-size="pageInfo.size" @current-change="handelPageChange" style="text-align: right;">
        </el-pagination>
    </div>
    </el-tab-pane>
    <!-- <el-tab-pane label="少量增加兴趣组成员" name="third">
      <div style="width:300px;padding-top:20px;">
         <el-form v-model="form_add" ref="numberValidateForm" label-width="100px" class="demo-ruleForm">
            <el-form-item label="成员账户">
                <el-input v-model="form_add.members" auto-complete="off" placeholder="多个账户用@分割"></el-input>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="()=>submitForm()">提交</el-button>
            </el-form-item>
        </el-form>
      </div>
    </el-tab-pane> -->
  </el-tabs>

    
    
</div>
</template>

<script>
import api from "@/api";
export default {
  data() {
    return {
      form_add: {
        members: ""
      },
      tabName: "first",
      actionUrl: "",
      fileList: [],
      list: [],
      uploading: false,
      groupId: "",
      form: {
        groupName: "",
        description: "", //简介
        avatarUrl: "" //头像
      },
      pageInfo: {
        total: 0,
        page: 1,
        size: 50
      }
    };
  },
  mounted() {
    this.actionUrl = api.getActionAvatarUrl;
    this.groupId = this.$route.query.id;
    this.queryDetail();
    this.queryList();
  },
  methods: {
    handleSizeChange(num){
         this.pageInfo.size = num;
         this.pageInfo.page = 1;
         this.queryList();
      },
    submitForm() {
      if (this.form_add.members) {
        api
          .addInterestMembers({
            ...this.form_add,
            groupId: this.groupId
          })
          .then(e => {
            if (e.data.success && e.data.data) {
              this.$message.success("创建成功");
              this.queryList();
            } else {
              this.$message.error("创建失败");
            }
          });
      } else {
        this.$message.error("账户信息请填写完整");
      }
    },
    successUpload(response, file, fileList) {
      if (response.success) {
        this.$message.success("上传成功");
        this.form.avatarUrl = response.data;
      } else {
        this.$message.error("上传失败");
      }
    },
  gotoConetent(username){
    this.$router.push({name:"ContentManagement",query:{username}})
  },
    queryList() {
      api
        .interestAccountList({
          groupId: this.groupId,
          page: this.pageInfo.page - 1,
          size: this.pageInfo.size
        })
        .then(e => {
          if (e.data.success) {
            this.list = e.data.data.content;
            this.pageInfo.total = e.data.data.totalElements;
          } else {
            this.$message.error("查询失败");
          }
        });
    },
    handelPageChange(num) {
      this.pageInfo.page = num;
      this.queryList();
    },
    queryDetail() {
      api
        .detailInterestGroup({
          id: this.groupId
        })
        .then(e => {
          if (e.data.success) {
            this.form = e.data.data;
          } else {
            this.$message.error("查询失败");
          }
        });
    },
    onSubmit() {
      let groupBrief = this.form.description;
      let groupName = this.form.groupName;
      let groupId = this.groupId;
      let groupProfile = this.form.avatarUrl;
      api
        .updateInterestGroupInfo({
          groupBrief,
          groupName,
          groupId,
          groupProfile
        })
        .then(e => {
          console.log(e);
          if (e.data.success) {
            this.$message.success("更新成功");
          } else {
            this.$message.error("更新失败");
          }
        });
    },
    onCancle() {
      this.$router.go(-1);
    },
    uploadImg(event) {
      var file = event.target.files[0];
      console.log(file);
      if (file) {
        this.uploading = true;
        api
          .updateInterestGroupAvatar({
            file: file
          })
          .then(e => {
            this.uploading = false;
          });
      }
    },
    remove(row) {
      this.$confirm("删除操作, 是否继续?", "提示", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
        .then(() => {
          api
            .removeAccount({
              groupId: this.groupId,
              accountId: row.id
            })
            .then(e => {
              if (e.data.success) {
                this.$message.success("删除成功");
                this.queryList();
              } else {
                this.$message.error("删除失败");
              }
            });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    }
  }
};
</script>

<style lang="less" scoped>
.box {
  margin: 20px;
  width: 500px;
}

.upload {
  width: 100px;
  height: 100px;
  background: #eee;
  position: relative;
  img {
    top: 0;
    position: absolute;
    width: 100px;
    height: 100px;
  }
  .mark {
    top: 0;
    position: absolute;
    width: 100px;
    height: 100px;
    background: rgba(0, 0, 0, 0.5);
    text-align: center;
    line-height: 100px;
    color: #fff;
  }
  input {
    position: absolute;
    display: block;
    width: 100%;
    height: 100%;
    opacity: 0;
  }
}
</style>
