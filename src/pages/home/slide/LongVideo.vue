<script setup>

import {computed, onMounted, onUnmounted, reactive, ref, watch} from "vue";
import {uniqueId} from "lodash-es";
import api from "@/api";
import {useStore} from "vuex";
import {_checkImgUrl, _duration, _formatNumber} from "@/utils";

const store = useStore()
const loading = computed(() => store.state.loading)

const props = defineProps({
  active: Boolean
})

const p = {
  onShowComments() {
    console.log('onShowComments')
  }
}

const listRef = ref(null)
const state = reactive({
  index: 0,
  list: [],
  uniqueId: uniqueId('uniqueId_'),
  totalSize: 0,
  pageSize: 10,
  pageNo: 0,
})

function loadMore() {
  if (!loading.value) {
    state.pageNo++
    getData()
  }
}

async function getData(refresh = false) {
  if (loading.value) return
  store.commit('setLoading', true)
  let res = await api.videos.recommended({pageNo: refresh ? 0 : state.pageNo, pageSize: state.pageSize})
  console.log('getSlide4Data-', 'refresh', refresh, res)
  store.commit('setLoading', false)
  if (res.code === 200) {
    state.totalSize = res.data.total
    if (refresh) {
      state.list = []
    }
    state.list = state.list.concat(res.data.list)
  } else {
    state.pageNo--
  }
}

watch(() => props.active, n => {
  if (!state.list.length && n) {
    store.commit('setLoading', false)
    getData()
  }
})

onMounted(() => {
})
onUnmounted(() => {
})
</script>


<template>
  <div class="page">
    <div class="item"
         :class="[
             i % 5 === 0 && 'big',
              i % 5 === 0 ? '' : (i % 2 === 1 && 'l'),
              i % 5 === 0 ? '' : (i % 2 === 0 && 'r'),
         ]"
         v-for="(item,i) in state.list">
      <video
          controls
          :poster="_checkImgUrl(item.video.cover.url_list[0])"
          :src="item.video.play_addr.url_list[0]"
      ></video>
      <img v-lazy="_checkImgUrl(item.video.cover.url_list[0])" alt="" class="poster">
      <div class="duration">{{ _duration(item.duration / 1000) }}</div>
      <div class="title">
        {{ item.desc }}
      </div>
      <div class="bottom">
        <div class="l">
          <img v-lazy="item.author.avatar_168x168.url_list[0]" alt="" class="avatar">
          <div class="name">{{ item.author.nickname }}</div>
        </div>
        <div class="r">
          <Icon icon="icon-park-outline:like"/>
          <div class="num">{{ _formatNumber(item.statistics.digg_count) }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="less">
@import "@/assets/less/index";

.page {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  row-gap: 15rem;
  height: calc(100vh - @header-height - @footer-height);
  margin-top: @header-height;
  overflow: auto;
  box-sizing: border-box;

  .item {
    margin: 0 10rem;
    display: flex;
    flex-direction: column;
    gap: 8rem;
    position: relative;

    .poster {
      border-radius: 12rem;
      width: 100%;
      height: 140rem;
      object-fit: cover;
    }

    video {
      display: none;
      height: 220rem;
      object-fit: cover;
    }

    .title {
      height: 36rem;
      color: white;
      font-size: 14rem;
      overflow: hidden;
      text-overflow: ellipsis;
      display: -webkit-box; //作为弹性伸缩盒子模型显示。
      -webkit-box-orient: vertical; //设置伸缩盒子的子元素排列方式--从上到下垂直排列
      -webkit-line-clamp: 2; //显示的行
    }

    .f {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 5rem;
    }

    .duration {
      color: white;
      position: absolute;
      bottom: 80rem;
      right: 10rem;
      font-size: 13rem;
    }

    .bottom {
      color: gray;
      .f;
      font-size: 13rem;

      .l {

        .f;

        .avatar {
          @w: 20rem;
          width: @w;
          height: @w;
          object-fit: cover;
          border-radius: 50%;
        }
      }

      .r {
        .f;

        svg {
          font-size: 16rem;
        }

      }
    }

    &.big {
      grid-column-start: 1;
      grid-column-end: 3;
      margin: 0;

      .duration {
        display: none;
      }

      .poster {
        display: none;
      }

      video {
        display: block;
      }

      .title {
        height: unset;
        -webkit-line-clamp: 1;
      }

      .title, .bottom {
        padding: 0 10rem;
      }
    }

    &.l {
      margin-right: 5rem;
    }

    &.r {
      margin-left: 5rem;
    }
  }
}
</style>