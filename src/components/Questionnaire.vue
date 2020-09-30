<template>
  <div class="qa-list">
    <b class="title" ref="top">QUESTIONNAIRE</b>

    <form v-if="list.length" class="container py-2" @submit="validate">
      <div
        class="question-card mx-auto row"
        v-for="(question, qIndex) in list"
        :key="question.number"
      >
        <label v-bind:class="{ unanswered: unanswered[qIndex] }"
          v-html="'Q' + question.number + '. ' + question.question">
          </label
        >
        <div class="col-12">
          <div
            class="options row"
            v-for="(option, aIndex) in question.answers"
            :key="option"
          >
            <label
              class="card card-options"
              :id="'Q' + qIndex + '_' + question.answers[aIndex]"
            >
              <input
                class="radio-button"
                type="radio"
                :value="option"
                v-model="selectedAnswers[qIndex]"
              />
              <span v-html="option"></span></label>
          </div>
        </div>
      </div>

      <div v-show="alert" class="row mx-auto">
        <div class="col-12 mt-2">
          <span class="text-danger"> Please answer all the questions ! </span>
        </div>
      </div>

      <div class="row mx-auto">
        <div class="col-sm-12 col-md-3 offset-md-3">
          <button class="btn btn-success submit text-center" type="submit">
            Submit
          </button>
        </div>

        <div class="col-sm-12 col-md-3">
          <button
            @click="reset"
            class="btn btn-info reset text-center"
            type="reset"
          >
            Clear
          </button>
        </div>
      </div>
    </form>

    <hr v-if="list.length && (answeredCorrectly || answeredIncorrectly)" />
    <b
      v-if="list.length && (answeredCorrectly || answeredIncorrectly)"
      class="title"
      >RESULTS</b
    >

    <div
      id="results"
      ref="resultView"
      v-if="list.length && (answeredCorrectly || answeredIncorrectly)"
      class="chart-container mx-auto"
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
      options: {},
      alert: false,
    };
  },
  mounted() {
    axios
      .get(
        "https://opentdb.com/api.php?amount=10&type=multiple"
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
        this.alert = false;
        this.calculateScore();
        this.datasets = [];
        this.datasets.push({
          label: "Quiz Results",
          backgroundColor: ["green", "red"],
          data: [this.answeredCorrectly, this.answeredIncorrectly],
        });
        this.options = {
          responsive: true,
          maintainAspectRatio: false,
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
        // After the whole view is rendered, use the reference (for v-if element)
        this.$nextTick(() => {
          this.$refs["resultView"].scrollIntoView({behavior: "smooth"});
        });
      } else {
        this.alert = true;
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
      this.alert = false;
      for (var i = 0; i < this.list.length; i++) {
        for (var j = 0; j < this.list[i].answers.length; j++) {
          let answerElement = document.getElementById(
            "Q" + i + "_" + this.list[i].answers[j]
          );
          answerElement.style.border = "";
        }
      }
      this.$refs["top"].scrollIntoView({behavior: "smooth"});
    },
    calculateScore: function () {
      for (var i = 0; i < this.list.length; i++) {
        if (this.selectedAnswers[i] === this.list[i]["correct_answer"]) {
          this.answeredCorrectly++;
          let correctAnswerElement = document.getElementById(
            "Q" + i + "_" + this.list[i]["correct_answer"]
          );
          correctAnswerElement.style.border = "thin solid #016148";
        } else {
          this.answeredIncorrectly++;
          let wrongAnswerElement = document.getElementById(
            "Q" + i + "_" + this.selectedAnswers[i]
          );
          wrongAnswerElement.style.border = "thin solid #FF0000";
          let correctAnswerElement = document.getElementById(
            "Q" + i + "_" + this.list[i]["correct_answer"]
          );
          correctAnswerElement.style.border = "thin solid #016148";
        }
      }
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.title {
  /* font-size: 20px; */
}

.radio-button {
  margin-top: 5px;
  margin-right: 5px;
}

.submit {
  margin-top: 20px;
  text-align: right;
  width: 100%;
}

.reset {
  margin-top: 20px;
  text-align: left;
  width: 100%;
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
  margin-bottom: 10px;
}

.options {
  display: inherit !important;
  overflow-wrap: break-word;
}

.card-options {
  display: inherit !important;
  padding: 5px;
  overflow-wrap: break-word;
}

@media screen and (max-width: 760px) {
  .submit {
    margin-top: 20px;
    text-align: right;
    width: 70%;
  }

  .reset {
    margin-top: 20px;
    text-align: left;
    width: 70%;
  }

  body {
    font-size: 12px;
  }
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
