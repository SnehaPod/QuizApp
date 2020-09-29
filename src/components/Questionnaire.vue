<template>
  <div class="qa-list">
    <b class="title">QUESTIONNAIRE</b>

    <form v-if="list.length" class="container py-2" @submit="validate">
      <div
        class="question-card mx-auto row"
        v-for="(question, qIndex) in list"
        :key="question.number"
      >
        <label v-bind:class="{ unanswered: unanswered[qIndex] }"
          >Q{{ question.number }}. {{ question.question }}</label
        >
        <div class="col-12 inline-flex">
          <div
            class="options row inline-flex"
            v-for="option in question.answers"
            :key="option"
          >
            <input
              class="radio-button"
              type="radio"
              :id="option"
              :value="option"
              v-model="selectedAnswers[qIndex]"
            />
            <label :for="option">{{ option }}</label>
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-3"></div>
        <div class="col-3">
          <button
            class="btn btn-success submit w-100 text-center"
            type="submit"
          >
            Submit
          </button>
        </div>

        <div class="col-3">
          <button
            @click="reset"
            class="btn btn-info reset w-100 text-center"
            type="reset"
          >
            Reset
          </button>
        </div>

        <div class="col-3"></div>
      </div>
    </form>

    <hr v-if="list.length && (answeredCorrectly || answeredIncorrectly)" />
    <b
      v-if="list.length && (answeredCorrectly || answeredIncorrectly)"
      class="title"
      >RESULTS</b
    >

    <div
      v-if="list.length && (answeredCorrectly || answeredIncorrectly)"
      class="chart-container mx-auto h-50"
    >
      <bar-chart
        :datasets="datasets"
        :labels="labels"
        :options="options"
      ></bar-chart>
    </div>

    <div v-if="!list.length" class="loader mx-auto my-5"></div>
  </div>
</template>

<script>
import axios from "axios";
import BarChart from "./BarChart";
export default {
  name: "Questionnaire",
  components: {
    BarChart,
  },
  data() {
    return {
      list: [],
      selectedAnswers: [],
      answeredCorrectly: 0,
      answeredIncorrectly: 0,
      unanswered: [],
      datasets: [],
      labels: [],
    };
  },
  mounted() {
    axios
      .get(
        "https://opentdb.com/api.php?amount=10&category=18&difficulty=medium&type=multiple"
      )
      .then((response) => {
        var questions = response.data.results;
        this.unanswered = new Array(questions.length).fill(false);
        this.selectedAnswers = new Array(questions.length).fill(undefined);
        for (var i = 0; i < questions.length; i++) {
          questions[i].incorrect_answers.push(questions[i].correct_answer);
          let questionObj = {
            number: i + 1,
            question: questions[i].question,
            answers: this.shuffle(questions[i].incorrect_answers),
            correct_answer: questions[i].correct_answer,
          };
          this.list.push(questionObj);
        }
      });
  },
  methods: {
    validate: function (e) {
      e.preventDefault();
      for (var i = 0; i < this.selectedAnswers.length; i++) {
        if (!this.selectedAnswers[i]) {
          this.$set(this.unanswered, i, true);
        } else {
          this.$set(this.unanswered, i, false);
        }
      }
      if (this.unanswered.every((el) => el == false)) {
        this.calculateScore();
        this.datasets = [];
        this.datasets.push({
          label: "Quiz Results",
          backgroundColor: ["green", "red"],
          data: [this.answeredCorrectly, this.answeredIncorrectly],
        });
        this.options = {
          responsive: false,
          scales: {
            yAxes: [
              {
                gridLines: {
                  display: true,
                },
                ticks: {
                  beginAtZero: true,
                  max: this.list.length,
                  stepSize: 1,
                },
              },
            ],
          },
        };
        this.labels = ["Correct Answers", "Incorrect Answers"];
      }
    },
    shuffle(array) {
      array.sort(() => Math.random() - 0.5);
      return array;
    },
    reset: function () {
      this.unanswered = new Array(this.list.length).fill(false);
      this.selectedAnswers = new Array(this.list.length).fill(undefined);
      this.answeredCorrectly = 0;
      this.answeredIncorrectly = 0;
    },
    calculateScore: function () {
      console.log("this.list", this.list);
      for (var i = 0; i < this.list.length; i++) {
        if (this.selectedAnswers[i] === this.list[i]["correct_answer"]) {
          this.answeredCorrectly++;
        } else {
          this.answeredIncorrectly++;
        }
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.radio-button {
  margin-top: 5px;
  margin-right: 5px;
}

.submit {
  margin-top: 20px;
  text-align: right;
}

.reset {
  margin-top: 20px;
  text-align: left;
}

.question-card {
  border: 2px solid black;
  border-radius: 5px;
  text-align: left;
  margin-top: 10px;
  padding: 5px;
  width: 70%;
  font-family: "Lucida Sans", "Lucida Sans Regular", "Lucida Grande",
    "Lucida Sans Unicode", Geneva, Verdana, sans-serif;
}

.loader {
  border: 10px solid #f3f3f3;
  border-radius: 50%;
  border-top: 10px solid grey;
  width: 120px;
  height: 120px;
  -webkit-animation: spin 2s linear infinite; /* Safari */
  animation: spin 2s linear infinite;
}

.unanswered {
  color: red;
}

.chart-container {
  width: 50%;
  border: 2px solid black;
  margin-bottom: 10px;
}

/* Safari */
@-webkit-keyframes spin {
  0% {
    -webkit-transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
