<template>
  <div class="after">
    <div class="result">
      <p>已选择：{{ result.length > 0 ? "" : "无" }}</p>
      <div
        style="margin-left: 20px; margin-bottom: 10px"
        v-for="item in result"
        :key="item.id"
      >
        <template v-if="item.nurses.length > 0">
          <span style="margin-left: 30px">护士：</span>
          <span style="margin-right: 10px" v-for="doc in item.nurses" :key="doc.id">{{
              doc.nurName
            }}</span>
        </template>

        <template v-if="item.doctors.length > 0">
          <span>医生：</span>
          <span class="org" style="margin-right: 20px">{{ item.orgName }}</span>
          <span style="margin-right: 10px" v-for="doc in item.doctors" :key="doc.id">{{
              doc.docName
            }}</span>
        </template>

      </div>
    </div>
    <div>
      <template>
        <div>本地住院护士：</div>
        <template>
          <div class="nur">
            <el-checkbox-group v-model="choosedNur">
              <el-row :gutter="5">
                <el-col
                  v-for="nur in nurList.slice(0, this.nurShowNum)"
                  :key="nur.UserID"
                  :xs="8"
                  :sm="6"
                  :md="4"
                  :lg="4"
                >
                  <div class="nurCard">
                    <div class="pic">
                      <img :src="nur.Image" @click="nurDetail(nur.UserID)" alt />
                    </div>
                    <div class="check">
                      <el-checkbox :label="nur">{{ nur.Name }}</el-checkbox>
                    </div>
                  </div>
                </el-col>

                <el-col
                  v-if="this.nurShowNum < nurList.length"
                  :xs="8"
                  :sm="6"
                  :md="4"
                  :lg="4"
                >
                  <div @click="moreNur" class="docCard">
                    <div class="more">
                      <i class="iconfont icon-more icon"></i>
                    </div>
                  </div>
                </el-col>
              </el-row>
            </el-checkbox-group>
          </div>
        </template>
        <div>远程医疗机构：</div>
        <div class="org">
          <el-row :gutter="5">
            <el-col v-for="item in info" :key="item.id" :xs="8" :sm="6" :md="4" :lg="4">
              <div @click="orgSelect(item)" class="orgCard">
                <div class="pic">
                  <img :src="item.orgPic" @click="orgDetail(item.orgId)" alt />
                </div>
                <div
                  :class="
                    currentOrg.orgId && currentOrg.orgId == item.orgId
                      ? ' check choosed'
                      : 'check'
                  "
                >
                  {{ item.orgName }}
                </div>
              </div>
            </el-col>
          </el-row>
        </div>
        <template v-if="currentOrg.orgId">
          <div>医生：</div>
          <div class="doc">
            <el-checkbox-group v-model="choosedDoc">
              <el-row :gutter="5">
                <el-col
                  v-for="doc in currentOrg.doctors.slice(0, this.docShowNum)"
                  :key="doc.id"
                  :xs="8"
                  :sm="6"
                  :md="4"
                  :lg="4"
                >
                  <div class="docCard">
                    <div class="pic">
                      <img :src="doc.docPic" @click="docDetail(doc.docId)" alt />
                    </div>
                    <div class="check">
                      <el-checkbox :label="doc">{{ doc.docName }}</el-checkbox>
                    </div>
                  </div>
                </el-col>
                <el-col
                  v-if="this.docShowNum < currentOrg.doctors.length"
                  :xs="8"
                  :sm="6"
                  :md="4"
                  :lg="4"
                >
                  <div @click="moreDoc" class="docCard">
                    <div class="more">
                      <i class="iconfont icon-more icon"></i>
                    </div>
                  </div>
                </el-col>
              </el-row>
            </el-checkbox-group>
          </div>
        </template>
      </template>
    </div>
  </div>
</template>

<script>
import { getOrgInfo } from "@api/orgmanage/orgmanage.js";
import { getUserDetails } from "@api/infodesc/infodesc.js";
export default {
  props: {
    medicalInfo: {
      type: Array,
      default: () => {
        return [
          {
            orgId: "1",
            orgPic:
              "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
            orgName: "张三的专家团队",
            doctors: [
              {
                docId: "101002",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "李四",
              },
              {
                docId: "101003",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "王五",
              },
              {
                docId: "101004",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "赵六",
              },
            ],
            nurses: [
              {
                nurId: "1",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王三",
              },
              {
                nurId: "2",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王二",
              },
              {
                nurId: "3",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王大",
              },
            ],
          },
          {
            orgId: "7",
            orgPic:
              "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
            orgName: "成都市第一人民医院",
            doctors: [
              {
                docId: "8",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "李四",
              },
              {
                docId: "9",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "王五",
              },
              {
                docId: "10",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "赵六",
              },
            ],
            nurses: [
              {
                nurId: "11",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王三",
              },
              {
                nurId: "12",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王二",
              },
              {
                nurId: "13",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王大",
              },
            ],
          },
          {
            orgId: "14",
            orgPic:
              "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
            orgName: "成都市第三人民医院",
            doctors: [
              {
                docId: "15",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "李四",
              },
              {
                docId: "16",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "王五",
              },
              {
                docId: "17",
                docPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                docName: "赵六",
              },
            ],
            nurses: [
              {
                nurId: "18",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王三",
              },
              {
                nurId: "19",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王二",
              },
              {
                nurId: "20",
                nurPic:
                  "https://fuss10.elemecdn.com/e/5d/4a731a90594a4af544c0c25941171jpeg.jpeg",
                nurName: "王大",
              },
            ],
          },
        ];
      },
    },
    nurList: {
      type: Array,
      default: () => {
        return [];
      }
    },
    state: {
      type: String,
      default: () => {
        return "";
      },
    },
    preInfo: {
      type: Array,
      default: () => {
        return [];
      },
    }
  },
  computed: {
    info: function () {
      let obj = this.medicalInfo;
      obj.forEach((item) => {
        item.doctors.forEach((doc) => {
          doc.orgId = item.orgId;
          doc.orgName = item.orgName;
        });
        item.nurses.forEach((nur) => {
          nur.orgId = item.orgId;
          nur.orgName = item.orgName;
        });
      });
      return obj;
    },
    result: function () {
      let orgs = [];
      this.choosedDoc.forEach((doc) => {
        let flag = true;
        orgs.forEach((org) => {
          if (org.orgId == doc.orgId) {
            org.doctors.push({
              docName: doc.docName,
              docId: doc.docId,
            });
            flag = false;
          }
        });
        if (flag) {
          orgs.push({
            orgName: doc.orgName,
            orgId: doc.orgId,
            doctors: [
              {
                docName: doc.docName,
                docId: doc.docId,
              },
            ],
            nurses: [],
          });
        }
      });

      this.choosedNur.forEach((nur) => {
        let flag = true;
        orgs.forEach((org) => {
          org.nurses.push({
            nurName: nur.Name,
            nurId: nur.UserID,
          });
          flag = false;
        });
        if (flag) {
          orgs.push({
            doctors: [],
            nurses: [
              {
                nurName: nur.Name,
                nurId: nur.UserID,
              },
            ],
          });
        }

        // this.$emit("after", orgs);
      });
      this.$emit("after", orgs);
      return orgs;
    },
  },
  data() {
    return {
      docShowNum: 5,
      nurShowNum: 5,
      choosed: [],
      currentOrg: {},
      choosedDoc: [],
      choosedNur: [],
      orgInfos: [],
      orgId: 0,
      orgInfo: [],
      orgDialogVisible: false,
      docDialogVisible: false,
      nurDialogVisible: false,
      docInfo: [],
      nurInfo: [],
    };
  },
  watch: {
    currentOrg() {
      this.docShowNum = 5;
      this.nurShowNum = 5;
    },
  },
  mounted() {
    this.$store.commit("startLoading");
    getOrgInfo().then((res) => {
      this.orgInfos = res;
      this.$store.commit("endLoading");
    });
  },
  methods: {
    orgHandleClose() {
      // this.orgDialogVisible=false;
    },
    docHandleClose() {
      // this.docDialogVisible = false;
    },
    nurHandleClose() {
      // this.nurDialogVisible = false;
    },
    orgDetail(id) {
      // this.orgDialogVisible = true;
      this.orgInfos.forEach((org) => {
        if (org.HospitalID === id) {
          this.orgId = org.HospitalID;
          this.orgInfo = org.HospitalIntroduction;
        }
      });
    },
    docDetail(id) {
      // this.docDialogVisible = true;
      getUserDetails(id).then((res) => {
        this.docInfo = res.info;
      });
    },
    nurDetail(id) {
      // this.nurDialogVisible = true;
      getUserDetails(id).then((res) => {
        this.nurInfo = res.info;
      });
    },
    moreDoc() {
      this.docShowNum += 5;
    },
    moreNur() {
      this.nurShowNum += 5;
    },
    docChanged(docs) {
      this.choosed.forEach((item) => {
        item.doctors = [];
        docs.forEach((doc) => {
          if (doc.orgId == item.orgId) {
            item.doctors.push(doc);
          }
        });
      });
    },
    nurChanged(nurs) {
      this.choosed.forEach((item) => {
        item.nurses = [];
        nurs.forEach((nur) => {
          if (nur.orgId == item.orgId) {
            item.nurses.push(nur);
          }
        });
      });
    },
    orgSelect(org) {
      if (this.currentOrg.orgId == org.orgId) {
        this.currentOrg = {};
      } else {
        // org.nurses = this.nurList;
        this.currentOrg = org;
      }
    },
  },
};
</script>

<style lang="scss" scoped>
.after {
  font-size: 16px;
  width: 100%;
  .org {
    .orgCard {
      cursor: pointer;
      width: 100%;
      height: 170px;
      display: flex;
      flex-direction: column;
      .pic {
        margin: auto;
        img {
          width: 100px;
          height: 120px;
        }
      }
      .check {
        margin: auto;
      }
    }
  }
  .doc {
    .docCard {
      width: 100%;
      height: 170px;
      display: flex;
      flex-direction: column;
      .pic {
        margin: auto;
        img {
          width: 100px;
          height: 120px;
        }
      }
      .check {
        margin: auto;
      }
      .more {
        text-align: center;
        .icon {
          font-size: 80px;
          color: #c0c4cc;
          line-height: 170px;
          cursor: pointer;
        }
      }
    }
  }
  .nur {
    .nurCard {
      width: 100%;
      height: 170px;
      display: flex;
      flex-direction: column;
      .pic {
        margin: auto;
        img {
          width: 100px;
          height: 120px;
        }
      }
      .check {
        margin: auto;
      }
      .more {
        text-align: center;
        .icon {
          font-size: 80px;
          color: #c0c4cc;
          line-height: 170px;
          cursor: pointer;
        }
      }
    }
  }
  .choosed {
    padding: 1px;
    border-radius: 3px;
    background-color: #409eff;
  }
  .result {
    font-size: 16px;
    .org {
      color: #1c7e7c;
    }
  }
}
</style>
