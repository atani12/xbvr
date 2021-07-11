<template>
  <div class="content">
    <h3 class="title">{{$t('JAVR scraper')}}</h3>
    <div class="columns is-multiline">
      <div class="column is-multiline is-one-third">
        <div class="card">
          <div class="card-content content">
            <h5 class="title">R18</h5>
            <b-field grouped>
              <b-input v-model="javrQuery" placeholder="URL or ID (XXXX-001)" type="search"></b-input>
              <b-button class="button is-primary" v-on:click="scrapeJAVR()">{{$t('Go')}}</b-button>
            </b-field>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import ky from 'ky'
import VueLoadImage from 'vue-load-image'
import { formatDistanceToNow, parseISO } from 'date-fns'

export default {
  name: 'OptionsSites',
  components: { VueLoadImage },
  data () {
    return {
      javrQuery: ''
    }
  },
  mounted () {
    this.$store.dispatch('optionsSites/load')
  },
  methods: {
    getImageURL (u) {
      if (u.startsWith('http')) {
        return '/img/128x/' + u.replace('://', ':/')
      } else {
        return u
      }
    },
    taskScrape (site) {
      ky.get(`/api/task/scrape?site=${site}`)
    },
    forceSiteUpdate (site) {
      ky.post('/api/options/scraper/force-site-update', {
        json: { site_name: site }
      })
      this.$buefy.toast.open(`Scenes from ${site} will be updated on next scrape`)
    },
    deleteScenes (site) {
      this.$buefy.dialog.confirm({
        title: this.$t('Delete scraped scenes'),
        message: `You're about to delete scraped scenes for <strong>${site}</strong>. Previously matched files will return to unmatched state.`,
        type: 'is-danger',
        hasIcon: true,
        onConfirm: function () {
          ky.post('/api/options/scraper/delete-scenes', {
            json: { site_name: site }
          })
        }
      })
    },
    scrapeJAVR () {
      ky.post('/api/task/scrape-javr', { json: { q: this.javrQuery } })
    },
    parseISO,
    formatDistanceToNow
  },
  computed: {
    items () {
      return this.$store.state.optionsSites.items
    },
    runningScrapers () {
      this.$store.dispatch('optionsSites/load')
      return this.$store.state.messages.runningScrapers
    }
  }
}
</script>

<style scoped>
  .running {
    opacity: 0.6;
    pointer-events: none;
  }

  .card {
    overflow: visible;
    height: 100%;
  }

  .card-content {
    padding-top: 1em;
    padding-left: 1em;
  }

  .avatar {
    margin-right: 1em;
  }

  p {
    margin-bottom: 0.5em !important;
  }

  h5 {
    margin-bottom: 0.25em !important;
  }

  .switch {
    position: absolute;
    bottom: 0.25em;
    right: 0em;
  }

  .invisible {
    display: none;
  }

  .menu {
    position: absolute;
    top: 0.75em;
    right: 0.5em;
  }
</style>
