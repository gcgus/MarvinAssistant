<template>
<section id ="app">

    <!-- The input -->
    <div class="query">
        <div class="wrapper" v-if="micro == false">
            <i class="material-icons iicon" @click="microphone(true)">mic</i>
            <input :aria-label="config.locale.strings.queryTitle" autocomplete="off" v-model="query" class="queryform" @keyup.enter="submit()" :placeholder="config.locale.strings.queryTitle" autofocus type="text">
            <i class="material-icons iicon t2s" @click="mute(true)" v-if="muted == false">volume_up</i>
            <i class="material-icons iicon t2s" @click="mute(false)" v-else>volume_off</i>
        </div>
        <div class="wrapper" v-else>
            <i class="material-icons iicon recording" @click="microphone(false)">mic</i><input class="queryform" :placeholder="speech" readonly>   
        </div>
    </div>

    <main class="wrapper ai-window">

        <br>
        <br>

        <!-- Display Welcome Message -->
        <div v-if="answers.length == 0 && online == true">
            <h1 class="title mdc-typography--headline">
                <div class="material-icons up">arrow_upward</div>
                <br>
                <br>
                    {{config.locale.strings.welcomeTitle}}

                    <p class="mdc-typography--body2">{{config.locale.strings.welcomeDescription}}</p>
            </h1>
        </div>

        <!-- Display offline message -->
        <div v-if="answers.length == 0 && online == false">
            <h1 class="title mdc-typography--headline">
                <div class="material-icons up">cloud_off</div>
                <br>
                <br>
                    {{config.locale.strings.offlineTitle}}

                    <p class="mdc-typography--body2">{{config.locale.strings.offlineDescription}}</p>
            </h1>
        </div>

        <!-- Chat window -->
        <table v-for="a in answers" class="chat-window" :key="a.responseId">

            <!-- Your messages -->
            <tr>
                <td class="bubble">{{a.queryResult.queryText}}</td>
            </tr>


            <!-- Dialogflow messages -->
            <tr>
                <td>

                    <!-- Bot message types / Speech -->


                    <!-- Google Assistant output -->
                    <div v-for="(r, index) in a.queryResult.fulfillmentMessages" :key="index">

                        <div v-if="r.text">
                            <td class="bubble bot">{{r.text.text[0]}}</td>
                        </div>

                        <div v-if="r.message=='card'">
                            <td class="bubble bot">{{r.card.imageUrl}}</td>
                        </div>

                        <div class="card" v-if="r.message=='cards'">
                            <img class="card-image" v-if="r.card.imageUrl" :src="r.card.imageUrl" :alt="'Broken Link'" />
                                <div class="card-content">
                                  <div class="card-title" v-if="r.card.title">{{r.card.title}}</div>
                                  <div class="card-subtitle" v-if="r.card.subtitle">{{r.card.subtitle}}</div>
                                  <div class="card-text" v-if="r.card.text">{{r.card.text}}</div>
                                <slot></slot>
                              </div>
                        </div>

                        <!-- Bot message types / Card -->
          
                        <div class="mdc-card" v-if="r.message == 'cards'">
                            <img :alt="'Error'" class="mdc-card__media-item" :src="r.card.imageUrl" v-if="r.card.imageUrl">
                            <section class="mdc-card__primary">
                                <h1 class="mdc-card__title">{{r.card.title}}</h1>
                                <br>
                                <h2 class="mdc-card__subtitle">{{r.card.subtitle}}</h2>
                            </section>
                            <section class="mdc-card__supporting-text">
                                {{r.card.text}}
                            </section>
                            <section class="mdc-card__actions" v-for="(button, index) in r.buttons" :key="index">
                                <a class="mdc-button mdc-button--compact themed mdc-card__action" target="_blank" :href="button.openUrlAction.url">{{button.title}} <i class="material-icons openlink">open_in_new</i></a>
                            </section>
                        </div>

                        <!-- Bot message types / Carousel Card -->

                        <div class="slider" v-if="r.message == 'carousel_card'">
                            <carousel 
                                    :perPage="1" 
                                    :navigationEnabled="true"
                                    :paginationEnabled="false"
                                    navigationNextLabel="<button class='mdc-fab mdc-fab--mini material-icons rightnav'><span class='mdc-fab__icon'>keyboard_arrow_right</span></button>"
                                    navigationPrevLabel="<button class='mdc-fab mdc-fab--mini material-icons leftnav'><span class='mdc-fab__icon'>keyboard_arrow_left</span></button>"
                                    :navigationClickTargetSize="0"
                                    :loop="true">

                                <slide v-for="item in r.items" :key="item.id">
                                    <div class="mdc-card slide">
                                        <img class="mdc-card__media-item" :src="item.image.url" v-if="item.image">
                                        <section class="mdc-card__primary">
                                            <h1 class="mdc-card__title themed" @click="autosubmit(item.optionInfo.key)">{{item.title}}</h1>
                                        </section>
                                        <section class="mdc-card__supporting-text">
                                            {{item.description}}
                                        </section>
                                    </div>
                                </slide>
                            </carousel>
                        </div>

                        <!-- Bot message types / List -->

                        <div class="mdc-list-group mdc-card" v-if="r.message == 'list_card'">
                            <h3 class="mdc-list-group__subheader">{{r.title}}</h3>
                            <ul class="mdc-list mdc-list--two-line mdc-list--avatar-list" v-for="(item, index) in r.items" :key="index" @click="autosubmit(item.optionInfo.key)">
                                <li class="mdc-list-item">
                                    <img :title="item.image.accessibilityText" :alt="item.image.accessibilityText" class="mdc-list-item__start-detail" width="56" height="56" :src="item.image.url" v-if="item.image"/>
                                    <span class="mdc-list-item__text">
                                        {{item.title}}
                                    <span class="mdc-list-item__text__secondary">{{item.description}}</span>
                                    </span>
                                </li>
                            </ul>
                        </div>

                        <!-- Bot message types / Link Chip -->

                        <div v-if="r.message == 'link_out_chip'" class="chips">
                            <a class="suggestion link" :href="r.url" target="_blank">
                                {{r.destinationName}} <i class="material-icons openlink">open_in_new</i>
                            </a>
                        </div>

                        <!-- Bot message types / Suggestion Chip -->

                        <div v-if="r.message == 'suggestions'" class="chips">
                            <div class="suggestion" @click="autosubmit(s.title)" v-for="(s, index) in r.suggestions.suggestions" :key="index">
                                {{s.title}}
                            </div>
                        </div>

                        <!-- Bot message types / Google Suggestion Chip -->

                        <div v-if="r.unknown == true" class="google-chip chips">
                            <a class="suggestion" :href="'https://www.google.com/search?q=' + a.queryResult.queryText" target="_blank">
                                Search for "{{a.queryResult.queryText}}" on Google <i class="material-icons openlink">search</i>
                            </a>
                        </div>

                    </div>
                </td>
            </tr>
        </table>

        <br>
        <p class="copyright" v-if="answers.length > 0">Powered by <a href="https://dialogflow.com">Dialogflow</a></p>
        <a id="bottom"></a>
    </main>
</section>
</template>

<style lang="sass">
@import "App.sass";
</style>

<script>
import uuidv4 from "uuid/v4";
import axios from "axios";
import config from "../config";

const sessionId = uuidv4(); // Random Session ID
const apiUrl = "http://localhost:8000/api/";

export default {
  name: "app",
  data: function() {
    return {
      answers: [],
      query: "",
      speech: config.locale.strings.voiceTitle,
      micro: false,
      muted: config.app.muted,
      online: navigator.onLine,
      config
    };
  },
  watch: {
    answers: function(val) {
      setTimeout(() => {
        document.querySelector("#bottom").scrollIntoView({
          behavior: "smooth"
        });
      }, 2); // if new answers arrive, wait for render and then smoothly scroll down to #bottom selector, used as anchor
    }
  },
  methods: {
    submit() {
      let detectEndpoint =
        apiUrl +
        "detect?" +
        "sessionId=" +
        sessionId +
        "&langCode=" +
        config.locale.settings.recognitionLang +
        "&query=" +
        this.query;

      axios.get(detectEndpoint).then(response => {
        response = response.data; // Get Actual Data

        if (
          response.queryResult.action == "input.unknown" &&
          this.config.app.googleIt == true
        ) {
          response.queryResult.fulfillmentMessages[0].unknown = true;
          response.queryResult.fulfillmentMessages[0].text =
            response.queryResult.resolvedQuery;
        } // if the googleIt is enabled, show the button

        this.answers.push(response);
        this.handle(response); // <- handle the response in handle() method

        this.query = "";
        this.speech = config.locale.strings.voiceTitle; // <- reset query and speech
      });
    },
    handle(response) {
      if (
        response.outputAudio ||
        response.queryResult.fulfillmentMessages[0].platform ==
          "ACTIONS_ON_GOOGLE"
      ) {
        let speech = new SpeechSynthesisUtterance(
          response.outputAudio ||
            response.queryResult.fulfillmentMessages[0].simpleResponses
              .simpleResponses[0].textToSpeech
        );
        speech.voiceURI = "native";
        speech.lang = config.locale.settings.speechLang;

        if (this.muted == false) window.speechSynthesis.speak(speech); // <- Speech output if microphone is allowed
      }
    },
    autosubmit(suggestion) {
      this.query = suggestion;
      this.submit();
    },
    mute(mode) {
      this.muted = mode;
    },
    microphone(mode) {
      this.micro = mode;
      let self = this; // <- correct scope

      if (mode == true) {
        let recognition = new webkitSpeechRecognition(); // <- chrome speech recognition

        recognition.interimResults = true;
        recognition.lang = config.locale.settings.recognitionLang;
        recognition.start();

        recognition.onresult = function(event) {
          for (var i = event.resultIndex; i < event.results.length; ++i) {
            self.speech = event.results[i][0].transcript;
          }
        };

        recognition.onend = function() {
          recognition.stop();
          self.micro = false;
          self.autosubmit(self.speech);
        };
      }
    }
  }
};
</script>