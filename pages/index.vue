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
              :rules="[rules.pwdcheck]"
              ><span style="width: 80px; font-size: 1.3em" slot="prepend"
                >專屬碼</span
              >
              <v-icon slot="append" color="success">mdi-reload</v-icon>
            </v-text-field>
            <div class="text-center headline" style="color: red" v-if="master">
              注意：您正在使用管理專屬碼
            </div>
          </v-card-text>
          <v-card-text class="pt-0"
            ><v-btn
              @click="exclusiveContentshow = !exclusiveContentshow"
              class="info"
              style="color: white"
              >顯示問題反應區</v-btn
            >
            <v-btn
              v-if="master"
              @click="showmanagedig"
              class="primary"
              style="color: white"
              >管理設定</v-btn
            >
          </v-card-text>
          <v-card-text v-show="exclusiveContentshow">
            <v-textarea
              name="input-7-1"
              outlined
              v-model="exclusiveContent"
              label="內容"
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
                  slot="append" class="mr-1"
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
                :close="exclusiveCode!=decrypto(item.encrypted_code)"
                outlined
                :color="`${exclusiveCode==decrypto(item.encrypted_code)?'purple':'purple'}`"
                label
                v-for="item in masterDataOra"
                :key="item.id"
                @click:close="mandel(item)"
                >{{ `${decrypto(item.encrypted_code)}` }}</v-chip
              >
            </v-card-text>
            <v-divider></v-divider>
            <!-- 開放時段 -->
            <v-card-text>
              <span class="text-h6" style="color: purple">開放時段</span>
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
                    @click="datesubmit"
                    >確認</v-btn
                  >
                </v-col>
              </v-row>
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
          switchreply ? desserts.filter((x) => x.reply == false) : desserts
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
            :color="item.reply ? 'primary' : 'warning'"
            @click="showreplydig(item)"
            >{{ `${item.reply ? '查看' : '開始'}回覆` }}</v-btn
          >
        </template>
      </v-data-table>
      <v-dialog v-model="replydig" width="500">
        <v-card light>
          <v-card-title>提問內容</v-card-title>
          <v-card-text>{{ replyItem.content }}</v-card-text>
          <v-divider></v-divider>
          <v-card-title>回覆內容</v-card-title>

          <v-card-text v-if="master">
            <v-form ref="replyform">
              <v-textarea
                filled
                v-model="replyItem.replycontent"
                clearable
              ></v-textarea>
              <!-- <v-btn class="error" tile>刪除??(應該是不需要這功能)</v-btn> -->
              <v-btn block
                class="primary"
                tile
                style="color:white;"
                :disabled="String(replyItem.replycontent).trim().length == 0"
                @click="replysubmit"
                >確定送出</v-btn
              >
            </v-form>
          </v-card-text>
          <v-card-text v-else>
            {{ replyItem.reply == true }}
            {{ replyItem.replycontent }}
          </v-card-text>
        </v-card>
      </v-dialog>
    </v-col>
  </v-row>
</template>

<script>
import dayjs from 'dayjs'
export default {
  name: 'IndexPage',
  data() {
    return {
      cryptopwd: 'idwater@admin1235',
      // materCode: 'IDWater12345', //管理碼
      masterDataOra: [], //管理碼原始資料
      masterData: [], //管理碼
      // master: false,
      exclusiveCode: 'IDWater12345', //正在輸入的專屬碼
      exclusiveContent: '',
      exclusiveContentshow: false,
      headers: [
        { text: '提問時間', align: 'center', sortable: false, value: 'time' },
        { text: '提問內容', align: 'start', sortable: false, value: 'content' },
        { text: '回覆狀態', align: 'start', sortable: false, value: 'action' },
      ],
      desserts: [
        {
          id: 1,
          time: '2022-01-01 12:00',
          content:
            'Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquam, vitae?',
          reply: true,
          replycontent:
            '11111Lorem ipsum dolor sit, amet consectetur adipisicing elit. Impedit possimus optio nam quidem beatae, veritatis vel consequuntur mollitia corporis doloribus!',
        },
        {
          id: 2,
          time: '2022-01-01 12:00',
          content:
            'Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquam, vitae?',
          reply: false,
          replycontent:
            '22222Lorem ipsum dolor sit, amet consectetur adipisicing elit. Impedit possimus optio nam quidem beatae, veritatis vel consequuntur mollitia corporis doloribus!',
        },
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
    }
  },
  methods: {
    questionsubmit: async function () {
      var parm = {
        exclusiveCode: this.crypto(this.exclusiveCode), //專屬碼
        content: this.exclusiveContent, //內容
      }
      console.log(parm)
    },
    crypto: function (message) {
      return this.$CryptoJS.AES.encrypt(message, this.cryptopwd).toString()
    },
    decrypto: function (message) {
      try {
        var decode = this.$CryptoJS.AES.decrypt(
          message,
          this.cryptopwd
        ).toString(this.CryptoJS.enc.Utf8)
        if (decode == '') {
          return '無法解譯'
        } else {
          return decode
        }
        // return this.$CryptoJS.AES.decrypt(message, this.cryptopwd).toString(
        //   this.CryptoJS.enc.Utf8
        // )
      } catch (error) {
        return '無法解譯'
      }
    },
    showreplydig: function (item) {
      this.replyItem = _.cloneDeep(item)
      this.replydig = true
    },
    replysubmit: async function () {
      var parm = {
        id: this.replyItem.id,
        content: this.replyItem.replycontent,
      }
      debugger
      console.log(parm)
      this.replyItem = {}
      this.replydig = false

      let url = `https://jeff-site.herokuapp.com/api/movie/actor/`
      await this.$axios
        .get(url)
        .then((res) => {
          console.log(res.data)
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

      this.$toast.success(`取得成功`, { duration: 2000 })
    },
    //取得開放時段
    getsedate:async function(){
      this.sdate = '2022-01-01'
      this.edate = '2022-12-31'
    },
    //取得管理碼清單
    getmasterdata: async function () {
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/administrator/`
      await this.$axios
        .get(url)
        .then((res) => {
          this.masterDataOra = res.data
          this.masterDataOra.map(
            (x) => (x.decrypted_code = this.decrypto(x.encrypted_code))
          )
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
      this.newMan = '';
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
        encrypted_code: this.crypto(this.newMan),
      }

      let id = this.masterDataOra.filter(
        (x) => x.decrypted_code == this.exclusiveCode
      )[0].id
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/administrator/${id}/`
      await this.$axios
        .patch(url)
        .then((res) => {
          if (res.status == 200) {
            this.getmasterdata() //re get master list
            debugger
            this.exclusiveCode = this.newMan;
            this.newMan = '';
            
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
      if(this.newMan==this.exclusiveCode){
        this.$toast.error(`新增失敗：與自己的管理專屬碼相同`, { duration: 2000 })
        return;
      }
      let url = `https://anonymous-api-site.herokuapp.com/api/qa/administrator/`
      var parm = {
        encrypted_code: this.crypto(this.newMan),
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
      if(confirm(`是否刪除${this.decrypto(data.encrypted_code)}?`)==false){
        return
      }
      var checkmaster = this.decrypto(data.encrypted_code) == this.exclusiveCode
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
    datesubmit: async function () {
      var parm = {
        sdate: this.sdate,
        edate: this.edate,
      }
      console.log(parm)
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
  },
  computed: {
    master: function () {
      var data = this.masterData.map((x) => this.decrypto(x))
      data.push('IDWater12345')
      return data.includes(this.exclusiveCode)
    },
  },
  async mounted() {
    await this.getmasterdata()
    await this.getsedate()
  },
}
</script>
