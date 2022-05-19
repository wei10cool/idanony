<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="10">
      <v-card class="logo py-4 justify-center rounded-xl" light>
        <v-card-title class="justify-center display-1 pb-0"
          ><span class="text--center">艾滴科技</span></v-card-title
        >
        <v-card-title class="justify-center display-2"
          ><span class="text--center">執行長信箱</span></v-card-title
        >

        <v-form ref="mainform">
          <v-card-text>
            <v-text-field
              outlined
              class="mx-2"
              v-model="exclusiveCode"
              placeholder="請輸入專屬碼"
              clearable
              counter="12"
              maxlength="12"
              @focus="codekeyControl"
              @keydown.enter.prevent="enterControl"
              @keydown="exclusiveContentshow=false"
              :rules="[rules.pwdcheck]"
              ><span style="width: 80px; font-size: 1.3em" slot="prepend"
                >專屬碼</span
              >
              <v-icon @click="getquestionData" slot="append" color="success"
                >mdi-reload</v-icon
              >
            </v-text-field>
            <div class="text-center headline" style="color: red" v-if="master">
              注意：您正在使用管理專屬碼
            </div>
          </v-card-text>
          <v-card-text class="pt-0"
            ><v-btn
              @click="
                () => {
                  if (this.$refs.mainform.validate() == false) {
                    this.$toast.error(`您輸入的專屬碼不符合規定`, {
                      duration: 2000,
                    })
                    return
                  }
                  exclusiveContentshow = !exclusiveContentshow
                  if (exclusiveCode.toLowerCase().includes('lotus')) {
                    setcursor()
                  }
                }
              "
              :disabled="!dayisBetween"
              style="color: white"
              :class="dayisBetween ? 'info' : 'error'"
              >{{ dayisBetween ? '我要提問' : '未開放' }}</v-btn
            >

            <v-btn
              v-if="master"
              @click="showmanagedig"
              class="primary"
              style="color: white"
              >管理設定</v-btn
            >
            <br />
            <span>可提問日期：{{ `${opened_date}~${closed_date}` }}</span>
          </v-card-text>
          <v-card-text v-show="exclusiveContentshow">
            <v-textarea
              name="input-7-1"
              outlined
              v-model="exclusiveContent"
              label="請輸入您對公司的寶貴建議~♥♥♥"
            ></v-textarea>
          </v-card-text>
          <v-card-actions v-show="exclusiveContentshow">
            <v-spacer></v-spacer>
            <v-btn
              color="primary"
              style="color: white"
              :disabled="!exclusiveCode || exclusiveCode.length <= 0"
              @click="questionsubmit"
              ><v-icon>mdi-incognito</v-icon>送出</v-btn
            >
          </v-card-actions>
        </v-form>
      </v-card>
      <v-dialog v-model="managedig" width="500">
        <v-card light>
          <v-card-title class="headline">管理設定</v-card-title>
          <v-form ref="manform">
            <!-- 管理專屬碼 -->
            <v-card-text>
              <span class="text-h6" style="color: purple">管理專屬碼</span>
              <v-text-field
                filled
                dense
                v-model="newMan"
                :rules="[rules.pwdcheck]"
                counter="12"
                maxlength="12"
                placeholder="新增管理專屬碼"
              >
                <v-btn
                  rounded
                  :disabled="!newMan"
                  @click="maneditsubmit"
                  color="primary mb-1"
                  style="color: white"
                  slot="append"
                  class="mr-1"
                  >修改</v-btn
                >
                <v-btn
                  rounded
                  :disabled="!newMan"
                  @click="mansubmit"
                  color="primary mb-1"
                  style="color: white"
                  slot="append"
                  >新增</v-btn
                >
              </v-text-field>
              <v-chip
                class="mx-1"
                :close="exclusiveCode != item.encrypted_code"
                outlined
                :color="`${
                  exclusiveCode == item.encrypted_code ? 'purple' : 'purple'
                }`"
                label
                v-for="item in masterDataOra"
                :key="item.id"
                @click:close="mandel(item)"
                >{{ `${item.encrypted_code}` }}</v-chip
              >
            </v-card-text>
            <v-divider></v-divider>
            <!-- 開放日期 -->
            <v-card-text>
              <span class="text-h6" style="color: purple">開放日期</span>
              <v-row>
                <!-- 選擇日期sdate -->
                <v-col cols="12" sm="5">
                  <div>
                    <v-menu
                      v-model="menu_sdate"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      transition="scale-transition"
                      offset-y
                      min-width="auto"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="sdate"
                          label="起日"
                          filled
                          dense
                          prepend-icon="mdi-calendar"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                          clearable
                          @click:prepend="() => (sdate = getNowDate())"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="sdate"
                        light
                        @input="menu_sdate = false"
                      ></v-date-picker>
                    </v-menu>
                  </div>
                </v-col>
                <!-- 選擇日期edate -->
                <v-col cols="12" sm="5">
                  <div>
                    <v-menu
                      v-model="menu_edate"
                      :close-on-content-click="false"
                      :nudge-right="40"
                      transition="scale-transition"
                      offset-y
                      min-width="auto"
                    >
                      <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                          v-model="edate"
                          label="訖日"
                          filled
                          dense
                          prepend-icon="mdi-calendar"
                          readonly
                          v-bind="attrs"
                          v-on="on"
                          clearable
                          @click:prepend="() => (edate = getNowDate())"
                        ></v-text-field>
                      </template>
                      <v-date-picker
                        v-model="edate"
                        light
                        @input="menu_edate = false"
                      ></v-date-picker>
                    </v-menu>
                  </div>
                </v-col>
                <!-- 修改 -->
                <v-col cols="12" sm="2">
                  <v-btn
                    class="primary"
                    style="color: white"
                    @click="sedatesubmit"
                    >確認</v-btn
                  >
                </v-col>
              </v-row>
            </v-card-text>
            <v-divider></v-divider>
            <!-- 清空提問 -->
            <v-card-text v-if="false">
              <span class="text-h6" style="color: purple">清空提問</span>
              <br />
              <v-btn class="error" style="color: white">清空所有提問</v-btn>
            </v-card-text>
          </v-form>
        </v-card>
      </v-dialog>
    </v-col>
    <v-col cols="12" sm="10">
      <v-data-table
        class="rounded-xl"
        :headers="headers"
        :items="
          switchreply ? desserts.filter((x) => x.is_reply == false) : desserts
        "
        :footer-props="{ 'items-per-page-options': [-1, 25, 50, 100] }"
        light
      >
        <template v-slot:top>
          <v-toolbar flat class="rounded-xl">
            <v-spacer></v-spacer>
            <v-switch
              v-model="switchreply"
              inset
              :label="`只顯示未回覆`"
            ></v-switch>
          </v-toolbar>
        </template>
        <template v-slot:[`item.action`]="{ item }">
          <v-btn
            style="color: white"
            :color="item.is_reply ? 'primary' : 'warning'"
            @click="showreplydig(item)"
            >{{ `${item.is_reply ? '查看' : '開始'}回覆` }}</v-btn
          >
        </template>
      </v-data-table>
      <v-dialog v-model="replydig" width="500">
        <v-card light>
          <v-card-title style="color: purple">提問內容</v-card-title>
          <v-card-text>
            <v-textarea
              outlined
              filled
              v-model="replyItem.content"
              :readonly="true"
            ></v-textarea>
          </v-card-text>
          <v-divider></v-divider>
          <v-card-title style="color: purple">回覆內容</v-card-title>
          <v-card-text>
            <v-form ref="replyform">
              <span>{{ replyItem.answer_time }}</span>
              <v-textarea
                outlined
                filled
                v-model="replyItem.answer"
                :readonly="!master"
                :clearable="master"
              ></v-textarea>
              <!-- <v-btn class="error" tile>刪除??(應該是不需要這功能)</v-btn> -->
              <v-btn
                block
                class="primary"
                tile
                style="color: white"
                v-if="master"
                :disabled="String(replyItem.answer).trim().length == 0"
                @click="replysubmit"
                >送出回覆(可重覆修改)</v-btn
              >
              <v-btn
                block
                class="error"
                v-if="master"
                style="color: white"
                @click="replydelete"
                >刪除</v-btn
              >
            </v-form>
          </v-card-text>
          <!-- <v-card-text v-else>
            <span>{{replyItem.answer_time}}</span>
            <v-textarea :disabled="true" outlined style="white-space:pre-line;" v-model="replyItem.answer"></v-textarea>
          </v-card-text> -->
        </v-card>
      </v-dialog>
    </v-col>
  </v-row>
</template>

<script>
import dayjs from 'dayjs'
import isBetween from 'dayjs/plugin/isBetween'
dayjs.extend(isBetween)
export default {
  name: 'IndexPage',
  data() {
    return {
      cryptopwd: 'idwater@admin1235',
      // materCode: 'IDWater12345', //管理碼
      masterDataOra: [], //管理碼原始資料
      masterData: [], //管理碼
      // master: false,
      exclusiveCode: '', //正在輸入的專屬碼IDWater12345
      exclusiveContent: '',
      exclusiveContentshow: false,
      headers: [
        {
          text: '提問時間',
          align: 'center',
          sortable: false,
          value: 'created_time',
          width:200,
        },
        { text: '提問內容', align: 'start', sortable: false, value: 'content' },
        { text: '回覆狀態', align: 'start', sortable: false, value: 'action',width:100, },
      ],
      desserts: [
        // {
        //   id: 1,
        //   time: '2022-01-01 12:00',
        //   content:
        //     'Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquam, vitae?',
        //   reply: true,
        //   answer:
        //     '11111Lorem ipsum dolor sit, amet consectetur adipisicing elit. Impedit possimus optio nam quidem beatae, veritatis vel consequuntur mollitia corporis doloribus!',
        // },
        // {
        //   id: 2,
        //   time: '2022-01-01 12:00',
        //   content:
        //     'Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquam, vitae?',
        //   reply: false,
        //   answer:
        //     '22222Lorem ipsum dolor sit, amet consectetur adipisicing elit. Impedit possimus optio nam quidem beatae, veritatis vel consequuntur mollitia corporis doloribus!',
        // },
      ],
      replydig: false,
      replyItem: {},
      switchreply: false,
      rules: {
        //pwdcheck:value=>(value && /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{6,}$/.test(value)) || '6位數，(需包含大寫字母、小寫字母、數字)',
        pwdcheck: (value) =>
          (value &&
            /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[A-Za-z\d]{6,}$/.test(value)) ||
          '6位數，(僅包含大寫字母、小寫字母、數字，禁止使用符號)',
      },
      //--管理區域
      managedig: false,
      newMan: '',
      //------日曆
      menu_sdate: false,
      sdate: '',
      menu_edate: false,
      edate: '',
      //
      opened_date: '',
      closed_date: '',
    }
  },
  methods: {
    enterControl: async function () {
      await this.getquestionData();
    },
    codekeyControl: async function () {
      this.desserts = []
    },
    //取得問題清單
    getquestionData: async function () {
      if (this.$refs.mainform.validate() == false) {
        this.$toast.error(`您輸入的專屬碼不符合規定`, {
          duration: 2000,
        })
        return
      }
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/question/`
      var parm = {
        encrypted_code: this.exclusiveCode, //this.crypto(this.exclusiveCode),
        is_master: this.master,
      }
      console.log(parm)
      await this.$axios
        .get(url, { params: parm })
        .then((res) => {
          if (res.status == 200) {
            this.desserts = res.data
            this.$toast.success(`取得成功:${res.data.length}筆`, {
              duration: 2000,
            })
          } else {
            this.$toast.error(`取得失敗:${res.data}`, {
              duration: 2000,
            })
          }
        })
        .catch((error) => {
          this.$toast.error(`取得失敗:${error}`, {
            duration: 2000,
          })
        })
        .finally(() => {})
    },
    //提問送出
    questionsubmit: async function () {
      var parm = {
        encrypted_code: this.exclusiveCode, //專屬碼
        content: this.exclusiveContent, //內容
      }
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/question/`
      await this.$axios
        .post(url, parm)
        .then((res) => {
          if (res.status == 201) {
            this.exclusiveContent = ''
            this.getquestionData() //取得清單
            this.$toast.success(`新增成功`, {
              duration: 2000,
            })
          } else {
            this.$toast.error(`取得失敗:${res.data}`, {
              duration: 2000,
            })
          }
        })
        .catch((error) => {
          this.$toast.error(`取得失敗:${error}`, {
            duration: 2000,
          })
        })
        .finally(() => {
          //this.getdata();
        })
    },
    // crypto: function (message) {
    //   return this.$CryptoJS.AES.encrypt(message, this.cryptopwd).toString()
    // },
    // decrypto: function (message) {
    //   try {
    //     var decode = this.$CryptoJS.AES.decrypt(
    //       message,
    //       this.cryptopwd
    //     ).toString(this.CryptoJS.enc.Utf8)
    //     if (decode == '') {
    //       return '無法解譯'
    //     } else {
    //       return decode
    //     }
    //     // return this.$CryptoJS.AES.decrypt(message, this.cryptopwd).toString(
    //     //   this.CryptoJS.enc.Utf8
    //     // )
    //   } catch (error) {
    //     return '無法解譯'
    //   }
    // },
    showreplydig: function (item) {
      this.replyItem = _.cloneDeep(item)
      this.replyItem.content = this.replyItem.content.replace(
        /(\\r)*\\n/g,
        '<br>'
      )
      this.replydig = true
    },
    //刪除回覆
    replydelete: async function () {
      if (confirm(`確認刪除此提問？`)) {
        let id = this.replyItem.id
        let url = `https://anonymous-api-site.herokuapp.com/api/qa/question/${id}/`
        var parm = {
          is_master: this.master,
        }

        await this.$axios
          .delete(url, { data: parm })
          .then((res) => {
            if (res.status == 204) {
              this.replyItem = {}
              this.replydig = false
              //不要重抓資料，畫面更新內容即可
              var idx = this.desserts.indexOf(
                this.desserts.filter((x) => x.id == id)[0]
              )
              this.desserts.splice(idx, 1)
              this.$toast.success(`刪除成功`, {
                duration: 2000,
              })
            } else {
              this.$toast.error(`刪除失敗:${res.data}`, {
                duration: 2000,
              })
            }
          })
          .catch((error) => {
            this.$toast.error(`刪除失敗:${error}`, {
              duration: 2000,
            })
          })
          .finally(() => {
            //this.getdata();
          })
      }
    },
    //送出回覆
    replysubmit: async function () {
      let id = this.replyItem.id
      var parm = {
        answer: this.replyItem.answer,
        administrator_code: this.exclusiveCode,
      }
      console.log(parm)
      this.replyItem = {} //清空
      this.replydig = false
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/question/${id}/`
      await this.$axios
        .patch(url, parm)
        .then((res) => {
          if (res.status == 200) {
            var getitem = this.desserts.filter((x) => x.id == id)[0]
            getitem.answer = res.data.answer
            getitem.is_reply = res.data.is_reply
            getitem.answer_time = res.data.answer_time
            this.$toast.success(`修改成功`, {
              duration: 2000,
            })
          } else {
            this.$toast.error(`修改失敗:${res.data}`, {
              duration: 2000,
            })
          }
        })
        .catch((error) => {
          this.$toast.error(`修改失敗:${error}`, {
            duration: 2000,
          })
        })
        .finally(() => {
          //this.getdata();
        })
    },
    //設定開放時段
    sedatesubmit: async function () {
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/settings/1/`
      var parm = {
        opened_date: this.sdate,
        closed_date: this.edate,
      }
      await this.$axios
        .patch(url, parm)
        .then((res) => {
          if (res.status == 200) {
            this.$toast.success(`設定時段成功`, {
              duration: 2000,
            })
          }
          this.getsedate()
        })
        .catch((error) => {
          this.$toast.error(`設定時段失敗:${error}`, {
            duration: 2000,
          })
        })
    },
    //取得開放日期
    getsedate: async function () {
      // this.sdate = '2022-01-01'
      // this.edate = '2022-12-31'
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/settings/`
      await this.$axios
        .get(url)
        .then((res) => {
          if (res.status == 200) {
            this.sdate = _.cloneDeep(res.data.opened_date)
            this.edate = _.cloneDeep(res.data.closed_date)
            //--
            this.opened_date = _.cloneDeep(res.data.opened_date)
            this.closed_date = _.cloneDeep(res.data.closed_date)
          }
        })
        .catch((error) => {
          this.$toast.error(`取得失敗:${error}`, {
            duration: 2000,
          })
        })
        .finally(() => {
          //this.getdata();
        })
    },
    //取得管理碼清單
    getmasterdata: async function () {
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/administrator/`
      await this.$axios
        .get(url)
        .then((res) => {
          this.masterDataOra = res.data
          this.masterDataOra.map((x) => (x.decrypted_code = x.encrypted_code))
          this.masterData = res.data.map((x) => x.encrypted_code)
        })
        .catch((error) => {
          this.$toast.error(`取得失敗:${error}`, {
            duration: 2000,
          })
        })
        .finally(() => {
          //this.getdata();
        })
    },
    //管理顯示
    showmanagedig: function () {
      this.newMan = ''
      this.managedig = true
    },
    //修改管理
    maneditsubmit: async function () {
      //密碼是否符合規則
      if (this.$refs.manform.validate() == false) {
        return
      }
      //密碼是否與原本相同
      if (this.exclusiveCode == this.newMan) {
        this.$toast.error(`新密碼與舊密碼相同`, {
          duration: 2000,
        })
        return
      }
      //確認是否要修改
      if (
        confirm(
          `您要修改管理專屬碼：「${this.exclusiveCode}」為「${this.newMan}」？`
        ) == false
      ) {
        this.$toast.info(`取消修改：「${this.exclusiveCode}」`, {
          duration: 2000,
        })
        return
      }
      var parm = {
        encrypted_code: this.newMan, //this.crypto(this.newMan),
      }
      let id = this.masterDataOra.filter(
        (x) => x.decrypted_code == this.exclusiveCode
      )[0].id

      let url = `https://anonymous-api-site.herokuapp.com/api/qa/administrator/${id}/`
      await this.$axios
        .patch(url, parm)
        .then((res) => {
          debugger
          if (res.status == 200) {
            this.getmasterdata() //re get master list
            this.exclusiveCode = this.newMan
            this.newMan = ''

            this.$toast.success(`修改成功`, { duration: 2000 })
          } else {
            this.$toast.error(`修改失敗：${res.data}`, { duration: 2000 })
          }

          console.log('修改API:' + res.request.responseURL)
        })
        .catch((error) => {
          this.$toast.error(`取得失敗:${error}`, {
            duration: 2000,
          })
        })
        .finally(() => {})
    },
    //新增管理
    mansubmit: async function () {
      if (this.$refs.manform.validate() == false) {
        return
      }
      if (this.newMan == this.exclusiveCode) {
        this.$toast.error(`新增失敗：與自己的管理專屬碼相同`, {
          duration: 2000,
        })
        return
      }
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/administrator/`
      var parm = {
        encrypted_code: this.newMan, //this.crypto(this.newMan),
      }
      await this.$axios
        .post(url, parm)
        .then((res) => {
          if (res.data.hasOwnProperty('encrypted_code')) {
            // var yourmastercode = this.decrypto(res.data.encrypted_code);
            this.newMan = ''
            this.getmasterdata() //re get master list
            this.$toast.success(`新增成功`, { duration: 2000 })
          } else {
            this.$toast.error(`新增失敗：${res.data}`, { duration: 2000 })
          }
          console.log('取得API:' + res.request.responseURL)
        })
        .catch((error) => {
          this.$toast.error(`取得失敗:${error}`, {
            duration: 2000,
          })
        })
        .finally(() => {
          //this.getdata();
        })
    },
    //刪除管理
    mandel: async function (data) {
      //      if (confirm(`是否刪除${this.decrypto(data.encrypted_code)}?`) == false) {
      if (confirm(`是否刪除${data.encrypted_code}?`) == false) {
        return
      }
      // var checkmaster = this.decrypto(data.encrypted_code) == this.exclusiveCode
      var checkmaster = data.encrypted_code == this.exclusiveCode
      if (checkmaster == true) {
        this.$toast.error(`刪除失敗：無法刪除自己管理碼`, { duration: 2000 })
        return
      }
      let id = data.id
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/administrator/${id}/`
      await this.$axios
        .delete(url)
        .then((res) => {
          if (res.status == 204) {
            this.getmasterdata() //re get master list
            this.$toast.success(`刪除成功`, { duration: 2000 })
          } else {
            this.$toast.error(`刪除失敗：${res.data}`, { duration: 2000 })
          }

          console.log('取得API:' + res.request.responseURL)
        })
        .catch((error) => {
          this.$toast.error(`取得失敗:${error}`, {
            duration: 2000,
          })
        })
        .finally(() => {})
    },
    // isMaster: function(){
    //   var data = this.masterData.map(x=>this.decrypto(x));
    //   data.push('IDWater12345');
    //   return data.includes(this.exclusiveCode);
    // }
    getNowDate: function () {
      let mydate = dayjs().format('YYYY-MM-DD')
      return mydate
    },
    setcursor: function () {
      // 16
      // document.documentElement.style.cursor = 'url("data:image/x-icon;base64,AAABAAEAEBAAAAEAIABoBAAAFgAAACgAAAAQAAAAIAAAAAEAIAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADli/8A5Yz/BeWL/ybmjf8x55T/EeOC/wD/gv8A/5T/Ef+M/zD/iP8k/4n/BP+J/wAAAAAAAAAAAIwA/gDnlP8E5Ij/LuSH/3nlif9Z5o3/RumZ/x3he/8A/3P/AP6Y/x7/i/9G/4b/Wv+E/3f/hf8s/5L/A/9p/wDmjv8m5Ib/mOOF/9rkiP9k5If/WuWK/0vqof8E20r/AP5r/wD+mv8E/4b/S/+F/1j/hf9l/4L/2P+D/5b/iv8m5o7/CeSH/07khf+s5If/o+SG/5vli/8L55T/FOea/wn/lf8J/5H/E/+I/wz/g/+b/4T/ov+C/6//g/9R/4r/Cemd/wDjhP8A5In/K+SF/8bkh/9+5Ij/SeWK/2vkjf8Q/4n/EP+H/2n/hf9G/4T/ff+C/8X/h/8r/4L/AP6a/wDmj/8A55P/DuSH/4rjhP/S5IX/yOSH/6fkiP+T44n/I/+G/yP/hf+U/4T/qP+C/8j/gf/V/4T/iv+P/w7/jP8A55H/AOeU/wPli/8S5Yn/EeSI/23kh/+f5Ij/nOSH/3P/hP9y/4X/nP+E/53/hf9t/4n/FP+I/xP/kP8E/47/AAAAAAAAAAAA8r3/AOB5/wDkh/905IX/o+SJ/0/nhf/L+4L/y/+G/1L/gv+m/4T/df92/wD/vf8AAAAAAAAAAAAAAAAAAAAAAPO4/wD///8A5o3/EOaN/wf/Rv8A6ob/dfiE/3eMQv8A/4r/CP+L/xH4//8A/7j/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA8Yv/AO+M/wr1i/8K84r/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//8AAP//AAD//wAA//8AAOGHAACBgQAAAYAAAAAAAADAAwAAgAEAAIABAADwDwAA8k8AAP5/AAD//wAA//8AAA=="), auto';
      // 32
      // document.documentElement.style.cursor = 'url("data:image/x-icon;base64,AAABAAEAICAAAAEAIACoEAAAFgAAACgAAAAgAAAAQAAAAAEAIAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA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AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA////////////////////////////////////////////7////wPA//wDwD/wA8APAIfhAAAf+ICAGZgB4DGMB/gxjB/4AYAf4AGAB+ABgAfgAAAH/wAA//8AAP//AAD//xw4///8P////n////////////////////////////8="), auto';
      // 64
      document.documentElement.style.cursor =
        'url("data:image/x-icon;base64,AAABAAEAQEAAAAEAIAAoQgAAFgAAACgAAABAAAAAgAAAAAEAIAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA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////gP///4D///+D/+X/iP94/pb/E/5//wD+tf8AAAAAAAAAAADtr/8A+eb/AeiX/y7li/+g5IX/8uOD///jg///44P//+OD///jg///44T//OSH/9fmj/9x8LH/DOWN/43jhP/944P//+WL/6rus/8I66f/AAAAAAAAAAAAAAAAAAAAAAD///8A////AP/+/wD/+v8A//j/AAAAAAAAAAAA////AP///wD///8A////AAAAAAAAAAAAAAAAAAAAAAAAAAAA/6D/AP6p/wr/h/+v/4D///+B//z/iv+D/br/Cf6O/2j/hf/N/4H/+f+A////gP///4D///+A////gP///4L/9v+H/63/kf84/rr/Av6m/wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADqnf8A7Kb/COeT/03liv+544X/9+OD///jg///44P//+OD///jhP/65o//bueT/1zkhf/z44P//+SF//Xnk/9P44P/AP7f/wAAAAAAAAAAAAAAAAAAAAAA66//AOyy/xLvvf8Y6qv/AP/6/wAAAAAAAAAAAP///wD+nf8A/rn/FP6r/w3+pf8AAAAAAAAAAAAAAAAAAAAAAP7T/wD/ff8A/4//Vf+C//j/gP///4L/7/6P/1L/jf93/4H//P+A////gP///4D///+A////gf/5/4X/xP6O/1r+of8M/oj/APz+/wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAPXG/wDkh/8A7KX/EOeT/1jlif+25IX/8uOD///jg///5Yv/quia/zjkh//W44P//+OD///kiP/E6p7/Eumb/wAAAAAAAAAAAAAAAAAAAAAA6qD/AOqd/wrol/9t6qT/Iuqj/wD///8AAAAAAAAAAAD///8A/p3/AP6h/yT+lf9m/6D/CP+j/wAAAAAAAAAAAAAAAAAAAAAA/5v/AP+e/xb/hf/K/4D///+A////hP/P/pf/Nv+F/7D/gP///4D///+C//b/hv/C/4n/Yf+Y/xT/AP8A/tr/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAPC6/wDnjv8A66H/DOeS/0Tli/+X5Yr/v+iY/z3li/+i44P//+OD///jhP/85o7/at1t/wDurP8AAAAAAAAAAAAAAAAA7bD/AOJ7/wDmkP9a5o//mfG4/wXtq/8AAAAAAAAAAAAAAAAAAAAAAP+o/wD/rv8G/4z/mv+Q/1T/ev8A/rz/AAAAAAAAAAAAAAAAAP+1/wD/UP8A/4z/c/+B//3/gP///4D///6I/5n+lf9B/4f/zP+J/6f/jv9R/53/Ev8A/wD91P8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADuqv8A77L/BO6w/w/mj/9j44T/9uOD///jg///5Yn/w+qf/xfpm/8AAAAAAAAAAAAAAAAAAAAAAOui/wDrpf8P5Yr/v+aO/3jecP8A+tr/AAAAAAAAAAAAAAAAAAAAAAD/zP8A/23/AP+K/3r/h/+5/qb/Df6h/wAAAAAAAAAAAAAAAAAAAAAA/pb/AP6c/xv/hf/J/4D///+A////gv/0/5D/Xv6n/xb+rP8H/pv/APn//wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA8cn/AOWJ/wDnlv8x5If/2uOD///jg///5Ib/6eeS/0fhfP8A8bj/AAAAAAAAAAAA///6AP35+gDdZv8A55X/P+SH/+/nkf9f5IX/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAP+B/wD/jf9g/4T/7f+S/zv/gv8A/M7/AAAAAAAAAAAAAAAAAP2p/wD/ev8A/o7/T/+D/+3/gP///4D///+F/9T/lf8q/4n/AP///wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA//z+AOaU/wDonP8e5In/u+OD///jg///5Ib/7eaS/2b+4v8BAAD/APXR/wPvrv8L6p//GuiV/zbmj/9o6Jf/WOaQ/3Hkhf/+55H/XeOF/wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAD/gv8A/4//Xf+C//3/jf9u/5f/UP+N/1n/lv8s/5//Ev+8/wb8+/8B/qf/APu9/wL/jf9w/4L/8f+A////gP///4b/sf6Z/xj+jf8A////AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA+Ov/AON9/wDokf8Y5Ij/quOE///jg///44P//+WK/9Pnjv+J5Yn/h+WN/5bliv+l5Yn/vuSI/9fkhv/v44T//+aN/4nljf+M44T//+aN/27he/8A//T/AAAAAAAAAAAAAAAAAAAAAAD///8A/3n/AP+L/2z/gf///4r/i/+K/4v/gv/+/4P/5/+F/8z/h/+w/4n/mf+L/4f/iP93/4X/dv+F/83/gP///4D///+B//7/iv+l/6H/Fv+V/wD4//8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA8Ln9AOOD/wDomP8k5Yn/seOD//7jg///44P//+OD///jg///44P//+OD///jg///44P//+OD///jg///44P//+OD///mkP+A5Ib/kOOD///li/+SfwD/AO6x/wAAAAAAAAAAAAAAAAAAAAAA/67/AP8A/wD/if+Q/4D///+D/5D/jf+A/4D///+A////gP///4D///+A////gP///4D///+A////gP/+/4D///+A////gP///4H//f+I/6f+lf8c/4D/APy3/wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA7qv/APjN/wLnk/8/5Ij/x+OD///jg///44P//+OD///jg///44P//+OD///jg///44P//+OD///jg///44P//+OD///khf/455D/buWL/4/jg///5In/wuyo/w/spf8AAAAAAAAAAAAAAAAAAAAAAP+i/wD/pv8N/4b/v/+A////h/+P/4z/cv+C//z/gP///4D///+A////gP///4D///+A////gP///4D///+A////gP///4D///+A////gP/+/4b/vf+S/zX///8B/7P/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA7rH/APC9/wjolv9p5Ij/4+OE///jg///44P//+OD///jg///44P//+OD///jg///44P//+OE///jhP/65Ib/6eSH/8jli/+V5pH/Tuyl/w3mj/9844P//+SG/+3omP8555L/AAAAAAAAAAAAAAAAAAAAAAD/j/8A/5T/NP+D/+v/gP///4v/ff+b/xL/jv9d/4j/pP+E/9T/gv/x/4H//f+A////gP///4D///+A////gP///4D///+A////gP///4D///+B////hf/e/5L/YP6z/wb+qP8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAO60/wDvuf8I6Jn/O+aQ/13mjv915o3/i+WL/5vli/+m5Ib/puSF/6blif+m5Yz/n+aM/43mj/906JX/YemY/1Xpm/9L77D/DO6u/wDkhv8A55L/WuOE//zjg///5o3/g5oA/wDyuv8AAAAAAAAAAAD+vv8A/xr/AP+L/37/gf///4H//P+N/1v/gv8A/6D/AP+r/wz/mf9J/5T/WP+Q/2r/i/+D/4n/nP+H/6v/if+5/4X/uf+G/7n/if+3/4b/qP+I/5r/iv+E/4z/bP+V/0T+rP8I/qn/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAP7+/wD+//8B/t3/AvGj/wLtnv8C/M3/A//7/wHnlP8A6Zr/KuWL/7bkhv/f5Yr/wOyk/xHspf8A6JP/AOiX/zHkhv/q44P//+SH/9TomP8e55T/AAAAAAAAAAAA/5n/AP+d/x3/hv/R/4D///+D/+z/lP80/5D/AP+j/wD/ov8S/4j/vf+E/9T/iP+o/5f/Jf8z/wD/yf8E/8T/CP+p/wj/r/8J/8j/CP/I/wP9+/8B/fb/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAD30v8A4Hb/AOaQ/2zjhP//44P//+WK/7XyvP8H9MD/AeeU/yPqnv8u5In/weOD///jhP/95Y3/dqcA/wDvs/8A/7X/AP83/wD/jf9x/4H//f+A////hf/F/qH/LP6c/x34//8A/rf/CP+H/7f/gP///4H//v+M/2j/ef8A/9D/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA77P/AP/x/wLli/+g44P//+OD///lif+w6Jf/QeaN/43kh//d5o7/h+aO/4HjhP//44P//+SH/9vomP8r55P/AP+P/wD/lf8l/4T/1f+A////gP///4r/hv+M/4H/hf/U/4r/ff+W/zX/hv+t/4D///+A////iP+d////Av+z/wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAOuj/wDrpP8P5Ij/xuOD///jg///5IX/8uSG//Djg///44P//+SI/8romP9P5Ib/6+OD///jg///5Yv/nOur/gj/tP8G/4n/lP+A////gP///4L/7v+W/1T/hf/H/4D///+B//7/g//q/4P/7v+A////gP///4b/xf+j/w7/ov8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADpmv8A6pz/IeSH/9/jg///44P//+OD///jg///44P//+OF//bljf+V7Kj/GeWK/7Djg///44P//+SF//Tmkv9h/47/WP+C//H/gP///4D///+H/7b+pf8e/4n/oP+B//n/gP///4D///+A////gP///4D///+E/93/mP8f/5f/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA55T/AOiW/zPkhv/t44P//+OD///jg///44T//eSI/8/mkf9e66X/CuOF/wDnkf9W44X/9+OD///jg///5ob/5/yE/+T/gP///4D///+C//n/jv9e/37/AP6g/w7/jP9r/4T/2v+B//7/gP///4D///+A////g//t/pT/Mv6S/wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAOaN/wDnlP9B5IX/9OOD///jhP/75Ij/1OaQ/3vpm/8eAAD/APG1/wDqnf8A66H/EeWJ/73jg///44P//+aD///7gP///4D///+A////hv/E/p//Ff6c/wD+tf8A/Oj/Af6X/yf/iv+G/4T/3f+B//3/gP///4L/9P+R/0D/i/8AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADljv8A6Jf/S+SI/+Tlif+15o7/ZOmb/x355P8B8cD/AAAAAAAAAAAA8LT/AOF9/wDnkv9Y44X/9uOD///mg///+4D///+A////gv/4/43/Xv9w/wD9sv8AAAAAAAAAAAD/q/8A/rn/Av+W/yb/i/9z/4X/wv+F/+v/lf9M/4v/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA6JX/AOuZ/xfol/8w7KX/CthW/wD42v8AAAAAAAAAAAAAAAAAAAAAAAAAAADqoP8A66b/DeWL/6/jg///5oP///uA////gP///4b/tv+g/xD/m/8AAAAAAAAAAAAAAAAAAAAAAAAAAAD/x/8A////AP+f/xD/lf89/pr/HP6W/wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA9dT9AOSH/wDnlv8/44b/6eaD///7gP///4P/7f+R/0b/gf8A/r//AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADrpv8A9c//A+WN/4XmhP/++4H///+I/4z/v/8E/6b/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAOiX/wDnnf8a6Yj/w/qG/8r/lP8e/5D/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAADvsf8A84D/AO6T/zn4k/8+6nn/AP+q/wAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////gP/wH/////wAf+AD////8AB/4AD////AP///wD///wDwf+DwH//8AcB/4DgD//ADg//8HAD+AAcH//4OAAYAHg///weABwAEH///gwAPgAAf//+AAB/gAD+fn8AAf/gAPx+PwAH//gB/H4/gB///gH4/x+Af///A/j/H8D///4CAP8AQH///AAA/wAAP//4AAD/AAAf/+AAAH4AAAf/wAAAfgAAA//AABh+GAAD//4EGDwYID////wAPAg/////+AAYAB/////4AAAAH/////gAAAAf////+AIAQB/////4DgBgH/////gfAPgf////+P8A/x///////4H/////////gf/////////D/////////+f/////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////8="), auto'
    },
  },
  computed: {
    master: function () {
      // var data = this.masterData.map((x) => this.decrypto(x))
      var data = this.masterData
      data.push('IDWater12345')
      return data.includes(this.exclusiveCode)
    },
    dayisBetween: function () {
      //減一加一是因為同日期會造成判定非區間內
      var sd = dayjs(this.opened_date).add(-1, 'day')
      var ed = dayjs(this.closed_date).add(1, 'day')
      return dayjs().isBetween(sd, ed)
    },
  },
  async mounted() {
    await this.getmasterdata()
    await this.getsedate()
  },
}
</script>
