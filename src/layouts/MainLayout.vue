<template>
  <q-layout view="lHh Lpr lff">
    <q-header elevated class="bg-cyan-8">
      <q-toolbar>
        <q-btn flat @click="drawer = !drawer" round dense icon="menu" />
      </q-toolbar>
    </q-header>

    <q-drawer v-model="drawer" show-if-above :width="200" :breakpoint="400">
      <q-scroll-area
        style="
          height: calc(100% - 150px);
          margin-top: 150px;
          border-right: 1px solid #ddd;
        "
      >
        <q-list padding>
          <q-item
            clickable
            v-ripple
            v-for="(poll, index) in polls"
            :key="index"
            v-on:click="getQuestions(index + 1)"
          >
            <q-item-section avatar>
              <q-icon name="star" />
            </q-item-section>

            <q-item-section> {{ poll.poll_name }} </q-item-section>
          </q-item>
        </q-list>
      </q-scroll-area>

      <q-img
        class="absolute-top"
        src="https://cdn.quasar.dev/img/material.png"
        style="height: 150px"
      >
        <div class="absolute-bottom bg-transparent">
          <q-avatar size="56px" class="q-mb-sm">
            <img src="https://cdn.quasar.dev/img/boy-avatar.png" />
          </q-avatar>
        </div>
      </q-img>
    </q-drawer>

    <q-page-container>
      <ul id="example-1">
        <q-toolbar class="bg-purple text-white shadow-2 rounded-borders">
          <q-item-section> Encuesta {{ this.numeroe }} </q-item-section>
        </q-toolbar>
        <div class="div-1">
          <li v-for="(question, index) in questions" :key="index">
            <h6>{{ question.question_text }}</h6>

            <label class="form-control">
              <input
                type="radio"
                class="free-label four col"
                :id="index"
                :value="question.option_one"
                v-model="responses1[index]"
              />
              {{ question.option_one }}</label
            >
            <br />
            <label class="form-control">
              <input
                type="radio"
                class="free-label four col"
                :id="index + 1"
                :value="question.option_two"
                v-model="responses1[index]"
              />
              {{ question.option_two }}</label
            >
            <br />
          </li>
          <li v-for="(question, index) in questionso" :key="index">
            <h6>{{ question.question_text }}</h6>

            <input type="text" v-model="responses2[index]" />
            <br />
          </li>
          <li v-for="(question, index) in questionsm" :key="index">
            <h6>{{ question.question_text }}</h6>
            <label class="form-control">
              <input
                type="checkbox"
                class="option-input checkbox"
                id="jack"
                :value="question.option_one"
                v-model="checkedNames"
              />
              {{ question.option_one }}</label
            >
            <br />
            <label class="form-control">
              <input
                type="checkbox"
                class="option-input checkbox"
                id="john"
                :value="question.option_two"
                v-model="checkedNames"
              />
              {{ question.option_two }}</label
            >
            <br />
            <label class="form-control">
              <input
                type="checkbox"
                class="option-input checkbox"
                id="mike"
                :value="question.option_three"
                v-model="checkedNames"
              />
              {{ question.option_three }}</label
            >
            <br />

            <br />
            <q-btn
              color="amber"
              glossy
              label="Guardar"
              type="submit"
              class="btn btn-primary"
              v-on:click="checkForm()"
            />
          </li>
        </div>
      </ul>

      <!-- <router-view> </router-view> -->
    </q-page-container>
  </q-layout>
</template>

<script>
import { ref } from "vue";
import axios from "axios";
import Swal from "sweetalert2";

export default {
  setup() {
    const drawer = ref(false);
    return {
      drawer,
    };
  },
  data() {
    return {
      numeroe: 0,
      responses1: ["", ""],
      responses2: ["", ""],
      checkedNames: [],
      polls: [],
      questions: [],
      questionsm: [],
      questionso: [],
    };
  },
  methods: {
    getPolls() {
      const path = "http://127.0.0.1:8000/api/polls/pname/";
      axios
        .get(path)
        .then((response) => {
          this.polls = response.data;
        })
        .catch((error) => {
          console.log(error);
        });
    },
    getQuestions(numero) {
      console.log("resultado", numero);
      this.numeroe = numero;
      const path = "http://127.0.0.1:8000/api/polls/questions/";
      axios
        .get(path)
        .then((response) => {
          const questionst = response.data;
          this.questions = questionst.filter(
            (question) =>
              (question.poll == numero) & (question.type == "unique")
          );
          this.questionsm = questionst.filter(
            (question) =>
              (question.poll == numero) & (question.type == "multiple")
          );
          this.questionso = questionst.filter(
            (question) => (question.poll == numero) & (question.type == "open")
          );
        })
        .catch((error) => {
          console.log(error);
        });
    },
    checkForm() {
      this.errors = [];
      if (
        this.numeroe == 1 &&
        this.responses1[0] != "" &&
        this.responses1[1] != "" &&
        this.responses2[0] != "" &&
        this.responses2[1] != "" &&
        this.checkedNames[0] != undefined
      ) {
        this.postChoices();
      } else if (
        this.numeroe == 2 &&
        this.responses1[0] != "" &&
        this.responses2[0] != "" &&
        this.responses2[1] != "" &&
        this.checkedNames[0] != undefined
      ) {
        this.postChoices();
      } else if (
        this.numeroe == 3 &&
        this.responses1[0] != "" &&
        this.responses2[0] != "" &&
        this.checkedNames[0] != undefined
      ) {
        this.postChoices();
      } else {
        const Swal = require("sweetalert2");
        Swal.fire({ icon: "error", text: "Todos los campos son requeridos" });
        console.log(this.responses1[0]);
        console.log(this.responses2[0]);
        console.log(this.responses2[1]);
        console.log(this.checkedNames[0]);
      }
    },
    postChoices() {
      let post = {
        poll: this.numeroe,
        choice_q1: this.responses1[0],
        choice_q2: this.responses1[1],
        choice_q3: this.responses2[0],
        choice_q4: this.responses2[1],
        choice_q5:
          this.checkedNames[0] +
          "," +
          this.checkedNames[1] +
          "," +
          this.checkedNames[2],
      };
      axios
        .post("http://127.0.0.1:8000/api/polls/choice/", post)
        .then((result) => {
          console.log(result);
          const Swal = require("sweetalert2");
          if (result.status == 201) {
            Swal.fire({
              position: "top-end",
              icon: "success",
              title: "Gracias por responder la encuesta",
              showConfirmButton: true,
            });
            location.reload();
          } else {
            Swal.fire("Error!", result.statusText, "error");
          }
        });
    },
  },
  created() {
    this.getPolls();
    this.getQuestions();
  },
};
</script>
