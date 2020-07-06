<template>
  <div>
    <div class="comment-floor">
      <!-- 组件在这个位置自己调用自己 -->
      <div class="floor-top">
        <div class="user">
          <em>{{data.account.nickname}}</em>
          <span>{{data.created_at | dateForm}}</span>
          <span>{{data.level}}</span>
        </div>
        <!-- 点击回复按钮时候触发父组件传递过来的回复函数 -->
      </div>
      <commentFloor v-if="data.parent" :data="data.parent" @reply="handleReply" />
      <div class="content">{{data.content}}</div>
      <div class="picture">
        <img src alt />
      </div>
      <span class="reply" @click="handleReply(data)">回复</span>
    </div>
  </div>
</template>

<script>
import moment from "moment";
import { log } from "util";
export default {
  filters: {
    dateForm(time) {
      return moment(new Date(time)).format("YYYY-MM-DD HH:MM");
    }
  },
  name: "commentFloor",
  data() {
    return {
      isShow: false,
      moment
    };
  },
  props: ["data", "count"],
  methods: {
    // 组件内部的回复的事件
    handleReply(data) {
      // 触发父组件传递过来的reply函数
      this.$emit("reply", data);
    }
  }
};
</script>


<style scoped lang="less">
.comment-floor {
  margin-bottom: 20px;
  border: 1px #ddd solid;
  padding: 2px;
  font-size: 13px;
  .reply {
    margin-left: 600px;
  }

  .floor-top {
    display: flex;
    justify-content: space-between;
    padding: 20px 5px;

    span {
      font-size: 12px;
      color: #999;
      margin-right: 5px;
    }
    em {
      margin-right: 5px;
    }
  }

  .content {
    padding: 20px 5px;
    padding-top: 0;
  }
}
</style>