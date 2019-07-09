<template>
  <q-page padding>
    <q-scroll-area style="height: calc(100vh - 257px);">
      <div class="q-pa-md row items-start justify-center q-gutter-md">
        <q-card
          class="my-card"
          v-for="video in videoList"
          :key="video.id"
          @click="gotoPlayer(video)"
        >
          <q-img
            :src="video.pic"
            spinner-color="red"
            style="height: 200px;width: 290px"
          >
            <div class="absolute-bottom text-subtitle1 text-center q-pa-xs">{{video.name}}</div>
            <template v-slot:error>
              <div class="absolute-full flex flex-center bg-negative text-white">
                <span>Cannot load image</span>
              </div>
            </template>
          </q-img>
          <q-card-section>
            <div class="text-h6">{{video.type}}</div>
            <div class="text-subtitle2">{{ video.last }}</div>
          </q-card-section>
        </q-card>
      </div>
      <div class="q-pa-lg flex flex-center">
        <q-pagination
          v-model="pagination.page"
          :max="pagination.total"
          :input="true"
        ></q-pagination>
      </div>
      <q-inner-loading :showing="loading">
        <q-spinner-gears
          size="50px"
          color="primary"
        />
      </q-inner-loading>
    </q-scroll-area>
  </q-page>
</template>

<script>
import util from 'util';
import { parseString } from 'xml2js';
import _toInteger from 'lodash/toInteger';
import { mapGetters, mapMutations, mapState } from 'vuex';

const parseStringSync = util.promisify(parseString);
export default {
  name: 'VideoList',
  data() {
    return {
      loading: true,
      videoList: [],
      pagination: {
        page: 1,
        total: 0,
      },
    };
  },
  mounted() {
    this.getVideoList(1);
  },
  watch: {
    // eslint-disable-next-line func-names
    'pagination.page': function (value) {
      this.getVideoList(value);
    },
    currentSite() {
      this.getVideoList(1);
    },
    currentClass() {
      this.getVideoList(1);
    },
  },
  methods: {
    ...mapMutations(['setCurrentVideo']),
    getVideoList(page) {
      this.loading = true;
      const params = {
        ac: 'videolist',
        pg: page,
      };
      if (this.currentClass !== 'all') {
        params.t = this.currentClass;
      }
      const uri = this.https
        ? this.currentSite.httpsApi
        : this.currentSite.httpApi;
      this.$axios(uri, {
        params,
      })
        .then(res => parseStringSync(res.data, { explicitArray: false }))
        .then((data) => {
          this.videoList = data.rss.list.video;
          this.pagination.total = _toInteger(data.rss.list.$.recordcount);
        })
        .catch(console.error)
        .finally(() => {
          this.loading = false;
        });
    },
    gotoPlayer(video) {
      this.setCurrentVideo(video);
      this.$router.push('/video');
    },
  },
  computed: {
    ...mapGetters(['currentSite']),
    ...mapState({
      currentClass: state => state.site.currentClass,
      https: state => state.app.https,
    }),
  },
};
</script>

<style>
.my-card {
  width: 100%;
  max-width: 290px;
}
</style>