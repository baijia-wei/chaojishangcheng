<template>
  <div class="contianer">
    <el-col :span="17">
      <div class="Left_layout">
        <span>旅游攻略/攻略详情</span>
        <h6>{{detailData.title}}</h6>
        <!-- 横线 -->
        <div class="box"></div>
        <!--攻略 阅读 -->
        <div class="ntroduction_reading">
          <div>
            <span>攻略{{detailData.city.created_at}}</span>
            <span>阅读：{{detailData.watch}}</span>
          </div>
        </div>
        <!-- 标题 -->

        <!-- 内容 -->
        <div class="content1" v-html="detailData.content"></div>
        <!-- 评论 -->
        <div class="comment">
          <div class="comment_in">
            <div>
              <img src="" alt />
              <img src="" alt />
            </div>
            <span>评论(100)</span>
            <span>分享(100)</span>
          </div>
          <!-- 发表 -->
          <div class="form">
            <span>评论</span>
            <div class="glyuan">
              <el-tag
                type="info"
                closable
                class="replyTag"
                v-if="isTagShow"
                @close="closeTag"
              >回复：@{{nickname}}</el-tag>
            </div>
            <el-input
              type="textarea"
              :autosize="{ minRows: 2, maxRows: 6}"
              placeholder="说点什么吧"
              v-model="textarea2"
            ></el-input>
            <div class="box3">
              <el-button type="primary" @click="submitComment">发布按钮</el-button>
            </div>
            <!-- 图片上传 -->
            <div class="box4">
              <el-upload
                :action="`${$axios.defaults.baseURL}/upload`"
                list-type="picture-card"
                name="files"
                :limit="3"
                :on-preview="handlePictureCardPreview"
                :on-remove="handleRemove"
                :file-list="fileList"
                :on-success="fileUploadSuccess"
              >
                <i class="el-icon-plus"></i>
              </el-upload>
            </div>
            <el-dialog :visible.sync="dialogVisible">
              <img width="100%" :src="dialogImageUrl" alt />
            </el-dialog>
          </div>
          <!--  评论-->
          <div class="comment_frame">
            <div class="comment1" v-for="(item, index) in comment" :key="index">
              <div class="comment-top">
                <div class="user">
                  <img :src="$axios.defaults.baseURL+item.account.defaultAvatar" />
                  <div class="user-info">
                    <p>{{item.account.nickname}}</p>
                    <!-- moment().fromNow 就是显示距离到当前的时间 -->
                    <span>{{item.created_at | dateForm}}</span>
                    <span>{{item.level}}</span>
                  </div>
                </div>
              </div>
              <CommentFloor v-if="item.parent" :data="item.parent" @reply="handleReply" />
              <div class="content">{{item.content}}</div>
              <div class="picture" v-for="(item1, index1) in item.pics" :key="index1">
                <img :src="$axios.defaults.baseURL+item1.url" alt />
              </div>
              <span class="reply" @click="handleReply(item)">回复</span>
              <!-- 递归组件 -->
            </div>
          </div>
        </div>
        <!-- 分页 -->
        <div class="paging">
          <!-- 分页组件 -->
          <!-- size-change: 分页条数切换时候触发的事件 -->
          <!-- current-change: 页数切换触发的事件 -->
          <!-- current-page: 当前页数 -->
          <!-- page-size：当前显示的条数 -->
          <!-- total: 总条数 -->
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="pageIndex"
            :page-sizes="[2, 4, 6, 8]"
            :page-size="pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="+total"
          ></el-pagination>
        </div>
      </div>
    </el-col>

    <!-- 右边相关攻略 -->
    <el-col :span="7">
      <div class="related_strategies">
        <span>相关攻略</span>
      </div>

      <div
        class="related_biaoti"
        :to="`/post/detail?id=${item.id}`"
        v-for="(item, index) in xiangguangl"
        :key="index"
      >
        <div class="related_r">
          <img :src="$axios.defaults.baseURL+item.images[0]" alt />
        </div>
        <div class="related_l">
          <span>{{item.title}}</span>
          <p>{{item.created_at | dateForm}} 阅读 -{{item.watch }}</p>
        </div>
      </div>
      <div class="related_biaoti">
        <div class="related_r"></div>
        <div class="related_l">
          <span>小黑屋一日</span>
          <p>2020-04-13 2:02 阅读 60</p>
        </div>
      </div>
    </el-col>
  </div>
</template>

<script>
moment.locale("zh-CN");
import moment from "moment";
import CommentFloor from "@/components/post/CommentFloor.vue";
import { log } from "util";
export default {
  components: {
    CommentFloor
  },
  filters: {
    dateForm(time) {
      return moment(new Date(time)).format("YYYY-MM-DD HH:MM");
    }
  },
  data() {
    return {
      isTagShow: false, //切换
      nickname: "", //回复的名字
      follow: false, //回复id
      iSfo: false,
      // 文章
      detailData: {
        city: {}
      },
      // 评论数据
      comment: [],
      fileList: [],
      pics: [], // 图片上传列表
      // 相关攻略
      xiangguangl: [],
      // 发布框的数据
      textarea2: "",
      dialogImageUrl: "",
      dialogVisible: false,
      // 当前页数
      pageIndex: 1,
      // 当前条数
      pageSize: 5,
      // 总跳数
      total: "",
      // 回复评论的对象
      reply: {} //,\
    };
  },
  mounted() {
    this.getCommentData();
    // 页面内容请求
    this.$axios({
      url: "/posts",
      params: {
        id: this.$route.query.id
      }
    }).then(res => {
      // console.log(res);

      this.detailData = res.data.data[0];
    });
    // 先关攻略
    this.$axios({
      url: "/posts/recommend"
    }).then(res => {
      this.xiangguangl = res.data.data;
    });
  },
  methods: {
    // 评论请求
    getCommentData() {
      this.$axios({
        url: "/posts/comments",
        params: {
          post: this.$route.query.id,
          _limit: this.pageSize,
          _start: this.pageIndex - 1
        }
      }).then(res => {
        const { data, total } = res.data;
        console.log(data);

        this.total = total;
        this.comment = data;
      });
    },
    // 图片移除
    handleRemove(file, fileList) {
      this.fileList = fileList;
    },
    // 图片上传是回调
    fileUploadSuccess(response, file, fileList) {
      this.pics = [...this.pics, ...response];
    },
    // 图片上预览
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    // 分页组件
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
      this.pageSize = val;
      this.getCommentData();
    },
    // 分页组件
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.pageIndex = val;
      this.getCommentData();
    },

    // 回复..............................................................

    handleReply(data) {
      this.follow = data.id;

      this.nickname = data.account.nickname;
      console.log(this.nickname);
      this.isTagShow = true;

      // 点击回复跳转到
      const cmt = this.$el.querySelector(".form");
      this.$nextTick(() => {
        document.querySelector("html").scrollTop = cmt.offsetTop;
      });
    },

    // 点击回复拿到当前点击对对象信息
    closeTag() {
      this.isTagShow = false;
      this.follow = false;
    },

    // 失去焦点输入框
    handleshiqu() {
      this.iSfo = false;
    },
    // 获得焦点输入框
    handlehuodei() {
      this.iSfo = true;
    },

    submitComment() {
      if (!this.textarea2 && !this.pics.length) {
        this.$message.error("输入内容不能为空");
        return;
      }
      const data = {
        post: +this.$route.query.id,
        content: this.textarea2,
        pics: this.pics
      };
      if (this.follow) {
        data.follow = this.follow;
      }

      this.$axios({
        url: "/comments",
        method: "POST",
        data,

        headers: {
          Authorization: "Bearer " + this.$store.state.user.userInfo.token
        }
      }).then(res => {
        this.pics = [];
        this.textarea2 = [];
        this.getCommentData();
        this.fileList = [];
        this.closeTag();
      });
    }
  }
};
</script>

<style scoped lang="less">
.contianer {
  height: 4000px;
  width: 1000px;
  margin: 0 auto;
  .related_strategies {
    height: 40px;
    margin-top: 25px;
    border-bottom: 1px solid rgb(179, 179, 179);
    margin-left: 30px;
    span {
      display: block;
      margin-bottom: 10px;
    }
  }
  .related_biaoti {
    overflow: hidden;
    height: 100px;
    border-bottom: 1px solid rgb(179, 179, 179);
    margin-left: 30px;
    .related_r {
      float: left;

      img {
        width: 100px;
        height: 80px;
      }
    }
    .related_l {
      span {
        margin-top: 5px;
        display: block;
      }
      p {
        display: block;
        font-size: 14px;
        color: rgb(104, 104, 104);
        margin-top: 50px;
      }
    }

    span {
      // float:right;
      display: block;
      margin-bottom: 10px;
    }
    p {
      margin-top: 20px;
      margin-bottom: 40px;
    }
  }
  .Left_layout {
    // 标题
    margin-top: 20px;
    // ********************
    .content1 {
      /deep/p {
        text-indent: 2em;
        line-height: 25px;
        padding: 10px 0;
        span {
          display: block;
          margin: 10px 0;
        }
      }
      /deep/img {
        display: block;
        width: 100%;
      }
    }
    // 标题大小
    h6 {
      margin-top: 20px;
      font-size: 36px;
    }
    // 横线
    .box {
      border-bottom: 1px solid rgb(179, 179, 179);
      margin: 25px 0px 25px 0;
    }
    .Title {
      height: 600px;
      .image {
        margin-top: 20px;
        width: 710px;
        height: 340px;
        img {
          margin-top: 20px;

          width: 100%;
        }
      }
    }
    // 图片

    // 攻略阅读
    .ntroduction_reading {
      margin-top: 20px;
      margin-left: 420px;
      margin-bottom: 20px;
      color: rgb(163, 163, 163);
    }
    // 图片标题
  }
  // <!-- 内容 -->

  // 评论
  .comment {
    // 评论分享
    .comment_in {
      margin-top: 30px;
      height: 60px;
      margin-left: 280px;
      img {
        margin-left: 25px;
        margin-bottom: 10px;
      }
      span {
        margin-left: 10px;
      }
    }
    .form {
      height: 255px;
      margin-top: 40px;

      span {
        font-size: 20px;
      }
      .glyuan {
        margin-top: 30px;
        margin-bottom: 10px;
      }
      .box3 {
        float: right;

        margin-top: 10px;
        height: 60px;
      }
      .box4 {
        margin-top: 20px;
      }
    }
  }
  .comment_frame {
    margin-top: 40px;
    border: 1px #eee solid;
    // 评论列表
    .comment1 {
      margin-top: 20px;
      padding: 10px;
      border: 1px #eee solid;
      font-size: 13px;
      .reply {
        margin-left: 600px;
      }
      .picture {
        img {
          border: 1px rgb(136, 134, 134) solid;
          float: left;
          width: 90px;
          height: 90px;
          margin-right: 4px;
          margin-top: 3px;
        }
      }
    }
    .comment-top {
      display: flex;
      justify-content: space-between;
      margin-bottom: 20px;

      .reply {
        font-size: 12px;
      }
      .user {
        display: flex;
        align-items: center;
        img {
          display: block;
          width: 20px;
          height: 20px;
          border-radius: 50%;
          margin-right: 5px;
        }

        span {
          color: #999;
          font-size: 12px;
        }
      }
      .reply {
        font-size: 12px;
      }
    }
  }
  .content {
    margin-top: 20px;
  }

  .publish {
    position: fixed;
    width: 100%;
    left: 0;
    bottom: 0;
    padding: 20px;
    box-sizing: border-box;
    background: #fff;
    display: flex;
    justify-content: space-between;
    align-items: flex-end;

    .textarea {
      background: #eee;
      border-radius: 50px;
      padding: 5px 15px;
    }

    .ative {
      height: 82px !important;
      border-radius: 8px;
    }

    .submit {
      margin-left: 5px;
      padding: 3px 10px;
      color: #fff;
      background: red;
      border-radius: 50px;
      font-size: 12px;
      flex-shrink: 0;
    }
  }
}
</style>