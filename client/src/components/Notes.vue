<template>
  <div id="block">
    <div v-if="reg === true">
      <router-link to="/create" id="add-btn"
          ><a
            class="btn-floating btn-large waves-effect waves-light blue"
            to="/create"
            ><i class="material-icons">add</i></a
          ></router-link
        >
      <div class="loading" v-if="submitStatus === 'PENDING'">
        <div class="progress">
          <div class="indeterminate"></div>
        </div>
      </div>
        <h1>Заметки({{ filteredNotes(allNotes).length }})
        </h1>
      <div class="input-field col s6">
        <i class="material-icons prefix" id="icon">search</i>
        <input v-model="search" id="icon_prefix" type="text" class="validate" />
        <label for="icon_prefix">Найти</label>
      </div>
      <div v-if="filteredNotes(allNotes).length">
        <div
          v-for="(note, idx) in filteredNotes(allNotes)"
          :key="idx"
          class="notes"
        >
          <div class="note">
            <a
              class="delete-btn"
              v-on:click="delete_note(idx, note._id)"
            >
              <i class="material-icons">delete</i>
            </a>
            <div class="card" v-if="note.process !== 100">
              <h4 id="author">{{ note.title }}</h4>
              <h6 id="author">Автор: {{ note.author }}</h6>
              <div class="card-content">{{ note.content }}</div>
              <span id="time">{{ note.completeTime }} мин</span>
              <br />
              <span id="process">{{ note.process }}%</span>
              <br />
              <meter
                id="bar"
                min="0"
                low="50"
                max="100"
                optimum="80"
                v-bind:value="note.process"
              ></meter>
              <br />
              <input type="range" v-model="note.process" name="id" />
              <input hidden type="text" v-bind:value="note._id" name="id" />
              <a
                class="btn blue darken-4"
                v-on:click="comlete_note(note._id, note.process)"
                >Сохранить</a
              >
              <br />
              <b>{{ note.createdAt | formatDate }}</b>
            </div>
            <div class="card" id="completed" v-else>
              <i class="large material-icons">check_circle</i>
              <h4 id="author">{{ note.title }}</h4>
              <h6 id="author">Автор: {{ note.author }}</h6>
              <div class="card-content">{{ note.content }}</div>
              <span id="time">{{ note.completeTime }} мин</span>
              <br />
              <input hidden type="text" v-bind:value="note._id" name="id" />
              <br />
              <b>{{ note.createdAt | formatDate }}</b>
            </div>
            <br />
          </div>
        </div>
      </div>
      <div v-else>
        <img src="@/assets/notFound.jpg" width="50" height="50" />
        <h6>Ничего не найдено</h6>
      </div>
    </div>
    <div v-else>
      <h1>Страница недоступна</h1>
    </div>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import Config from "../Api-config";
export default {
  name: "Notes",
  data() {
    return {
      reg: JSON.parse(localStorage.getItem("auth")),
      submitStatus: null,
      search: "",
      token: localStorage.getItem("token"),
      username: localStorage.getItem("user"),
    };
  },
  computed: mapGetters(["allNotes"]),
  async mounted() {
    this.submitStatus = "PENDING";
    await this.$store.dispatch("fetchNotes");
    this.submitStatus = "OK";
  },
  methods: {
    delete_note(id,_id) {
      this.$store.commit('REMOVE_NOTE',this.$store.dispatch("fetchNotes"),id)
      this.$http({
        url: Config.todos_api + this.token + '/delete/'+ _id,
        method: "delete",
      });
      this.$store.dispatch("fetchNotes")
    },
    comlete_note(id, progress) {
      this.$store.commit('UPDATE_NOTE',this.$store.dispatch("fetchNotes"),progress, id)
      this.$http({
        url: Config.todos_api + this.token + "/complete",
        method: "post",
        data: { 
          id: id,
          process: progress,
        },
      });
      this.$store.dispatch("fetchNotes")
    },
    filteredNotes(todos) {
      const s = this.search.trim(" ").toLowerCase();
      return todos.filter((n) => {
        return Object.values(n).some((m) =>
          m.toString().toLowerCase().includes(s)
        );
      });
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1 {
  font-size: 2rem;
}
.note {
  padding-left: 1rem;
}
#author {
  padding-left: 1rem;
}
#time {
  color: rgb(133, 128, 128);
  padding-left: 1rem;
}
#process {
  padding-left: 0.5rem;
}
.notes {
  float: left;
  align-items: center;
}
#icon_prefix {
  width: auto;
}
#completed {
  background: linear-gradient(#64e675, #044113);
  color: white;
}
b {
  font-size: 0.9rem;
  margin-left: 0.4rem;
}
.head {
  display: block
}
#add-btn{
  position: absolute;
  width: 56px;
  height: 56px;
  left: 50%;
  right: 50%;
}
@media screen and (max-width: 414px ) {
    #add-btn{
    position: fixed;
    left: 65%;
    top: 40%;
  }
   }
</style>