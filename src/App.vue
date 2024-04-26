<template>
  <div>
    <div v-if="!gameFinished">
      <h2>{{ currentQuestion.text }}</h2>
      <div v-for="(input, index) in currentQuestion.inputs" :key="index">
        <input type="text" v-model="currentQuestion.inputs[index]" @input="onInputChange" />
      </div>
      <button :disabled="!isNextButtonEnabled" @click="nextQuestion">Next</button>
    </div>
    <div v-else>
      <h2>Random Number: {{ randomNumber }}</h2>
      <div v-for="(question, qIndex) in questions" :key="qIndex">
        <h3>{{ question.text }}</h3>
        <ul>
          <li v-for="(answer, aIndex) in question.answers" :key="aIndex"
            :class="{ crossed: answer.crossed, highlighted: answer.highlighted, current: answer === currentHighlight }">
            {{ answer.text }}
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      questions: [
        { text: 'Life partner', inputs: [''], answers: [] },
        { text: 'Number of kids', inputs: [''], answers: [] },
        { text: 'Job', inputs: [''], answers: [] },
        { text: 'Salary', inputs: [''], answers: [] },
        { text: 'Car', inputs: [''], answers: [] },
        { text: 'Where you live', inputs: [''], answers: [] },
      ],
      currentQuestionIndex: 0,
      gameFinished: false,
      randomNumber: null,
      currentHighlight: null,
      startingIndex: 0,
    };
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex];
    },
    isNextButtonEnabled() {
      return this.currentQuestion.inputs.filter(input => input.trim() !== '').length >= 2;
    },
  },
  methods: {
    onInputChange() {
      if (this.currentQuestion.inputs[this.currentQuestion.inputs.length - 1].trim() !== '') {
        this.currentQuestion.inputs.push('');
      }
    },
    nextQuestion() {
      this.currentQuestion.answers = this.currentQuestion.inputs.slice(0, -1).map(input => ({ text: input, crossed: false, highlighted: false }));
      this.currentQuestionIndex++;
      if (this.currentQuestionIndex === this.questions.length) {
        this.gameFinished = true;
        this.generateRandomNumber();
        this.startCrossingOut();
      }
    },
    generateRandomNumber() {
      this.randomNumber = Math.floor(Math.random() * 8) + 3;
    },
    startCrossingOut() {
      if (this.hasUncrossedAnswers()) {
        this.highlightAnswers();
      } else {
        this.currentHighlight = null;
        this.highlightRemainingAnswers();
      }
    },
    hasUncrossedAnswers() {
      return this.questions.some(({ answers }) => answers.filter(({ crossed }) => !crossed).length > 1);
    },
    highlightAnswers() {
      let count = 0;
      const highlightInterval = setInterval(() => {
        this.currentHighlight = null;
        const answer = this.getAnswerByIndex(this.startingIndex);
        this.currentHighlight = answer;
        count++;
        if (count === this.randomNumber) {
          clearInterval(highlightInterval);
          this.crossOutAnswer(answer);
        } else {
          this.startingIndex++;
          const uncrossedAnswers = this.getUncrossedAnswers();
          if (this.startingIndex >= uncrossedAnswers.length) {
            this.startingIndex = 0;
          }
        }
      }, 200);
    },
    getAnswerByIndex(index) {
      const uncrossedAnswers = this.getUncrossedAnswers();
      if (index >= 0 && index < uncrossedAnswers.length) {
        return uncrossedAnswers[index];
      }
      return null;
    },
    getTotalAnswers() {
      return this.questions.reduce((total, question) => total + question.answers.filter(answer => !answer.crossed).length, 0);
    },
    crossOutAnswer(answer) {
      let startingIndex = this.getAnswerIndex(answer);
      answer.crossed = true;
      const uncrossedAnswers = this.getUncrossedAnswers();
      if (startingIndex < 0) {
        startingIndex = uncrossedAnswers.length - 1;
      } else if (startingIndex >= uncrossedAnswers.length) {
        startingIndex = 0;
      }
      this.startingIndex = startingIndex;
      setTimeout(() => {
        this.startCrossingOut();
      }, 400);
    },
    getQuestionByAnswer(answer) {
      for (const question of this.questions) {
        if (question.answers.includes(answer)) {
          return question;
        }
      }
      return null;
    },
    getAnswerIndex(answer) {
      const uncrossedAnswers = this.getUncrossedAnswers(answer);
      return uncrossedAnswers.findIndex(a => a === answer);
    },
    getUncrossedAnswers(answerToExclude = null) {
      const uncrossedAnswers = [];
      for (const question of this.questions) {
        const questionUncrossedAnswers = question.answers.filter(a => !a.crossed);
        if (answerToExclude !== null && questionUncrossedAnswers.includes(answerToExclude) && questionUncrossedAnswers.length === 2) {
          uncrossedAnswers.push(answerToExclude);
        } else if (questionUncrossedAnswers.length > 1) {
          uncrossedAnswers.push(...questionUncrossedAnswers);
        }
      }
      return uncrossedAnswers;
    },
    highlightRemainingAnswers() {
      this.questions.forEach(question => {
        const remainingAnswer = question.answers.find(answer => !answer.crossed);
        if (remainingAnswer) {
          remainingAnswer.highlighted = true;
        }
      });
    },
  },
};
</script>

<style scoped>
.crossed {
  text-decoration: line-through;
}

.highlighted {
  font-weight: bold;
  color: green;
}

.current {
  background-color: yellow;
}
</style>