<template>
  <div id="wrapper">
    <main class="con">
      <el-row style="margin-bottom:20px;">
        <el-col :span="20">
          <el-radio v-model="radio" label="create">新建表</el-radio>
          <el-radio v-model="radio" label="2">修改表</el-radio>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="outputFile()">开始导出</el-button>
        </el-col>
      </el-row>
      <el-row style="margin-bottom:20px;">
        <el-col :span="4">
          <el-button type="primary" @click="readFile()">选择模板文件</el-button>
        </el-col>
        <el-col :span="18">
          <el-input placeholder="模板文件路径" v-model="path" :disabled="true"></el-input>
        </el-col>
      </el-row>
      <el-row style="margin-bottom:20px;">
        <el-col :span="4">
          <el-button type="primary" @click="selectPath()">选择导出路径</el-button>
        </el-col>
        <el-col :span="18">
          <el-input placeholder="导出路径" v-model="outputpath" :disabled="false"></el-input>
        </el-col>
      </el-row>
      <el-row style="margin-bottom:20px;">
        <el-col :span="8">
          <el-input v-model="name" placeholder="请输入" clearable>
            <template slot="prepend">表名</template>
          </el-input>
        </el-col>
      </el-row>

      <div class="row" v-for="(item,index) in list" :key="index">
        <el-col :span="5">
          <el-input v-model="item.field" placeholder="请输入" clearable>
            <template slot="prepend">字段名</template>
          </el-input>
        </el-col>
        <el-col :span="5">
          <el-input v-model="item.type" placeholder="请选择类型">
            <el-select v-model="item.type" slot="prepend" placeholder="请选择">
              <el-option label="String" value="String"></el-option>
              <el-option label="Array" value="Array"></el-option>
              <el-option label="Object" value="Object"></el-option>
            </el-select>
          </el-input>
        </el-col>
        <el-col :span="5">
          <el-input v-model="item.notes" placeholder="请输入" clearable>
            <template slot="prepend">注释</template>
          </el-input>
        </el-col>
        <el-col :span="5">
          <el-input v-model="item.default" placeholder="请输入" clearable>
            <template slot="prepend">默认值</template>
          </el-input>
        </el-col>
        <el-col :span="3">
          <el-switch v-model="item.required" inactive-text="是否必填"></el-switch>
        </el-col>
        <el-col :span="1">
          <i class="el-icon-delete" @click="deleteItem(index)"></i>
        </el-col>
      </div>
      <el-row type="flex" justify="center">
        <el-button type="primary" @click="addItem()">添加一行</el-button>
      </el-row>
    </main>
  </div>
</template>

<script>
// import SystemInformation from "./LandingPage/SystemInformation";
// import LarInput from "./LarInput/LarInput";
const { dialog } = require("electron").remote;
const fs = require("fs");
export default {
  name: "landing-page",
  data() {
    return {
      radio: "create",
      name: "", //表名
      list: [],
      template: {
        field: "",
        type: "String",
        notes: "",
        default: "",
        required: true
      },
      temfile: null, //读取的文件内容
      path: "", //读取的文件的额路径
      outputpath: "" //导出的路径
    };
  },
  components: {},
  watch: {
    list: {
      handler(list, oldlist) {
        if (list.length <= 0) {
          this.addFun();
        } else {
          if (list[list.length - 1].field != "") {
            this.addFun();
          }
        }
        console.log(list)
      },
      immediate: true,
      deep: true
    }
  },
  created() {},
  methods: {
    addItem() {
      this.addFun();
    },
    deleteItem(index) {
      this.list.splice(index, 1);
    },
    addFun() {
      let obj = Object.assign({}, this.template);
      this.list.push(obj);
    },
    // 读取文件
    readFile() {
      let openfile = dialog.showOpenDialog({ properties: ["openFile"] });
      if (!openfile) {
        return this.$message("操作失败");
      }
      this.path = openfile[0];
      // console.log(path.lastIndexOf("/"));
      // console.log(path);
      let buf = new Buffer(1024 * 1024);
      fs.open(this.path, "r+", (err, fd) => {
        if (err) {
          return console.log(err);
        }
        fs.read(fd, buf, 0, buf.length, 0, (err, bytes) => {
          if (err) {
            console.log(err);
          }
          if (bytes > 0) {
            this.temfile = buf.slice(0, bytes).toString();
          }
        });
        // 关闭文件
        fs.close(fd, function(err) {
          if (err) {
            console.log(err);
          }
        });
      });
    },
    // 选择导出路径
    selectPath() {
      if (this.temfile === null) {
        return this.$message("未选择模板文件");
      }
      let dirpath = dialog.showOpenDialog({
        properties: ["openFile", "openDirectory"]
      });
      if (!dirpath) {
        return this.$message("操作失败");
      }
      this.outputpath = dirpath[0];
    },
    // 导出文件
    outputFile() {
      if (this.temfile === null) {
        return this.$message("未选择模板文件");
      }
      if (this.outputpath === "") {
        return this.$message("未选择导出路径");
      }
      if (this.name === "") {
        return this.$message("请先填写表名");
      }
      if (this.list.length - 1 === 0) {
        return this.$message("请先填写字段信息");
      }
      this.toCreate();
      fs.writeFile(
        this.outputpath + "/" + this.getFileName(),
        this.temfile,
        err => {
          if (err) {
            console.log(err);
          } else {
            this.$message({
              message: "保存成功",
              type: "success"
            });
          }
        }
      );
    },
    // 创建生成内容
    toCreate() {
      //替换主要内容
      this.temfile = this.temfile.replace(
        /\{TableName\}/g,
        this.toUpper("_" + this.name, "_")
      );
      this.temfile = this.temfile.replace(/\{table_name\}/g, this.name);
      let str = "";
      this.list.map((value, index) => {
        if (value.field != "") {
          str += `
          $table->${value.type}('${value.field}','${value.default}')->comment("${value.notes}")->default(${value.default});
          `;
        }
      });
      this.temfile = this.temfile.replace(/\{tocreate\}/g, str);
    },
    // 输出首字母大写
    toUpper(str, sl) {
      let arr = str.split(sl);
      let outstr = "";
      arr.forEach(item => {
        outstr += item.charAt(0).toUpperCase() + item.slice(1);
      });
      return outstr;
    },
    // 生成文件名
    getFileName() {
      let date = new Date();
      let year = date.getFullYear() + "";
      let month = date.getMonth() + 1 + "";
      let day = date.getDate() + "";
      let udate = date - new Date(`${year}/${month}/${day}`);
      let filename = `${year}_${month.padStart(2, "0")}_${day.padStart(
        2,
        "0"
      )}_${Math.ceil(udate / 100)}_${this.radio}_${this.name}_table.php`;

      return filename;
    }
  }
};
</script>
<style lang="scss" scoped>
.con {
  display: flex;
  flex-direction: column;
  .row {
    margin-bottom: 10px;
    display: flex;
    align-items: center;
    .el-col {
      text-align: center;
    }
  }
}
</style>
<style>
@import url("https://fonts.googleapis.com/css?family=Source+Sans+Pro");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Source Sans Pro", sans-serif;
}

#wrapper {
  background: radial-gradient(
    ellipse at top left,
    rgba(255, 255, 255, 1) 40%,
    rgba(229, 229, 229, 0.9) 100%
  );
  overflow-y: scroll;
  height: 100vh;
  padding: 20px 30px;
  width: 100vw;
}

#logo {
  height: auto;
  margin-bottom: 20px;
  width: 420px;
}

main {
  display: flex;
  justify-content: space-between;
}

main > div {
  flex-basis: 50%;
}

.left-side {
  display: flex;
  flex-direction: column;
}

.welcome {
  color: #555;
  font-size: 23px;
  margin-bottom: 10px;
}

.title {
  color: #2c3e50;
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 6px;
}

.title.alt {
  font-size: 18px;
  margin-bottom: 10px;
}

.doc p {
  color: black;
  margin-bottom: 10px;
}

.doc button {
  font-size: 0.8em;
  cursor: pointer;
  outline: none;
  padding: 0.75em 2em;
  border-radius: 2em;
  display: inline-block;
  color: #fff;
  background-color: #4fc08d;
  transition: all 0.15s ease;
  box-sizing: border-box;
  border: 1px solid #4fc08d;
}

.doc button.alt {
  color: #42b983;
  background-color: transparent;
}
</style>
