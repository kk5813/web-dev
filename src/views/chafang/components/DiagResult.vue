<template>
  <div>
    <div class="mainContent">
      <el-collapse v-model="pages.collapse_activeNames">
        <el-collapse-item name="2">
          <template slot="title">
            <h3 class="title">病情概况</h3>
          </template>
          <span style="margin-left: 40px;font-size: 18px">
            {{patientInfo.API_description}}
          </span>
        </el-collapse-item>
        <el-collapse-item name="3">
          <template slot="title">
            <h3 class="title">患者病史</h3>
          </template>
          <div class="history">
            <p>
              既往史：{{
                this.patientInfo.API_history.jiwang || "无"
              }}
            </p>
            <p>
              家族史：{{
                this.patientInfo.API_history.jiazu || "无"
              }}
            </p>
            <p>
              过敏史：{{
                this.patientInfo.API_history.guoming || "无"
              }}
            </p>
          </div>
        </el-collapse-item>
        <el-collapse-item
          v-if="
            API_diagInfo.API_diagResult &&
            API_diagInfo.API_diagResult.length > 0
          "
          name="5"
        >
          <template slot="title">
            <h3 class="title">诊断结论</h3>
          </template>
          <div class="diagResult">
            <div class="text">
              <div class="box">
                <p>{{ API_diagInfo.API_diagResult.join("，") }}</p>
              </div>
            </div>
          </div>
        </el-collapse-item>
        <el-collapse-item
          v-if="
            API_diagInfo.API_treatment.API_description &&
            API_diagInfo.API_treatment.API_description.length > 0
          "
          name="6"
        >
          <template slot="title">
            <h3 class="title">治疗方案</h3>
          </template>
          <div class="diagResult">
            <div class="text">
              <div class="box">
                <p>
                  {{ API_diagInfo.API_treatment.API_description.join("，") }}
                </p>
              </div>
            </div>
            <div
              v-show="API_diagInfo.API_treatment.API_prescription.length > 0"
              class="prescription"
            >
              <span class="label">处方</span>
              <PrescriptionTable
                :prescription="API_diagInfo.API_treatment.API_prescription"
              ></PrescriptionTable>
            </div>
          </div>
        </el-collapse-item>
      </el-collapse>
    </div>
  </div>
</template>

<script>
import Prescription from "../components/Prescription.vue";
import { getPatientDiagRecord } from "@api/docchafang/docchafang.js";

export default {
  components: {
    PrescriptionTable: Prescription

  },
  props: {
    pid: {
      type: String,
      default: () => {
        return "456";
      },
    },
  },
  data() {
    return {
      pages: {
        collapse_activeNames: [, "2", "3", "4", "5", "6", "7"], //页面中激活的折叠面板

        prescription: [], //默认处方表格格式
      },
      API_state: "已完成",
      //患者相关信息
      patientInfo: {
        // 患者病情描述
        API_description: "",
        API_history: {
          guoming: "",
          jiazu: "",
          jiwang: "",
        },
      },
      API_diagInfo: {
        //诊断结论
        API_diagResult: ["", ""],
        //治疗方案
        API_treatment: {
          API_description: ["", ""],
          API_prescription: [], //处方
          API_prescriptionFlag: true,
        },
      },
    };
  },
  methods: {
  },
  directives: {
    drag: {
      // 指令的定义
      bind: function (el) {
        let odiv = el; //获取当前元素
        el.onmousedown = (e) => {
          //算出鼠标相对元素的位置
          let disX = e.clientX - odiv.offsetLeft;
          let disY = e.clientY - odiv.offsetTop;
          let left = "";
          let top = "";
          document.onmousemove = (e) => {
            //用鼠标的位置减去鼠标相对元素的位置，得到元素的位置
            left = e.clientX - disX;
            top = e.clientY - disY;
            //绑定元素位置到positionX和positionY上面
            //移动当前元素
            odiv.style.left = left + "px";
            odiv.style.top = top + "px";
          };
          document.onmouseup = (e) => {
            document.onmousemove = null;
            document.onmouseup = null;
          };
        };
      },
    },
  },

  mounted() {
    let pid = this.pid;
    this.$emit("startLoading");
    getPatientDiagRecord(pid).then((res) => {
      this.$emit("endLoading");
      console.log(res);
      this.patientInfo.API_description = res.API_symptom.join("、");
      this.patientInfo.API_history.guoming = res.API_history[0].guoming || "";
      this.patientInfo.API_history.jiazu = res.API_history[0].jiazu || "";
      this.patientInfo.API_history.jiwang = res.API_history[0].jiwang || "";
      this.API_diagInfo.API_diagResult = res.API_zhenduanjielun || "";
      this.API_diagInfo.API_treatment.API_description = res.API_zhiliao;
      this.API_diagInfo.API_treatment.API_prescription = res.API_chufang;
      // this.API_diagInfo = res.API_diagInfo;
      // this.API_state = res.API_state;
    });
  },
};
</script>

<style scoped lang="scss">
.mainContent {
  width: 100%;
  height: 100%;
  .title {
    font-size: 17px;
    color: #1c7e7c;
  }
  .basicInfo {
    padding-left: 20px;
    padding-top: 20px;
    .pic {
      width: 170px;
      height: 150px;
      border-right: 1px solid #d9d9d9;
      padding-right: 20px;
      img {
        width: 100%;
        height: 100%;
      }
      margin: auto;
      float: left;
    }
    .info {
      float: left;
      margin-left: 20px;
      width: 65%;
      height: 130px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      div {
        display: flex;
        justify-content: space-between;
        font-size: 18px;
      }
    }
  }
  .illState {
    width: 100%;
    display: flex;
    justify-content: space-between;
    .description {
      width: 100%;
      height: 100%;
      padding: 10px;
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      p {
        font-size: 18px;
        text-indent: 40px;
      }
    }
    .media {
      flex-shrink: 0;
      align-self: stretch;
      width: 450px;
      border-left: 2px solid #d9d9d9;
      padding-left: 20px;
      .switch {
        margin: 0 5px;
      }
      audio {
        margin-top: 10px;
      }
    }
  }
  .history {
    p {
      font-size: 18px;
      text-indent: 40px;
    }
  }
  .examResult {
    width: 95%;
    margin: auto;
    .imgExam {
      display: flex;
      justify-content: space-between;
      .info {
        width: 300px;
        padding-left: 30px;
        border-left: 2px solid #d9d9d9;
        .label {
          font-weight: bold;
          font-size: 20px;
        }
        .text {
          text-indent: 20px;
          font-size: 18px;
          word-wrap: break-word;
          word-break: break-all;
        }
      }
      .img {
        flex-shrink: 0;
      }
    }
    .tableExam {
      display: flex;
      justify-content: space-between;
      max-height: 350px;
      .table {
        flex-shrink: 0;
        width: 50%;
        min-width: 500px;
      }
      .pic {
        width: 45%;
        margin-top: 30px;
        margin-left: 30px;
        // max-height: 300px;
        overflow-y: auto;
      }
    }
  }
  .diagResult {
    width: 95%;
    margin: auto;
    .head {
      .checkBox {
        float: left;
      }
      .more {
        float: right;
        .link {
          margin-left: 30px;
        }
      }
    }
    .box {
      width: 100%;
      min-height: 100px;
      margin-top: 5px;
      border: 1px solid #e4e7ed;
      p {
        margin-top: 5px;
        font-size: 18px;
        text-indent: 20px;
      }
    }

    .prescription {
      margin-top: 20px;
      .label {
        font-size: 18px;
        color: #1c7e7c;
      }
    }
  }
  .after {
    width: 95%;
    margin: auto;
    .select {
      margin-top: 20px;
      margin-right: 50px;
    }
    .label {
      font-size: 18px;
    }
    .doc-nur {
      margin-top: 20px;
      display: flex;
      div {
        margin-right: 20px;
      }
    }
    .box {
      width: 100%;
      margin-top: 5px;
      margin-bottom: 20px;
      // border: 1px solid #e4e7ed;
    }
  }
  .btn {
    float: right;
    margin-top: 50px;
    margin-bottom: 50px;
    margin-right: 50px;
    width: 120px;
  }
}
.clearfix:after {
  content: ""; /*设置内容为空*/
  height: 0; /*高度为0*/
  line-height: 0; /*行高为0*/
  display: block; /*将文本转为块级元素*/
  visibility: hidden; /*将元素隐藏*/
  clear: both; /*清除浮动*/
}
.drag {
  width: 300px;
  position: absolute;
  top: 30%;
  left: calc(50% - 150px);
  span {
    z-index: 100;
    position: absolute;
    top: 0px;
    right: 10px;
    color: white;
    cursor: pointer;
  }
}
</style>

<style lang="scss">
</style>
