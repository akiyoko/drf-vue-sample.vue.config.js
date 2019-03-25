<template>
  <div v-cloak>
    <!-- ヘッダナビゲーション -->
    <div id="header">
      <b-navbar type="dark" variant="dark">
        <a class="navbar-brand" href="/">
          <img src="assets/logo.png" width="120">
        </a>
      </b-navbar>
    </div>

    <!-- メッセージエリア -->
    <div id="messages">
      <b-alert variant="danger" show v-show="message.error" class="mb-0">
        {{ message.error }}
      </b-alert>
      <b-alert variant="warning" show v-show="message.warnings.length > 0" class="mb-0">
        <p v-for="(v, k, index) in message.warnings" :key="index" class="mb-0">{{ v[1][0] }}</p>
      </b-alert>
      <b-alert variant="info" show v-show="message.info" class="mb-0">
        {{ message.info }}
      </b-alert>
    </div>

    <!-- メイン -->
    <div id="home-page">
      <main class="container">
        <p class="h5 mb-4">ホーム</p>
        <b-form @submit.prevent="submitSave">
          <div class="row form-group">
            <label class="col-sm-3 col-form-label">タイトル</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" v-model="form.book.title">
            </div>
          </div>
          <div class="row form-group">
            <label class="col-sm-3 col-form-label">価格</label>
            <div class="col-sm-8">
              <input type="text" class="form-control" v-model="form.book.price">
            </div>
          </div>
          <div class="row text-center mt-5">
            <div class="col-sm-12">
              <b-button type="submit" variant="primary">{{ isCreated ? '更新' : '登録' }}</b-button>
            </div>
          </div>
        </b-form>
      </main>
    </div>

    <!-- For debug -->
    <div id="debug">
      <div class="container debug">
        <pre>{{ $data }}</pre>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from 'axios'

  // CSRFトークンの送信設定
  axios.defaults.xsrfCookieName = 'csrftoken'
  axios.defaults.xsrfHeaderName = 'X-CSRFToken'

  // APIクライアント
  const api = axios.create({
    baseURL: 'http://127.0.0.1:8000/api/v1/',
    timeout: 5000,
    headers: {
      'Content-Type': 'application/json',
      'X-Requested-With': 'XMLHttpRequest'
    }
  })

  // Vueインスタンスを作成
  export default {
    data () {
      return {
        form: {
          book: {
            title: '',
            price: 0
          }
        },
        message: {
          info: '',
          warnings: [],
          error: ''
        }
      }
    },
    computed: {
      isCreated: function () {
        return this.form.book.id !== undefined
      }
    },
    methods: {
      // 登録・更新ボタン押下時に呼び出されるメソッド
      submitSave: function () {
        this.clearMessages()
        api({
          method: this.isCreated ? 'put' : 'post',
          url: this.isCreated ? '/books/' + this.form.book.id + '/' : '/books/',
          data: {
            'id': this.form.book.id,
            'title': this.form.book.title,
            'price': this.form.book.price
          }
        })
          .then(response => {
            this.message.info = this.isCreated ? '更新しました。' : '登録しました。'
            this.form.book = response.data
          })
          .catch(error => {
            const status = error.response ? error.response.status : 500
            let message
            if (status === 400) {
              // バリデーションNG
              this.message.warnings = Object.entries(error.response.data)
            } else if (status === 401) {
              // 認証エラー
              this.message.error = '認証エラー'
            } else if (status === 403) {
              // 権限エラー
              this.message.error = '権限エラーです。'
            } else {
              // その他のエラー
              message = error.statusText ? error.statusText : '想定外のエラーです。'
              this.message.error = message
            }
          })
      },
      // メッセージエリアをクリア
      clearMessages: function () {
        this.message.error = ''
        this.message.warnings = []
        this.message.info = ''
      }
    }
  }
</script>

<style>
  #app {
    font-family: 'Avenir', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
  }
</style>
