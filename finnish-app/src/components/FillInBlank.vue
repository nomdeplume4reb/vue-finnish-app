<template>
  <div id='fill-in-blank-container'>
    <h1>Finnish Fill-in-the-Blank</h1>
    <h4 class='instructions'> Fill in the blank with the correct option below. </h4>
    <div class='sentence-container'>
      <h3 id='sentence'>{{ sentenceWithBlank }}</h3>
    </div><br />
    <form id='options-list' name='options-list'>
        <input type='radio' name='options' :value="option1" v-model="selected" />
          <label htmlFor='option1'> {{ option1 }}</label> <br />
        <input type='radio' name='options' :value='option2' v-model="selected" />
          <label htmlFor='option2'> {{ option2 }}</label><br />
        <input type='radio' name='options' :value='option3' v-model="selected" />
          <label htmlFor='option3'> {{ option3 }}</label><br />
        <input type='radio' name='options' id='invisible-option' value='invisible-option' v-model="selected" />
          <label htmlFor='invisible-option'></label><br />
    </form><br />
    <button class='button-check' @click='handleCheck()'>Check</button><br /><br />
    <p v-if="checking">{{ displayCheck }}</p>
    <button class='button-next' @click='handleNext()'>Next</button><br />
    <p>Score: 0 / 0</p>
  </div>
</template>

<script>
import _ from 'lodash';
// import natural from 'natural';
import nlp from 'compromise/one';
import wordBank from "../wordbank.json";

export default {
  name: 'FillInBlank',
  props: {},
  data() {
    return {
      randomArticleReqURL: "https://fi.wikipedia.org/w/api.php?format=json&action=query&origin=*&generator=random",
      randomArticleId: "5505",
      articleReqUrlBase: "https://fi.wikipedia.org/w/api.php?format=json&action=query&origin=*&prop=extracts&exintro&explaintext&redirects=1&pageids=",
      articleText: "",
      sentenceBank: [],
      sentence: '',
      correctWord: '',
      sentenceWithBlank: 'Select "NEXT" to get a randomly generated sentence',
      wordBank: wordBank,
      wordInfo: false,
      option1: "",
      option2: "",
      option3: "",
      selected: 'invisible-option',
      displayCheck: "",
      checking: false,
    }
  },
  mounted () {
    this.fetchArticleId()
    // await this.fetchArticleText();
    // this.searchArticleUntilWordFound()
    // console.log(this.randomArticleId)
  },
  created () {
  },
  computed: {
  },
  methods: {
    handleCheck(){
      this.checking = !this.checking;

      if(this.selected == this.correctWord){
        this.displayCheck = "Correct!"
      } else {
        this.displayCheck = "Nope! Try again!"
      }
    },
    handleNext(){
      this.searchArticleUntilWordFound();
      this.selected = 'invisible-option';
      this.checking = false;
      this.wordInfo = false;
    },
    searchArticleUntilWordFound(){

      const sentenceIndex = _.random(0, this.sentenceBank.length-1)

      this.sentence = this.sentenceBank[sentenceIndex]

      this.fetchArticleId();
      this.fetchArticleText(this.randomArticleId);
      this.tokenizeArticleSentences();
      this.selectSentenceFromBank();
      this.selectWordfromSentence();
      this.findWordForms();

    },
    // async fetchArticleId(){
    //
    //     try {
    //       const response = await fetch(this.randomArticleReqURL, {method: "GET"})
    //       // console.log(response)
    //       const json = await response.json()
    //       // console.log(json)
    //       this.randomArticleId = _.keys(json.query?.pages)[0];
    //       console.log(this.randomArticleId)
    //     } catch (err){
    //       console.log(err)
    //     }
    //
    //     // console.log(this.randomArticleId)
    // },
    // async fetchArticleText(articleId){
    //     try {
    //       const endpoint = this.articleReqUrlBase + articleId
    //       // fix error message when extract is not found
    //
    //       const response = await fetch(endpoint)
    //       const json = await response.json()
    //
    //       const extract = json.query?.pages[articleId]?.extract || []
    //       if (extract.length > 100){
    //         this.articleText = extract
    //       } else {
    //         this.fetchArticleId()
    //       }
    //       console.log(this.articleText)
    //     } catch(error) {
    //       console.log(error.message);
    //     }
    // },

    async fetchArticle(){
      try {
        const IDresponse = await fetch(this.randomArticleReqURL, {method: "GET"})
        const IDjson = IDresponse.json()
        this.randomArticleId = _.keys(IDjson.query?.pages)[0];

        const endpoint = this.articleReqUrlBase + this.randomArticleId
        // fix error message when extract is not found

        const response = await fetch(endpoint)
        const json = response.json()

        const extract = json.query?.pages[this.randomArticleId]?.extract || []
        if (extract.length > 100){
          this.articleText = extract
        } else {
          this.fetchArticleId()
        }
        console.log(this.articleText)
      } catch(error) {
        console.log(error.message);
      }
    },

    fetchArticleId(){
        fetch(this.randomArticleReqURL, {method: "GET"})
            .then(response => response.json())
            .then(json => {
              // console.log(JSON.stringify(json, null, 5));
              this.randomArticleId = _.keys(json.query.pages)[0];
            })
            .catch(error => {
              console.log(error.message);
            })
            .then(this.fetchArticleText(this.randomArticleId))
    },
    async fetchArticleText(articleId){
        const endpoint = this.articleReqUrlBase + articleId
        // fix error message when extract is not found
        await fetch(endpoint)
            .then(response => response.json())
            .then(json => {
              // console.log(JSON.stringify(json.query.pages[this.randomArticleId].extract, null, 5));
              const extract = json.query?.pages[articleId]?.extract || []
              if (extract.length > 100){
                this.articleText = extract
              } else {
                this.fetchArticleId()
              }
            })
            .catch(error => {
              console.log(error.message);
            })
    },
    tokenizeArticleSentences(){
      const doc=nlp(String(this.articleText))
      this.sentenceBank = doc.json().map(o=> o.text)
      // does not correctly recognize sentence boundaries or nonlatin characters
    },
    selectSentenceFromBank(){
      const sentenceIndex = _.random(0, this.sentenceBank.length-1)

      this.sentence = this.sentenceBank[sentenceIndex]
    },
    selectWordfromSentence(){
      // TEMPORARY SOLUTION!
      // Real solution should use Finnish NLP library to recognize proper nouns and punctuation

      const doc = nlp(this.sentence).terms().json()

      const compare = []
      const tokenized = []

      _.forEach(doc, (w)=> {
        compare.push(w.text)
        const word = w.text
        //remove words with capital letters, numbers; remove special characters
        if (word[0].toString().toUpperCase() != word[0].toString() && !/\d/.test(word)){
          tokenized.push(word.replace(/[.,#!$%^&*;:{}=-_`~()]/g,""))
        }
      })

      this.correctWord = tokenized[_.random(0, tokenized.length-1)]

      this.sentenceWithBlank = this.sentence.replace(this.correctWord, " ____ ")
    },
    findWordForms(){
      // TEMPORARY SOLUTION!
      // Real solution should use Finnish NLP library to lemmatize
      // also should check sentenceBank before fetching new article

      _.forEach(this.wordBank, (word) => {
        if(_.includes(word.taivutus, this.correctWord) || word.word == this.correctWord){
          this.wordInfo = word
        }
      })

      if (this.wordInfo){
        const options = [this.correctWord]
        options.push(...this.randomSample(this.wordInfo.taivutus, 2))
        const shuffledOptions = [...options].sort(() => 0.5 - Math.random());
        this.option1 = shuffledOptions[0];
        this.option2 = shuffledOptions[1];
        this.option3 = shuffledOptions[2];
        // console.log(this.shuffledOptions)
      } else {
        console.log(this.correctWord, "not found")
        this.fetchArticleId();
        this.fetchArticleText(this.randomArticleId);
        this.tokenizeArticleSentences();
        this.selectSentenceFromBank();
        this.selectWordfromSentence();
        this.findWordForms();
      }
    },
    randomSample(arr, num) {
      const shuffled = [...arr].sort(() => 0.5 - Math.random());
      return shuffled.slice(0, num);
    }
  },
  watch: {
  }
}



</script>

<style scoped>

#fill-in-blank-container {
  max-width: 700px;
  margin: 30px auto;
  overflow: auto;
  min-height: 300px;
  border: 1px solid black;
  padding: 30px;
  border-radius: 5px;
  background: white;
  opacity: .8;
}

.button-check, .button-next  {
    background-color: transparent;
    border-radius: 4px;
    border: 0;
    box-shadow: inset 0 0 0 2px #585858;
    color: #585858 !important;
    cursor: pointer;
    display: inline-block;
    font-size: 0.8em;
    font-weight: 900;
    height: 3.5em;
    letter-spacing: 0.35em;
    line-height: 3.45em;
    overflow: hidden;
    padding: 0 1.25em 0 1.6em;
    text-align: center;
    text-decoration: none;
    text-overflow: ellipsis;
    text-transform: uppercase;
    white-space: nowrap;
  }

.button-check:hover {
    color: #6495ED !important;
    box-shadow: inset 0 0 0 2px #6495ED;
}

.button-next {
    box-shadow: none;
    background-color: #585858;
    color: #ffffff !important;
}

.button-next:hover {
    background-color: #6495ED;
}

.button-check:active {
    background-color: rgba(224, 255, 255, 0.7);
}

.button-next:active {
    box-shadow: none;
    background-color: #191970;
    color: #ffffff !important;
    opacity: .7;
}

input[type='radio']:after {
    width: 15px;
    height: 15px;
    border-radius: 15px;
    top: -2px;
    left: -1px;
    position: relative;
    background-color: #d1d3d1;
    content: '';
    display: inline-block;
    visibility: visible;
    border: 2px solid white;
}

input[type='radio']:checked:after {
    background-color: #6495ED;
}

input[type='radio']:hover {
    top: -1px;
    left: -1px;
    position: relative;
}

#invisible-option {
   display: none;
}

.sentence-container {
  display: flex;
  justify-content: center;
  align-items: center;
  max-width: 600px;
  height: 100px;
  overflow: hidden;
  border: 1px solid black;
  padding: 5px;
  border-radius: 5px;
}

</style>

// https://github.com/axa-group/nlp.js
// http://naturalnode.github.io/natural/
// https://github.com/spencermountain/compromise/

// random article generator:
// https://fi.wikipedia.org/w/api.php?format=json&action=query&generator=random

//get article by title
// "https://fi.wikipedia.org/w/api.php?format=json&action=query&prop=extracts&exintro&explaintext&redirects=1&titles=Stack%20Overflow"

// get article by ID
// "https://fi.wikipedia.org/w/api.php?format=json&action=query&origin=*&prop=extracts&exintro&explaintext&redirects=1&pageids=1209635"
