<template>
  <aside class="aside-list">
    <ads-position :adverts-list="advertsList"></ads-position>
    <el-collapse v-model="activeNames" @change="handleChange">
      <el-collapse-item :title="$t('awesomeSentence')" name="awesomeSentence">
        <awesome-sentence :sentence="sentence" />
      </el-collapse-item>
      <el-collapse-item :title="$t('friendshipLinks')" name="friendshipLinks">
        <friendship-links />
      </el-collapse-item>
      <el-collapse-item :title="$t('rewardme')" name="rewardme">
        <reward-me />
      </el-collapse-item>
    </el-collapse>
  </aside>
</template>

<script>
import AdsPosition from './AdsPosition'
import FriendshipLinks from './FriendshipLinks'
import RewardMe from './RewardMe'
import AwesomeSentence from './AwesomeSentence'

export default {
  name: 'Sidebar',

  data () {
    return {
      activeNames: ['awesomeSentence', 'friendshipLinks'],
      advertsList: [],
      sentence: {}
    }
  },

  watch: {
  },

  created () {
    this.$apis.getSysConf().then(result => {
      this.advertsList = result.advertsList.sort((a, b) => {
        return a.sort - b.sort
      })

      this.sentence = result.sentence
    }).catch((error) => {
      this.$message.error(`${error}`)
    })
  },

  components: {
    AdsPosition,
    FriendshipLinks,
    RewardMe,
    AwesomeSentence
  },

  mounted () {
  },

  methods: {
    handleChange () {}
  }
}
</script>

<style type="text/css" lang="scss">
@import "./../../assets/scss/variables.scss";

.aside-list{
  background-color: $white;
  display: inline-block;
  max-width: 360px;
  width: 32%;
  float: right;
  .aside-list-li{
    list-style-type: bengali;
  }
  .aside-list-a{
    color: $common-link;
    font-size: $font-medium;
  }
  .el-collapse-item__header{
    text-align: left;
    font-size: $font-medium;
  }
}
</style>
