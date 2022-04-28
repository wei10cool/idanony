<template>
  <v-row justify="center" align="center">
    <v-col cols="12" sm="10">
      <v-card class="logo py-4 justify-center">
        <v-form ref="mainform">
          <v-card-text>
            <v-text-field
              outlined
              class="mx-2"
              v-model="exclusiveCode"
              type="password"
              placeholder="請輸入專屬碼"
              clearable
              ><span style="width: 80px" slot="prepend">專屬碼</span>
              <v-icon slot="append" color="success">mdi-reload</v-icon>
            </v-text-field>
          </v-card-text>
          <v-card-text
            ><v-btn
              @click="exclusiveContentshow = !exclusiveContentshow"
              class="error"
              >顯示問題反應區</v-btn
            ></v-card-text
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
              color="primary"
              :disabled="!exclusiveCode || exclusiveCode.length <= 0"
              ><v-icon>mdi-incognito</v-icon>送出</v-btn
            >
          </v-card-actions>
        </v-form>
      </v-card>
    </v-col>
    <v-col cols="12" sm="10">
      <v-data-table
        :headers="headers"
        :items="
          switchreply ? desserts.filter((x) => x.reply == true) : desserts
        "
        :footer-props="{ 'items-per-page-options': [-1, 25, 50, 100] }"
      >
        <template v-slot:top>
          <v-toolbar flat>
            <v-spacer></v-spacer>
            <v-switch
              v-model="switchreply"
              inset
              :label="`只顯示已回覆`"
            ></v-switch>
          </v-toolbar>
        </template>
        <template v-slot:[`item.action`]="{ item }">
          <v-btn v-if="exclusiveCode == materCode" color="warning">{{
            `${item.reply ? '編輯' : '開始'}回覆`
          }}</v-btn>
          <v-btn
            :disabled="!item.reply"
            color="primary"
            @click="showreplydig(item)"
            ><v-icon>mdi-message-reply-text-outline</v-icon>查看回覆</v-btn
          >
        </template>
      </v-data-table>
      <v-dialog v-model="replydig" width="500">
        <v-card light>
          <v-card-title>回覆內容</v-card-title>
          <v-card-text>{{ replyItem.replycontent }}</v-card-text>
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
      materCode: 'idwateradmin',
      exclusiveCode: '',
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
            'Lorem ipsum dolor sit, amet consectetur adipisicing elit. Impedit possimus optio nam quidem beatae, veritatis vel consequuntur mollitia corporis doloribus!',
        },
        {
          id: 2,
          time: '2022-01-01 12:00',
          content:
            'Lorem ipsum dolor, sit amet consectetur adipisicing elit. Aliquam, vitae?',
          reply: false,
          replycontent:
            'Lorem ipsum dolor sit, amet consectetur adipisicing elit. Impedit possimus optio nam quidem beatae, veritatis vel consequuntur mollitia corporis doloribus!',
        },
      ],
      replydig: false,
      replyItem: {},
      switchreply: false,
    }
  },
  methods: {
    showreplydig: function (item) {
      this.replyItem = item
      this.replydig = true
    },
  },
}
</script>
