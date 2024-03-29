<script>
import axios from "axios";
import { Toast } from "bootstrap";
import WordHighlighter from "vue-word-highlighter";
import { useMagicKeys, whenever, useSwipe, SwipeDirection } from "@vueuse/core";
import { computed } from "vue";

const nocoxios = axios.create();

const config = {
  keywords: [],
  endpoint: import.meta.env.VITE_ENDPOINT,
  authToken: null,
};

const api = config.endpoint;

const texts = {
  no_more_results: "no more results",
  updated: "updated",
};

export default {
  name: "Papers",
  components: { WordHighlighter },
  computed: {
    getCurrentLink() {
      if (!this.current || !this.current.DOILink || this.current.DOILink == "")
        return;
      if (this.current.DOILink.startsWith("http")) return this.current.DOILink;
      else return `https://doi.org/${this.current.DOILink}`;
    },
  },
  methods: {
    loadToken() {
      const cname = "token";
      let name = cname + "=";
      let ca = document.cookie.split(";");
      for (let i = 0; i < ca.length; i++) {
        let c = ca[i];
        while (c.charAt(0) == " ") {
          c = c.substring(1);
        }
        if (c.indexOf(name) == 0) {
          return c.substring(name.length, c.length);
        }
      }
      return null;
    },

    setToken() {
      nocoxios.defaults.headers.common["xc-token"] = this.token;
      const exdays = 20;
      const cname = "token";
      const cvalue = this.token;
      const d = new Date();
      d.setTime(d.getTime() + exdays * 24 * 60 * 60 * 1000);
      let expires = "expires=" + d.toUTCString();
      document.cookie = cname + "=" + cvalue + ";" + expires + ";path=/";
      this.loadNext();
    },
    reload() {
      location.reload();
    },
    loadNext() {
      if (this.list.length > 0) this.current = this.list.shift();
      else {
        this.isLoading = true;
        nocoxios
          .get(
            `${api}?${encodeURIComponent(
              import.meta.env.VITE_NOCODB_QUERY
            )}&offset=${this.skip}`
          )
          .then((response) => {
            this.err = false;
            const { list, pageInfo } = response.data;
            this.list = list;
            this.isLastPage = pageInfo.isLastPage;
            this.totalRows = pageInfo.totalRows;
            this.page = pageInfo.page;
            if (this.list.length > 0) this.current = this.list.shift();
            else {
              const errToast = new Toast(document.getElementById("err-toast"), {
                autohide: false,
              });
              this.lastState = texts.no_more_results;
              this.err = true;
              errToast.show();
            }
          })
          .catch(() => {
            this.err = true;
          })
          .finally(() => {
            this.isLoading = false;
          });
      }
    },
    updateCurrent() {
      const toast = new Toast(document.getElementById("toast"), {
        autohide: true,
        delay: 500,
      });

      if (this.isLoading) {
        return;
      }

      this.isLoading = true;

      nocoxios
        .patch(`${api}/${this.current.ID}`, {
          RelevantTf: this.current.RelevantTf,
          ArtefactTf: this.current.ArtefactTf,
          FocusTf: this.current.FocusTf,
        })
        .then((resp) => {
          toast.show();
          this.lastState = `${resp.data.ID} ${texts.updated}`;
          this.loadNext();
        })
        .catch((err) => {
          const errToast = new Toast(document.getElementById("err-toast"), {
            autohide: false,
          });
          this.lastState = err;
          this.err = true;
          errToast.show();
        })
        .finally(() => {
          this.isLoading = false;
        });
    },
    isValidMode() {
      return !this.isLoading && !this.err && this.token;
    },
    getStart() {
      return {
        Title: "This is PaperSwipe",
        Abstract:
          "Introducing PaperSwipe – the revolutionary new tool that makes it easy to classify research papers. Just like a dating app, PaperSwipe allows you to swipe left or right to quickly categorize papers based on your interests. With PaperSwipe, you'll never have to spend hours sorting through stacks of papers again!",
        Type: "repo",
        Source: "tobias@dtdi.de",
        DOILink: "https://tobias-fehrer.de",
        Authors: "Tobias Fehrer",
      };
    },
  },

  data() {
    return {
      list: [],
      isLastPage: null,
      page: -1,
      totalRows: null,
      current: null,
      fromYear: 1980,
      skip: 0,
      lastState: null,
      token: null,
      err: false,
      isLoading: false,
      isSwiping: false,
      swipeTendency: null,
      left: 0,
      opacity: 1,
      queries: import.meta.env.VITE_HIGHLIGHT_REGEX,
    };
  },
  mounted() {
    const card = this.$refs.card;
    const that = this;
    const containerWidth = computed(() => this.$refs.container.offsetWidth);
    const { isSwiping, direction, lengthX } = useSwipe(card, {
      passive: true,
      onSwipe(e) {
        if (that.isValidMode() && containerWidth.value) {
          if (
            lengthX.value != 0 &&
            (direction.value == SwipeDirection.LEFT ||
              direction.value == SwipeDirection.RIGHT)
          ) {
            const length = Math.abs(lengthX.value);
            that.left = `${-lengthX.value}px`;
            that.opacity = 1.1 - length / containerWidth.value;

            if (Math.abs(lengthX.value) / containerWidth.value >= 0.5) {
              that.swipeTendency = direction.value;
            }
          } else {
            that.left = "0";
            that.opacity = 1;
            that.swipeTendency = null;
          }
        }
      },
      onSwipeEnd(e, direction) {
        //console.log(e, direction, lengthX.value, lengthY.value)

        if (
          containerWidth.value &&
          Math.abs(lengthX.value) / containerWidth.value >= 0.4
        ) {
          if (direction == SwipeDirection.LEFT) {
            if (!that.isValidMode()) return;
            that.current.RelevantTf = "0 - no";
            that.updateCurrent();
          }
          if (direction == SwipeDirection.RIGHT) {
            if (!that.isValidMode()) return;

            that.current.RelevantTf = "2 - yes";
            that.updateCurrent();
          }
        }

        that.left = "0";
        that.opacity = 1;
        that.swipeTendency = null;
        return e;
      },
    });
    this.isSwiping = isSwiping;
  },
  created() {
    this.token = this.loadToken();
    this.current = this.getStart();
    this.loadNext();

    const keys = useMagicKeys();

    whenever(keys.a, () => {
      if (!this.isValidMode()) return;
      this.current.RelevantTf = "0 - no";
      this.updateCurrent();
    });
    whenever(keys.left, () => {
      if (!this.isValidMode()) return;
      this.current.RelevantTf = "0 - no";
      this.updateCurrent();
    });
    whenever(keys.s, () => {
      if (!this.isValidMode()) return;
      this.current.RelevantTf = "1 - mid";
      this.updateCurrent();
    });

    whenever(keys.d, () => {
      if (!this.isValidMode()) return;

      this.current.RelevantTf = "2 - yes";
      this.updateCurrent();
    });
    whenever(keys.right, () => {
      if (!this.isValidMode()) return;

      this.current.RelevantTf = "2 - yes";
      this.updateCurrent();
    });
  },
};
</script>

<template>
  <div class="container site-container" ref="container">
    <div
      class="card card-swipe"
      ref="card"
      :class="{
        animated: !isSwiping,
        'bg-danger text-white': swipeTendency === 'LEFT',
        'bg-primary text-white': swipeTendency === 'RIGHT',
      }"
      :style="{ left, opacity }"
    >
      <div class="card-body marked">
        <WordHighlighter wrapper-tag="h5" :query="queries" class="card-title">{{
          current.Title
        }}</WordHighlighter>

        <h6 class="card-subtitle mb-2 text-muted">
          {{ current.Year }} {{ current.Source }}
        </h6>

        <div v-if="!this.token || this.err" class="input-group mb-3">
          <input
            v-model="fromYear"
            type="number"
            class="form-control"
            placeholder="From year"
          />
          <input
            v-model="token"
            type="text"
            class="form-control"
            placeholder="API Token"
            aria-describedby="button-addon2"
          />
          <button
            class="btn btn-outline-secondary"
            type="button"
            id="button-addon2"
            @click="setToken"
          >
            Save
          </button>
        </div>
        <WordHighlighter
          wrapper-tag="p"
          :query="queries"
          :case-sensitive="false"
          class="card-text text-read"
          >{{ current.Abstract }}</WordHighlighter
        >
      </div>
      <ul class="list-group list-group-flush">
        <li class="list-group-item">{{ current.Authors }}</li>
        <li class="list-group-item">
          <a target="_blank" :href="getCurrentLink">{{ current.DOILink }}</a>
        </li>
        <li class="list-group-item">{{ current.Type }}</li>
      </ul>
    </div>
  </div>

  <div v-if="this.token && !this.err" class="fixed-bottom action-panel">
    <div class="container pt-2">
      <div class="row mb-2">
        <div class="btn-group btn-group-sm" role="group">
          <input
            class="btn-check"
            id="focus0"
            type="radio"
            value="0 - no"
            v-model="current.FocusTf"
          />
          <label for="focus0" class="btn btn-outline-secondary"
            >Focus - No</label
          >
          <input
            class="btn-check"
            id="focus1"
            type="radio"
            value="1 - bpr"
            v-model="current.FocusTf"
          />
          <label for="focus1" class="btn btn-outline-secondary">BPR</label>
          <input
            class="btn-check"
            id="focus2"
            type="radio"
            value="2 - design time"
            v-model="current.FocusTf"
          />
          <label for="focus2" class="btn btn-outline-secondary"
            >Design Time</label
          >
        </div>
      </div>
      <div class="row mb-2">
        <div class="btn-group btn-group-sm">
          <input
            class="btn-check"
            id="art0"
            type="radio"
            value="0 - no"
            v-model="current.ArtefactTf"
          />
          <label for="art0" class="btn btn-outline-secondary"
            >Artefact- No</label
          >
          <input
            class="btn-check"
            id="art1"
            type="radio"
            value="1 - yes"
            v-model="current.ArtefactTf"
          />
          <label for="art1" class="btn btn-outline-secondary">Yes</label>
          <input
            class="btn-check"
            id="art2"
            type="radio"
            value="2 - yes instantiated"
            v-model="current.ArtefactTf"
          />
          <label for="art2" class="btn btn-outline-secondary"
            >Instantiated</label
          >
        </div>
      </div>
      <div class="row pb-2">
        <div class="btn-group">
          <input
            v-on:change="updateCurrent"
            class="btn-check"
            id="rel0"
            type="radio"
            value="0 - no"
            v-model="current.RelevantTf"
          />
          <label for="rel0" class="btn btn-danger">Relevant - No</label>
          <input
            v-on:change="updateCurrent"
            class="btn-check"
            id="rel1"
            type="radio"
            value="1 - mid"
            v-model="current.RelevantTf"
          />
          <label for="rel1" class="btn btn-secondary">Mid</label>
          <input
            v-on:change="updateCurrent"
            class="btn-check"
            id="rel2"
            type="radio"
            value="2 - yes"
            v-model="current.RelevantTf"
          />
          <label for="rel2" class="btn btn-primary">Yes</label>
        </div>
      </div>
    </div>
  </div>
  <div v-if="this.token" class="fixed-top action-panel">
    <div class="container">
      <div class="row center-text">
        For filter year >= {{ fromYear }}: {{ list.length }} from
        {{ totalRows }}
      </div>
    </div>
  </div>
  <div class="toast-container position-fixed top-50 end-0 p-3">
    <div
      id="err-toast"
      class="toast text-bg-alert"
      role="alert"
      aria-live="assertive"
      aria-atomic="true"
    >
      <div class="toast-body">
        {{ lastState }}
        <div class="mt-2 pt-2 border-top">
          <button @click="reload" type="button" class="btn btn-primary btn-sm">
            Reload
          </button>
          <button
            type="button"
            class="btn btn-secondary btn-sm"
            data-bs-dismiss="toast"
          >
            Close
          </button>
        </div>
      </div>
    </div>
    <div
      class="toast align-items-center text-bg-primary border-0"
      id="toast"
      role="alert"
      aria-live="assertive"
      aria-atomic="true"
    >
      <div class="toast-body">{{ lastState }}</div>
    </div>
  </div>
</template>
