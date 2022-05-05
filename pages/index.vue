<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="10">
      <v-card class="logo py-4 justify-center rounded-xl" light>
        <v-card-title class="justify-center display-1 pb-0"><span class="text--center">艾滴科技</span></v-card-title>
        <v-card-title class="justify-center display-2"><span class="text--center">執行長信箱</span></v-card-title>

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
              ><span style="width: 80px;font-size:1.3em;" slot="prepend">專屬碼</span>
              <v-icon slot="append" color="success">mdi-reload</v-icon>
            </v-text-field>
          </v-card-text>
          <v-card-text class="pt-0"
            ><v-btn
              @click="exclusiveContentshow = !exclusiveContentshow"
              class="info" style="color:white;"
              >顯示問題反應區</v-btn
            >
            <v-btn
              v-if="exclusiveCode == materCode"
              @click="exclusiveContentshow = !exclusiveContentshow"
              class="primary" style="color:white;"
              >修改管理專屬碼</v-btn
            >
            </v-card-text
          >
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
              color="primary" style="color:white;"
              :disabled="!exclusiveCode || exclusiveCode.length <= 0"
              ><v-icon>mdi-incognito</v-icon>送出</v-btn
            >
          </v-card-actions>
        </v-form>
      </v-card>
    </v-col>
    <v-col cols="12" sm="10">
      <v-data-table
      class="rounded-xl"
        :headers="headers"
        :items="
          switchreply ? desserts.filter((x) => x.reply == true) : desserts
        "
        :footer-props="{ 'items-per-page-options': [-1, 25, 50, 100] }" light
      >
        <template v-slot:top>
          <v-toolbar flat class="rounded-xl">
            <v-spacer></v-spacer>
            <v-switch
              v-model="switchreply"
              inset
              :label="`只顯示已回覆`"
            ></v-switch>
          </v-toolbar>
        </template>
        <template v-slot:[`item.action`]="{ item }">
          <v-btn
            style="color:white;"
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
          <v-card-text v-if="exclusiveCode == materCode">
            <v-form ref="replyform">
              <v-textarea filled v-model="replyItem.replycontent" clearable></v-textarea>
              <v-btn  class="error" tile>刪除??(應該是不需要這功能)</v-btn>
              <v-btn class="primary" tile :disabled="String(replyItem.replycontent).trim().length==0" @click="replysubmit">確定送出</v-btn>
            </v-form>
          </v-card-text>
          <v-card-text v-else>
            {{replyItem.reply == true}}
            {{ replyItem.replycontent }}
          </v-card-text>
        </v-card>
      </v-dialog>
    </v-col>
  </v-row>
</template>

<script>

export default {
  name: 'IndexPage',
  data() {
    return {
      materCode: 'IDWater12345',//管理碼
      exclusiveCode: 'IDWater12345',//專屬碼
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
      rules:{
        //pwdcheck:value=>(value && /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{6,}$/.test(value)) || '6位數，(需包含大寫字母、小寫字母、數字)',
        pwdcheck:value=>(value && /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[A-Za-z\d]{6,}$/.test(value)) || '6位數，(需包含大寫字母、小寫字母、數字)',
      }
    }
  },
  methods: {
    showreplydig: function (item) {
      this.replyItem = _.cloneDeep(item);
      this.replydig = true;
    },
    replysubmit:async function(){
      var parm = {
        id : this.replyItem.id,
        content: this.replyItem.replycontent,
      };
      console.log(parm);
      this.replyItem = {};
      this.replydig = false;

      let url = `https://test-website-jeff.herokuapp.com/api/movie/`;
      await this.$axios
        .get(url)
        .then(res => {
          
          console.log("取得API:" + res.request.responseURL);
        })
        .catch(error => {
          this.$toast.error(`取得失敗:${error}`, {
            duration: 2000
          });
        })
        .finally(() => {
          //this.getdata();
        });
   
      this.$toast.success(`取得成功`, { duration: 2000 });

    }
  },
}
</script>
