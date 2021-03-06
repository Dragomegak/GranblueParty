<template>
  <div>
    <h1>Spark Maker</h1>

    <div class="flex flex-row flex-wrap items-center space-x-2 mb-4">
      <button class="btn" :class="show_help ? 'btn-blue' : 'btn-white'" @click="show_help = ! show_help">
        <fa-icon :icon="['fas', 'info-circle']" class="text-xl"></fa-icon> Usage
      </button>
      <checkbox v-model="include_sr">Include Rs and SRs</checkbox>
      <button class="btn btn-blue" @click="screenSpark()">
        <fa-icon :icon="['fas', 'save']" class="text-xl"></fa-icon> Save as PNG
      </button>
      <button v-if="isUserLogged" class="btn btn-blue" @click="saveToCollection()">
        <fa-icon :icon="['fas', 'save']" class="text-xl"></fa-icon> Add Units to my Collection
      </button>
      <button class="btn btn-red" @click="newSpark()">
        <fa-icon :icon="['fas', 'trash']" class="text-xl"></fa-icon> New Spark
      </button>
    </div>

    <div class="bg-secondary rounded p-4 mb-2" v-if="show_help">
      <h2>Search</h2>
      <p class="pb-4">
        You can search units by English, Japanese or Weapon name.<br>
        You need to type at least 2 characters to display the result of your query.
      </p>
      <h2>Click</h2>
      <p class="pb-4">
        Left-click on a unit in the search section to add it to your spark.<br>
        Right-click on a unit to add it as a Gold Moon.<br>
        To remove a unit from your spark, left-click on its image in the spark section.
      </p>
      <h2>Add to my Collection</h2>
      <p class="pb-4">
        If you are logged in with your account, you can add the result of your Spark to your Collection.<br>
        Characters will be added with a base amount of stars (no blue stars unlocked).<br>
        New summons will be added with no stars, and existing ones will gain one star (if possible).<br>
        Be careful not to add the same Spark multiple times, or you might add extra unwanted stars to your summons.
      </p>
    </div>

    <div v-if="loading === true">
      Loading...
    </div>
    <div v-else class="flex flex-row flex-wrap">
      <!-- Search -->
      <div class="flex flex-col md:w-1/4 pb-4">
        <div class="mb-4 flex flex-row flex-wrap space-x-2">
          <input class="input" type="text" v-model="search_text" placeholder="Search" ref="searchfield" autofocus>
          <button class="btn btn-blue" @click="clearText()">
            <fa-icon :icon="['fas', 'times']" class="text-xl"></fa-icon> Clear
          </button>
        </div>

        <span class="flex flex-col overflow-y-auto mr-2" :class="screenshot ? '' : 'h-128'">
          <h2 v-if="searchCharacters.length > 0">Characters</h2>
          <span class="flex flex-row flex-wrap">
            <spark-unit
              v-for="chara in searchCharacters"
              :key="chara.id"
              :unit="chara"
              @left-click-unit="selectCharacter(chara)"
              @right-click-unit="selectGM(chara)"
            ></spark-unit>
          </span>
          <h2 v-if="searchSummons.length > 0">Summons</h2>
          <span class="flex flex-row flex-wrap">
            <spark-unit
              v-for="summon in searchSummons"
              :key="summon.id"
              :unit="summon"
              @left-click-unit="selectSummon(summon)"
            ></spark-unit>
          </span>
        </span>        
      </div>

      <!-- Display -->
      <div id="spark" class="flex flex-col bg-primary md:w-3/4">
        <div class="flex flex-row flex-wrap">
          <div class="flex pb-2" :class="include_sr ? 'md:w-1/4' : 'md:w-1/3'">
            <div class="flex flex-col items-center bg-secondary rounded py-4 mr-2 px-2 w-full">
              <img class="pb-2" src="/img/item/crystal.jpg" width="65" height="65">
              <div class="flex flex-row flex-wrap">
                <spark-unit
                  v-for="(chara, index) in drawn_characters"
                  :key="index"
                  :unit="chara"
                  :width="126" :height="72"
                  @left-click-unit="drawn_characters.splice(index, 1)"
                ></spark-unit>
              </div>
            </div>
          </div>
          <div class="flex pb-2" :class="include_sr ? 'md:w-1/4' : 'md:w-1/3'">
            <div class="flex flex-col items-center bg-secondary rounded py-4 mr-2 px-2 w-full">
              <img class="pb-2" src="/img/item/goldmoon.jpg" width="65" height="65">
              <div class="flex flex-row flex-wrap">
                <spark-unit
                  v-for="(chara, index) in drawn_GM"
                  :key="index"
                  :unit="chara"
                  :width="126" :height="72"
                  @left-click-unit="drawn_GM.splice(index, 1)"
                ></spark-unit>
              </div>
            </div>
          </div>
          <div class="flex pb-2" :class="include_sr ? 'md:w-1/4' : 'md:w-1/3'">
            <div class="flex flex-col items-center bg-secondary rounded py-4 px-2 w-full" :class="include_sr ? 'mr-2' : ''">
              <img class="pb-2" src="/img/item/sunlightstone.jpg" width="65" height="65">
              <div class="flex flex-row flex-wrap">
                <spark-unit
                  v-for="(summon, index) in drawn_summons"
                  :key="index"
                  :unit="summon"
                  :width="126" :height="72"
                  @left-click-unit="drawn_summons.splice(index, 1)"
                ></spark-unit>
              </div>
            </div>
          </div>
          <div class="flex pb-2" :class="include_sr ? 'md:w-1/4' : 'md:w-1/3'" v-if="include_sr">
            <div class="flex flex-col items-center bg-secondary rounded py-4 px-2 w-full">
              <img class="pb-2" src="/img/item/silvermoon.jpg" width="65" height="65">
              <div class="flex flex-row flex-wrap">
                <spark-unit
                  v-for="(chara, index) in drawn_SRs"
                  :key="index"
                  :unit="chara"
                  :width="126" :height="72"
                  @left-click-unit="drawn_SRs.splice(index, 1)"
                ></spark-unit>
              </div>
            </div>
          </div>          
        </div>

        <div class="flex flex-row justify-between">
          <span>
            <label v-if="! screenshot">Draws <input class="input input-sm" type="number" min="1" style="width: 7ch;" v-model="draws"></label>
            <span class="ml-2">SSR ratio: {{ ssrRatio }}%</span>
          </span>
          <span v-if="screenshot" class="pr-2">https://www.granblue.party/spark</span>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'

import domtoimage from '@/js/libs/dom-to-image-more.min.js'
import Utils from '@/js/utils.js'
import collectionModule from '@/store/modules/collection-tracker'
import sparkModule from '@/store/modules/spark'

import Checkbox from '@/components/common/Checkbox.vue'
import SparkUnit from '@/components/SparkUnit.vue'

const lsMgt = new Utils.LocalStorageMgt('Spark');

export default {
  components: {
    Checkbox,
    SparkUnit
  },
  head: {
    title: 'Granblue.Party - Spark Maker',
    desc: 'Granblue Fantasy Spark Maker',
    image: 'https://www.granblue.party/img/preview_spark.png',
    keywords: 'spark, gacha, draws, characters, summons, weapons'
  },
  data() {
    return {
      search_text: '',
      include_sr: false,
      loading: true,
      show_help: false,
      drawn_characters: [],
      drawn_SRs: [],
      drawn_GM: [],
      drawn_summons: [],
      draws: 300,
      screenshot: false
    }
  },
  methods: {
    loadData() {
      let promises = [
        this.$store.dispatch('spark/fetchCharacters'),
        this.$store.dispatch('spark/fetchSummons')
      ];
      if (this.isUserLogged) {
        promises.push(this.$store.dispatch('collection/fetchCollection', this.$store.getters.getUserId));
      }

      return Promise.all(promises);
    },
    newSpark() {
      this.drawn_characters = [];
      this.drawn_SRs = [];
      this.drawn_GM = [];
      this.drawn_summons = [];
    },
    clearText() {
      this.search_text = '';
      this.$nextTick().then(() => {
        this.$refs.searchfield.focus();
      });
    },
    selectCharacter(element) {
      if (element.r == 2) {
        // SSR
        if (this.drawn_characters.some(c => c.id == element.id) || this.drawn_GM.some(c => c.id == element.id)) {
          this.drawn_GM.push(element);
        }
        else if (this.ownedCharactersMap[element.id] && this.ownedCharactersMap[element.id].owned === true) {
          this.drawn_GM.push(element);
        }
        else {
          this.drawn_characters.push(element);
        }
      }
      else {
        // SR
        this.drawn_SRs.push(element);
      }
      this.clearText();
    },
    selectGM(element) {
      if (element.r == 2) {
        this.drawn_GM.push(element);
      }
      else {
        this.drawn_SRs.push(element);
      }
      this.clearText();
    },
    selectSummon(element) {
      this.drawn_summons.push(element);
      this.clearText();
    },
    screenSpark() {
      this.screenshot = true;
      let node = document.getElementById('spark');
      domtoimage.toPng(node)
        .then((dataUrl) => {
          this.screenshot = false;
          var link = document.createElement('a');
          link.download = 'spark.png';
          link.href = dataUrl;
          link.click();
        });
    },
    saveToCollection() {
      if (this.ownedCharactersMap !== null) {
        const aw = 0;
        const postData = {c: [], s: []};

        this.drawn_characters.forEach(chara => {
          const chara_owned = this.ownedCharactersMap[chara.id];
          if (chara_owned && chara_owned.owned !== true) {
            postData.c.push([chara.id, chara_owned.sb, true, aw]);
          }
        });
        this.drawn_SRs.forEach(chara => {
          const chara_owned = this.ownedCharactersMap[chara.id];
          if (chara_owned && chara_owned.owned !== true) {
            postData.c.push([chara.id, chara_owned.sb, true, aw]);
          }
        });

        this.drawn_summons.forEach(summ => {
          const summon_owned = this.ownedSummonsMap[summ.id];
          if (summon_owned) {
            const index = postData.s.findIndex(s => s[0] === summ.id);
            if (index >= 0) {
              if (postData.s[index][1] < summon_owned.sm) {
                postData.s[index][1]++;
              }
            }
            else {
              let stars = summon_owned.sc;
              if (summon_owned.owned == null || summon_owned.owned === false) {
                stars = 0;
              }
              else if (stars < summon_owned.sm) {
                stars++;
              }

              postData.s.push([summ.id, stars, true]);
            }
          }
        });

        if (postData.c.length > 0 || postData.s.length > 0) {
          this.axios.post('/tracker/save', postData)
            .then(response => this.$store.dispatch('addMessage', {message: 'Units successfully added to the Collection'}))
            .catch(error => this.$store.dispatch('addAxiosErrorMessage', error));
        }
        else {
          this.$store.dispatch('addMessage', {message: 'No units to add.'});
        }
      }
    }
  },
  computed: {
    ...mapState('spark', [
      'characters',
      'summons'
    ]),
    ...mapState('collection', {
      collection_charas: 'characters',
      collection_summons: 'summons'
    }),
    isUserLogged() {
      return this.$store.getters.getUserId !== null;
    },
    searchCharacters() {
      if (this.search_text.length < 2) {
        return {};
      }
      const text_lower = this.search_text.toLowerCase();

      return this.characters.filter(c => 
        (this.include_sr || c.r == 2) && (
          c.n.toLowerCase().includes(text_lower) ||
          c.nj.includes(this.search_text) ||
          c.w.toLowerCase().includes(text_lower)
        )
      )
    },
    searchSummons() {
      if (this.search_text.length < 2) {
        return {};
      }
      const text_lower = this.search_text.toLowerCase();

      return this.summons.filter(s =>
        s.n.toLowerCase().includes(text_lower) ||
        s.nj.includes(this.search_text)
      );
    },
    ssrRatio() {
      if (this.draws < 1) {
        return 0;
      }
      return ((this.drawn_characters.length + this.drawn_GM.length + this.drawn_summons.length) / this.draws * 100).toFixed(2);
    },
    ownedCharactersMap() {
      let result = {};
      this.collection_charas.forEach(e => e.reduce((map, obj) => {
        map[obj.id] = obj;
        return map;
      }, result));
      return result;
    },
    ownedSummonsMap() {
      let result = {};
      this.collection_summons.forEach(e => e.reduce((map, obj) => {
        map[obj.id] = obj;
        return map;
      }, result));
      return result;
    }
  },
  serverPrefetch() {
    return this.loadData();
  },
  async mounted() {
    await this.loadData()
      .then(_ => {
        this.loading = false;
        this.$nextTick().then(() => {
          this.$refs.searchfield.focus();
        });
      });
    
    lsMgt.getValue(this, 'include_sr');
    lsMgt.getValue(this, 'drawn_characters');
    lsMgt.getValue(this, 'drawn_SRs');
    lsMgt.getValue(this, 'drawn_GM');
    lsMgt.getValue(this, 'drawn_summons');
  },
  watch: {
    '$store.getters.getUserId'(id) {
      if (id === null) {
        this.$store.commit('collection/resetCollection');
      }

      this.loading = true;
      this.loadData()
        .then(_ => this.loading = false);
    },
    include_sr() {
      lsMgt.setValue('include_sr', this);
    },
    drawn_characters() {
      lsMgt.setValue('drawn_characters', this);
    },
    drawn_SRs() {
      lsMgt.setValue('drawn_SRs', this);
    },
    drawn_GM() {
      lsMgt.setValue('drawn_GM', this);
    },
    drawn_summons() {
      lsMgt.setValue('drawn_summons', this);
    },
  },
  beforeCreate() {
    const preserve_state_spark = !! this.$store.state.spark;
    this.$store.registerModule('spark', sparkModule, { preserveState: preserve_state_spark });

    const preserve_state_coll = !! this.$store.state.collection;
    this.$store.registerModule('collection', collectionModule, { preserveState: preserve_state_coll });
  },
  beforeRouteLeave(to, from, next) {
    this.$store.unregisterModule('collection');
    this.$store.unregisterModule('spark');
    next();
  },
}
</script>