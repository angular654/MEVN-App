<template>
  <div id="block">
    <div v-if="reg === true">
      <form @submit.prevent="logout">
        <h2>Вы уже вошли в ToDoApp</h2>
        <button class="btn blue darken-4" type="submit" name="action">
          Выйти
        </button>
        <p class="ok" v-if="submitStatus === 'OK'">Готово!</p>
      </form>
    </div>
    <div v-else>
      <h2 id="file-text">Войти</h2>
      <form @submit.prevent="submit">
        <label for="name">Name</label>
        <input type="text" v-model="name" required />
        <span v-if="msg.name">{{ msg.name }}</span>
        <br />
        <label for="password">Password:</label>
        <input type="password" v-model="password" required />
        <br />
        <span v-if="msg.password">{{ msg.password }}</span>
        <br />
        <button class="btn blue darken-4" type="submit" name="action">
          Войти
        </button>
        <p class="ok" v-if="submitStatus === 'OK'">Готово!</p>
        <p class="loading" v-if="submitStatus === 'PENDING'">Вход...</p>
        <p class="error" v-if="submitStatus === 'ERROR'">{{ errors }}</p>
      </form>
    </div>
  </div>
</template>
<script>
import Config from "../Api-config";
export default {
  name: "Signin",
  data() {
    return {
      name: "",
      errors: "",
      password: "",
      msg: [],
      submitStatus: null,
      reg: JSON.parse(localStorage.getItem("auth")),
    };
  },
  watch: {
    password(value) {
      this.password = value;
      this.validatePassword(value);
    },
    name(value) {
      this.name = value;
      this.validateName(value);
    },
  },
  methods: {
    validatePassword(value) {
      let difference = 10 - value.length;
      if (value.length < 8) {
        this.msg["password"] =
          "Длина  пароля должна быть минимум 10 символов! " +
          `осалось символов : ${difference}`;
      } else {
        this.msg["password"] = "";
      }
    },
    validateName(value) {
      let difference = 5 - value.length;
      if (value.length < 5) {
        this.msg["name"] =
          "Длина  логина  должна быть минимум 5 символов! " +
          `осалось символов : ${difference}`;
      } else {
        this.msg["name"] = "";
      }
    },
    async submit() {
      this.submitStatus = "PENDING";
      await this.$http
        .post(Config.auth_api + "signin", {
          username: this.name,
          password: this.password,
        })
        .then((response) => {
          if (response.data.token) {
            localStorage.setItem("token", response.data.token);
            localStorage.setItem("user", response.data.user);
            localStorage.setItem("user_id", `${response.data.user_id}`);
            localStorage.setItem("auth", response.data.auth);
            this.$router.push("/");
            this.submitStatus = "OK";
          } else {
            this.submitStatus = "ERROR";
            this.errors = response.data.status;
          }
        });
    },
    async logout() {
      await this.$http({
        url: Config.auth_api + "signout",
        method: "post",
        data: {
          username: this.name,
        },
      });
      this.reg = false;
      localStorage.clear();
      localStorage.setItem("auth", false);
    },
  },
};
</script>
<style scoped>
h2 {
  font-size: 2rem;
}
span {
  color: rgb(199, 15, 15);
  font-weight: bold;
}
.error {
  color: rgb(199, 15, 15);
  font-weight: bold;
}
.ok {
  font-weight: bold;
  color: rgb(63, 63, 173);
}
.loading {
  font-weight: bold;
  color: rgb(87, 179, 148);
}
</style>