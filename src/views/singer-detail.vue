<template>
  <div class="singer-detail">
    <music-list
      :songs="songs"
      :title="title"
      :pic="pic"
      :loading="loading"
    ></music-list>
  </div>
</template>

<script>
import { getSingerDetail } from '@/service/singer'
import { processSongs } from '@/service/song'
import { SINGER_KEY } from '@/js/constant'
import storage from 'good-storage'

import musicList from '@/components/music-list/music-list.vue'

export default {
  components: { musicList },
  name: 'singer-detail',
  props: {
    singer: {
      type: Object,
      default() {
        return {}
      }
    }
  },
  data() {
    return {
      songs: null,
      loading: true
    }
  },
  computed: {
    // 计算准确歌手信息
    computedSinger() {
      let ret = null
      const singer = this.singer
      if (singer) {
        ret = singer
      } else {
        const cacheSinger = storage.session.get(SINGER_KEY)
        if (cacheSinger && cacheSinger.mid === this.$route.params.id) {
          ret = cacheSinger
        }
      }
      return ret
    },
    title() {
      const singer = this.computedSinger
      return singer && singer.name
    },
    pic() {
      const singer = this.computedSinger
      return singer && singer.pic
    }
  },
  async created() {
    // 保护更改网址后自动跳转会歌手页面
    if (!this.computedSinger) {
      const path = this.$route.matched[0].path
      this.$router.push({
        path
      })
      return
    }
    const result = await getSingerDetail(this.computedSinger)
    this.songs = await processSongs(result.songs)
    this.loading = false
  }
}
</script>

<style lang="scss" scoped>
.singer-detail {
  position: fixed;
  z-index: 10;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background: $color-background;
}
</style>
