<template>
  <el-container>
    <el-header>后台管理系统</el-header>
    <el-container>
      <el-aside width="200px">Aside</el-aside>
      <el-main>
        <el-button class="addbtn" @click="showdialog" plain>添加按钮</el-button>
        <el-table :data="tableData" style="width: 100%">
          <el-table-column prop="message" label="备注" width="180"></el-table-column>
          <el-table-column prop="url" label="网址" width="180"></el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-pagination
          layout="prev, pager, next"
          :total="total"
          :page-size="pageSize"
          @current-change="changepage"
        ></el-pagination>
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="changepage"
          :current-page.sync="page"
          :page-sizes="[1,2, 3, 4, 5]"
          :page-size="1"
          layout="sizes, prev, pager, next"
          :total="total"
        ></el-pagination>
        <el-dialog title="添加内容" :visible.sync="dialogFormVisible">
          <el-form :model="form">
            <el-form-item label="备注" :label-width="formLabelWidth">
              <el-input v-model="form.message" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="网址" :label-width="formLabelWidth">
              <el-input v-model="form.url" autocomplete="off"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="add">确 定</el-button>
          </div>
        </el-dialog>
        <el-dialog title="编辑内容" :visible.sync="dialogFormVisible1">
          <el-form :model="form">
            <el-form-item label="备注" :label-width="formLabelWidth">
              <el-input v-model="form.message" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="网址" :label-width="formLabelWidth">
              <el-input v-model="form.url" autocomplete="off"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisible1 = false">取 消</el-button>
            <el-button type="primary" @click="edit">确 定</el-button>
          </div>
        </el-dialog>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      tableData: [],
      pageSize: 1,
      page: 1,
      total: 0,
      dialogFormVisible: false,
      dialogFormVisible1: false,
      form: {
        message: "",
        url: ""
      },
      formLabelWidth: "120px",
      updataid: 0
    };
  },
  created() {
    this.getVal();
  },
  methods: {
    async handleEdit(index, row) {
      this.dialogFormVisible1 = true;
      this.updataid = row.id;
      let result = await this.axios.get(`/api/get?id=${row.id}`);
      this.form.message = result.data.data[0].message;
      this.form.url = result.data.data[0].url;
    },
    handleDelete(index, row) {
      this.$confirm("此操作将永久删除该文件, 是否继续?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          this.axios.get(`/api/del?id=${row.id}`).then(res => {
            if (res.data.code === 1) {
              this.$message({
                type: "success",
                message: "删除成功!"
              });
              // this.getVal();
            } else {
              this.$message({
                type: "info",
                message: "已取消删除"
              });
            }
          });
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除"
          });
        });
    },
    async getVal() {
      let result = await this.axios.get(
        `/api/list?pageSize=${this.pageSize}&page=${this.page}`
      );
      this.tableData = result.data.data;
      this.total = result.data.total;
    },
    async changepage(currentpage) {
      this.page = currentpage;
      this.getVal();
    },
    showdialog() {
      this.dialogFormVisible = true;
    },
    async add() {
      let { message, url } = this.form;
      let result = await this.axios.post("/api/add", { message, url });
      if (result.data.code === 0) {
        this.$message.error("网址已存在");
      } else {
        this.$message({
          message: "添加成功",
          type: "success"
        });
      }
      this.reset();
    },
    reset() {
      this.dialogFormVisible = false;
      this.dialogFormVisible1 = false;
      this.form.message = "";
      this.form.url = "";
      this.getVal();
    },
    async edit() {
      let result = await this.axios.post(`/api/updata`, {
        id: this.updataid,
        message: this.form.message,
        url: this.form.url
      });
      if (result.data.code === 0) {
        this.$message.error("信息无法改变");
      } else {
        this.$message({
          message: "更改成功",
          type: "success"
        });
      }
      this.reset();
    },
    handleSizeChange(pageSize){
      console.log(pageSize)
      this.pageSize = pageSize;
      this.getVal();
    }
  }
};
</script>

<style scoped>
.addbtn {
  float: left;
  margin: 10px 0;
}
.el-container,
.is-vertical {
  width: 100%;
  height: 100%;
}

.el-header,
.el-footer {
  background-color: #b3c0d1;
  color: #333;
  text-align: center;
  line-height: 60px;
}

.el-aside {
  background-color: #d3dce6;
  color: #333;
  text-align: center;
  line-height: 200px;
}

.el-main {
  background-color: #e9eef3;
  color: #333;
}

body > .el-container {
  margin-bottom: 40px;
}
</style>
