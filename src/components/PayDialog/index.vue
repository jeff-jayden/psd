<template>
  <base-dialog dialog-title="title">
    <template #header>
      <el-row>
        <el-col :span="20">订单支付</el-col>
        <el-col :span="4">
          <svg-icon icon-class="close" @click="closeDialog"></svg-icon>
        </el-col>
      </el-row>
    </template>
    <template #main>
      <div class="header-info">
        <el-row>
          <el-col :span="12">
            <span>姓名：</span>
            <span>李晓</span>
          </el-col>
          <el-col :span="12">
            <span>产品名称：</span>
            <span>销售展示</span>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <span>公司：</span>
            <span>花花信息技术有限公司</span>
          </el-col>
          <el-col :span="12">
            <span>套餐方式：</span>
            <span>按次收费</span>
          </el-col>
        </el-row>
        <el-row>
          <el-col :span="12">
            <span>电话：</span>
            <span>15826354152</span>
          </el-col>
          <el-col :span="12"></el-col>
        </el-row>
      </div>
      <el-row class="pay-type">
        <el-col :span="24">
          <span>支付方式</span>
        </el-col>
      </el-row>
      <el-row class="pay-type-switch">
        <el-col :span="24">
          <el-button>支付宝</el-button>
        </el-col>
      </el-row>
      <el-row class="pay-qrcode">
        <el-col :span="24">
          <el-image
            style="width: 250px; height: 250px"
            :src="imgUrl"
            fit="contain"
          ></el-image>
        </el-col>
      </el-row>
    </template>
    <template #footer>
      <el-row>
        <el-col :span="24">
          <el-button @click="closeDialog">取消</el-button>
          <el-button type="primary" @click="checkPaySucc">确定</el-button>
        </el-col>
      </el-row>
    </template>
  </base-dialog>
</template>

<script>
import axios from "axios";
import BaseDialog from "@/components/BaseDialog";
import { mapState } from "vuex";
export default {
  name: "PayDialog",
  components: {
    BaseDialog,
  },
  props: ["IsPayDialogShow", "work"],
  /*
  * 是在Vue应用中导入vuex库中的mapState函数，
  * 用于在组件中映射vuex中的状态到组件的计算属性中
  * mapState函数可以简化组件中对vuex状态的访问，
  * 使得组件代码更加简洁易懂
  *
  * mapState函数接受一个对象作为参数，
  * 对象的属性名是组件中的计算属性名，
  * 属性值是一个函数，用于返回vuex中的状态
  *
  * ...mapState通常用于将vuex中的状态映射到组件的计算属性中，
  * 使得组件可以直接访问vuex中的状态
  * */
  computed: {
    ...mapState({
      id: "files.fileMsg.data.id",
      code: "files.fileMsg.data.code",
      workObj: (s) => s.compare.workObj,
    }),
  },
  mounted() {
    let timer = setInterval(() => {
      if (this.workObj.id) {
        this.imgUrl = this.$api.reqQRCodeUrl(this.workObj.id);
        clearInterval(timer);
      }
    }, 100);
  },
  methods: {
    closeDialog() {
      this.$emit("on-close");
    },
    formatDatetime(datetime) {
      return new Date(datetime).format("yyyy-MM-dd hh:mm:ss");
    },
    // 支付成功
    IsPaySuccess() {
      setInterval(() => {
        axios
          .get(
            "http://buchitang.top:8081/swagger-ui.html#/compare-controller/getStatusUsingGET"
          )
          .then((response) => {
            if (response.data.id === this.id) {
              // 对比文件渲染
              this.$bus.$emit("paySuccess", this.$store.files.fileMsg);
            }
          });
      }, 1000);
    },
    // 点击确定后
    checkPaySucc() {
      let succ = false;
      this.$api.reqGetCompareOrderStatus(this.workObj.id).then((r) => {
        if (r.status == 200 && r.data.message == "完成") {
          succ = true;
          if (succ) {
            // this.$message({
            //   message: "支付成功",
            //   type: "success",
            // });
            this.$notify.success({
              title: "成功",
              message: "支付成功",
            });
            //刷新workObj数据
            this.$store.dispatch("freshWorkObj", this.workObj.id);
            // this.$store.commit('setWorkStatus',r.data.message)
            this.$bus.$emit("requestUpload");
            this.closeDialog();
          }
        } else {
          this.$message({
            message: r.data.message,
            type: "error",
          });
        }
      });
    },
  },
  data() {
    return {
      publicPath: process.env.BASE_URL,
      imgUrl: "",
    };
  },
};
</script>

<style lang="scss" scoped>
.el-container {
  .el-header {
    .el-row {
      font-weight: bold;
      .el-col {
        .svg-icon {
          cursor: pointer;
        }
        &:first-of-type {
          text-align: left;
        }
        &:last-of-type {
          text-align: right;
        }
      }
    }
  }
  .el-main {
    .header-info {
      .el-row {
        margin-top: 20px;
        .el-col {
          &:nth-of-type(odd) {
            text-align: left;
          }
          &:nth-of-type(even) {
            text-align: right;
          }
          span {
            &:nth-of-type(odd) {
              color: #a8a8aa;
            }
          }
        }
      }
    }
    .pay-type {
      text-align: left;
      font-weight: bold;
      margin: 20px 0;
      padding: {
        left: 5px;
      }
    }
    .pay-qrcode {
      .el-col {
        text-align: left;
      }
      margin: {
        top: 20px;
      }
    }
    .pay-type-switch {
      text-align: left;
    }
    .el-footer {
      margin: {
        // bottom: 10px;
      }
      .el-row {
        .el-col {
          text-align: right;
        }
      }
    }
  }
}
</style>