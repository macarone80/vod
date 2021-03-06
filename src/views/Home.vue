<template>
  <v-layout align-center justify-center fill-height>
    <v-flex xs12>
      <v-layout row wrap>
        <v-flex xs12 mb-5>
          <h1
            class="text-xs-center text-uppercase font-weight-black"
            :class="[
              { 'display-2': $vuetify.breakpoint.mdAndUp },
              { 'display-1': $vuetify.breakpoint.smOnly }
            ]"
          >
            <span style="color: #d21414">
              <i class="fas fa-circle logo"></i>
              Watch
            </span> Torrent Movies
          </h1>
        </v-flex>
        <v-flex xs10 offset-xs1>
          <v-text-field
            v-model="query"
            label="Looking for something or you have a torrentID"
            @keydown.enter="search"
            :error-messages="errors"
            :disabled="loading"
            solo
            light
            :loading="loading"
            persistent-hint
            hint="torrentID can be torrent magnet, torrent HTTP/HTTPS url or torrent info hash"
            clearable
            :autofocus="!$vuetify.breakpoint.xsOnly"
          >
            <template v-slot:append>
              <v-btn icon @click="loadDemo" v-if="!query">
                <v-icon color="purple">fas fa-vial</v-icon>
              </v-btn>
              <v-btn icon @click="uploadTorrent" v-if="!query">
                <v-icon color="green">fas fa-upload</v-icon>
              </v-btn>
              <v-btn icon @click="search">
                <v-icon color="teal" v-if="validateQuery.isTorrentId">fas fa-eye</v-icon>
                <v-icon color="blue" v-else>fas fa-search</v-icon>
              </v-btn>
            </template>
          </v-text-field>
        </v-flex>
        <v-flex class="text-xs-center">
          <v-btn color="purple" href="https://www.surveymonkey.com/r/3CSJCQR" target="_blank">
            <v-icon left>fas fa-comments</v-icon>Feedback
          </v-btn>
        </v-flex>
        <v-flex xs10 offset-xs1 class="mt-5">
          <div class="mb-3">Share WATCH ONLINE WITHOUT DOWNLOADING:</div>
          <social-sharing
            :url="hostURL"
            title="Explore, download or watch torrent files online"
            description="WATCH ONLINE WITHOUT DOWNLOADING is a web app to explore, download or watch torrent files online"
            hashtags="live_torrent"
            twitter-user="fadi_davenchy"
            network-tag="a"
            inline-template
          >
            <div>
              <network network="facebook" class="blue--text text--darken-2 ma-3">
                <i class="fab fa-facebook"></i> Facebook
              </network>
              <network network="reddit" class="red--text text--lighten-1 ma-3">
                <i class="fab fa-reddit-alien"></i> Reddit
              </network>
              <network network="twitter" class="blue--text text--lighten-4 ma-3">
                <i class="fab fa-twitter"></i> Twitter
              </network>
              <network network="telegram" class="blue--text text--lighten-2 ma-3">
                <i class="fab fa-telegram"></i> Telegram
              </network>
              <network network="skype" class="blue--text ma-3">
                <i class="fab fa-skype"></i> Skype
              </network>
              <network network="sms" class="yellow--text ma-3">
                <i class="fas fa-sms"></i> SMS
              </network>
              <network network="email" class="orange--text ma-3">
                <i class="fas fa-envelope"></i> Email
              </network>
              <network network="vk" class="blue--text text--darken-3 ma-3">
                <i class="fab fa-vk"></i> VKontakte
              </network>
              <network network="weibo" class="red--text text--darken-1 ma-3">
                <i class="fab fa-weibo"></i> Weibo
              </network>
              <network network="whatsapp" class="green--text text--lighten-2 ma-3">
                <i class="fab fa-fw fa-whatsapp"></i> Whatsapp
              </network>
            </div>
          </social-sharing>
        </v-flex>
      </v-layout>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapActions } from "vuex";
import uploadTorrentFile from "../utils/uploadTorrentFile";

export default {
  name: "home",
  data() {
    return {
      loading: false,
      query: "",
      errors: ""
    };
  },
  methods: {
    ...mapActions(["loadTorrentInfo"]),
    search() {
      this.loading = true;
      this.errors = "";

      const { isEmpty, isTorrentId } = this.validateQuery;

      if (isEmpty) {
        this.errors = "Please enter torrent ID or Search Query";
        this.loading = false;
      } else if (isTorrentId) {
        this.$router.push({
          name: "explorer",
          query: {
            torrentId: this.query
          }
        });
      } else {
        this.$router.push({
          name: "search",
          query: {
            query: this.query
          }
        });
      }
    },
    loadDemo() {
      this.query =
        "magnet:?xt=urn:btih:08ada5a7a6183aae1e09d831df6748d566095a10&dn=Sintel";
    },
    uploadTorrent() {
      uploadTorrentFile.run(infoHash => {
        this.query = infoHash;
        window.sessionStorage["home-query"] = infoHash;
        this.search();
      });
    }
  },
  computed: {
    validateQuery() {
      const query = (this.query || "").trim();

      const isEmpty = query === "";
      const isMagnet = query.match(
        /^magnet:\?xt=urn:[a-z0-9]+:[a-f0-9]{40}(&.+=.+)*$/i
      );
      const isInfoHash = query.match(/^[a-f0-9]{40}$/i);
      const isTorrentFile = query.match(/^https?:\/\/.+\.torrent$/i);

      const isTorrentId = !isEmpty && (isMagnet || isInfoHash || isTorrentFile);

      const isSearchQuery = !isEmpty && !isTorrentId;

      return {
        isTorrentId,
        isMagnet,
        isInfoHash,
        isTorrentFile,
        isSearchQuery,
        isEmpty
      };
    }
  },
  watch: {
    query(n) {
      if (!n) n = "";
      if (n.indexOf("%3A") !== -1 || n.indexOf("%2F") !== -1)
        this.query = decodeURI(
          n
            .split("%3A")
            .join(":")
            .split("%2F")
            .join("/")
        );
      window.sessionStorage["home-query"] = n;
    }
  },
  created() {
    document.title = "WATCH ONLINE WITHOUT DOWNLOADING";
    const { query } = this.$route.query;
    if (query) {
      this.query = query;
      this.search();
    }
  },
  mounted() {
    this.query =
      window.sessionStorage["home-query"] || this.$route.query.query || "";
  }
};
</script>

<style scoped>
.v-messages {
  color: #eee !important;
}

.logo {
  animation: fade-in-out 2s ease-in-out infinite;
}

@keyframes fade-in-out {
  0% {
    opacity: 1;
  }
  10% {
    opacity: 1;
  }

  50% {
    opacity: 0;
  }

  90% {
    opacity: 1;
  }
}
</style>
