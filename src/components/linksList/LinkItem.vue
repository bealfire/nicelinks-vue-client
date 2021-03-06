<template>
  <div class="content">
    <div class="meta">
      <span class="item classify"
        @click.stop.prevent="onThemeClick(item.theme)">
        {{ fillThemeName(item.classify, item.theme) }}
      </span>
      <a class="item username" :href="getUserPath(item.createdBy)"
        @click.stop="onStopPropagationClick"
        target="_blank" rel="noreferrer noopener">
        {{ item.createdBy || '' }}
      </a>
      <span >{{ item.created | dateOffset }}</span>
      <a class="tag"
        v-for="(iitem, index) in item.tags" :key="index"
        :href="getTagPath(iitem)"
        @click.stop="onStopPropagationClick"
        target="_blank" rel="noreferrer noopener">
        {{ iitem }}
      </a>
    </div>
    <h3 class="title">
      <a class="title-link" :href="item.urlPath"
        @click.stop="onStopPropagationClick"
        target="_blank" rel="noreferrer noopener">
        {{ item.title }}
      </a>
    </h3>
    <div class="abstract" v-if="isAbstract">
      {{ item.abstract || $util.interceptString(item.desc) }}
    </div>
    <div v-if="!isAbstract">
      <div v-if="isShowKeywords && item.keywords" class="link-keywords">
        <strong>{{$t('keywordStr')}}</strong>
        {{ item.keywords }}
      </div>
      <div class="link-desc"
        v-html="this.obtainLinkDesc(item)">
      </div>
      <hr v-if="item.review" class="segmenting-line">
      <div v-if="item.review" class="link-review">
        <preview-md
          :value="reviewPrefix + item.review">
        </preview-md>
      </div>
    </div>
    <div class="action-list">
      <div class="action-item" @click.stop.prevent="onLikeClick(item)">
        <icon class="icons" :name="item.isLikes ? 'likes-down' : 'likes'"></icon>
        <span class="item-num">{{ item.likes }}</span>
      </div>
      <div class="action-item" @click.stop.prevent="onDislikeClick(item)">
        <icon class="icons" :name="item.isDislikes ? 'dislike-down' : 'dislike'"></icon>
        <span class="item-num">{{ item.dislikes }}</span>
      </div>
    </div>
    <slot name="link-share"></slot>
  </div>
</template>

<script>
import $config from 'config'
import PreviewMd from 'components/markdown/PreviewMd.vue'

export default {
  name: 'LinkItem',

  data () {
    return {
      classifyList: $config.classify,
      themeList: $config.theme,
      tagsList: $config.tags,
      isShowKeywords: true
    }
  },

  props: {
    item: {
      type: [Array, Object],
      default: () => { return {} }
    },
    isLoading: {
      type: Boolean,
      default: false
    },
    isAbstract: {
      type: Boolean,
      default: false
    }
  },

  computed: {
    reviewPrefix () {
      return `**${this.$t('reviewStr')}**`
    }
  },

  watch: {
  },

  components: {
    PreviewMd
  },

  created () {
  },

  mounted () {
  },

  methods: {
    dispatchAction (row, action) {
      if (!this.$isLogin()) {
        this.$switchToLogin()
        return
      }

      let params = {
        'userId': this.userInfo._id,
        '_id': row._id,
        'action': action
      }
      this.$apis.dispatchAction(params).then(result => {
        row[action] = result.count
        let actionIdx = action === 'likes' ? 'isLikes' : 'isDislikes'
        row[actionIdx] = !row[actionIdx]
      }).catch((error) => {
        this.$message.error(`${error}`)
      })
    },

    queryTagsArr (classify, tags) {
      let checkTagsArr = this.tagsList[classify]
      let resultArr = []
      checkTagsArr.map(obj => {
        tags.map(item => {
          if (item === obj.value) {
            resultArr.push({
              'key': obj.key,
              'value': obj.value
            })
          }
        })
      })
      return resultArr
    },

    fillThemeName (classify, theme) {
      let result = ''
      this.themeList[classify].map(item => {
        if (item.value === theme) {
          result = item.key
        }
      })
      return result
    },

    getUserPath (username) {
      let userName = username || this.userInfo.username
      return `/member/${userName}`
    },

    getTagPath (item) {
      return `/tags/${item}`
    },

    obtainLinkDesc (item = {}) {
      return item.desc ? item.desc.split('\n').join('<br>') : ''
    },

    /* -----------------------onClickEvent-----------------------Start */
    onStopPropagationClick (elem) {
    },

    onThemeClick (theme) {
      this.$switchRouteByTheme(theme)
    },

    onLinkClick (item) {
      window.open(item.urlPath, item.title)
    },

    onLikeClick (row) {
      this.dispatchAction(row, 'likes')
    },

    onDislikeClick (row) {
      this.$confirm(this.$t('dislikesTips'), this.$t('warmReminder'), {
        confirmButtonText: this.$t('confirm'),
        cancelButtonText: this.$t('cancel'),
        type: 'warning'
      }).then(() => {
        this.dispatchAction(row, 'dislikes')
      }).catch(() => {
      })
    }
  }
}
</script>

<style media="screen" lang="scss">
@import "./../../assets/scss/variables.scss";
@import './../../assets/scss/mixins.scss';

.content{
  margin: 5px;
  .title{
    margin: 15px 0;
    .title-link{
      font-size: 1.8rem;
      font-weight: 600;
      line-height: 1.2;
      text-decoration: none;
      color: $link-title;
      transition: color 0.3s ease-in;
      &:hover{
        transition: color 0.3s ease-out;
        color: $link-title-hover;
      }
    }
  }
  .abstract{
    width: 100%;
    margin-bottom: 1rem;
    font-size: 1.5rem;
    color: $silver-grey;
    line-height: 1.5;
    letter-spacing: .02rem;
  }

  .link-keywords,
  .link-review{
    margin: 15px 0;
    color: $black-grey;
    line-height: 1.8rem;
    strong{
      font-weight: 700;
      color: $link-title;
    }
  }
  .link-keywords{
    word-break: break-all;
  }
  .link-desc{
    color: $silver-grey;
    margin: 15px auto;
    padding-left: 10px;
    word-break: break-all;
    line-height: 1.8rem;
    font-size: $font-small;
    word-spacing: 3px;
    border-left: 2px solid #000000;
  }
  .segmenting-line{
    border-top: 1px solid #8c8b8b;
    border-bottom: 1px solid #ffffff;
  }
  .meta{
    font-size: 1.314rem;
    color: $black-grey;
    .item{
      cursor: pointer;
      &:after{
        content: "\B7";
        margin: 0 .4em;
      }
    }
    .classify{
      color: $brand;
      font-size: 500;
    }
    .username{
      color: $black-grey;
      &:hover{
        color: $brand;
      }
    }
    .tag{
      color: $black-grey;
      cursor: pointer;
      &:hover{
        color: $brand;
      }
    }
    .tag + .tag{
      &:before{
        margin: 0 .1em;
        content: "/";
        color: $black-grey;
      }
    }
  }
  .action-list{
    display: inline-flex;
    display: -webkit-flex;
    .action-item{
      @include flex-box-center();
      cursor: pointer;
      padding: .3rem .9rem;
      height: 100%;
      text-align: center;
      min-width: 3.6rem;
      border: 1px solid $item-border-color;
      .icons{
        width: 1.6rem;
        height: 1.6rem;
      }
      .icon-green{
        color: $green;
      }
      .item-num{
        color: $black;
        margin-left: .2em;
        font-weight: 700;
      }
    }
    .action-item + .action-item{
      border-left: none;
    }
  }
}
</style>
