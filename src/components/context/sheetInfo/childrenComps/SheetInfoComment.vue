<template>
  <div class="infoComment">
    <div class="nav">
      <div class="left" @click="back">
        <img src="~assets/img/commentList/fanhui.svg" alt="" />
      </div>
      <div class="center">
        {{ commentTitle }}
      </div>
      <div class="rightbox" @click="fxiang">
        <img src="~assets/img/commentList/fenxiang.svg" alt="" />
      </div>
    </div>
    <mscroll class="conscroll" :scrollY="true">
      <div
        class="content"
        v-if="commentList.length > 0"
        v-infinite-scroll="loadComment"
        infinite-scroll-distance="100"
        infinite-scroll-delay="500"
        infinite-scroll-immediate="fasle"
      >
        <div class="item" v-for="(item, index) in commentList" :key="index">
          <div class="topbox">
            <div class="userImg">
              <img @click="profile(item.userId)" v-lazy="item.userImg" alt="" />
            </div>
            <div class="userName">
              <div @click="profile(item.userId)" class="name">
                {{ item.userName }}
                <img
                  v-if="item.vipType !== 0"
                  src="~assets/img/common/vip1.svg"
                  alt=""
                />
              </div>
              <div class="addtime">
                {{ item.time | getTime }}
              </div>
            </div>
            <div class="liked">
              <div class="count">
                {{ item.likedCount }}
                <img :src="likedImg" alt="" />
              </div>
            </div>
          </div>
          <div class="bottombox">
            <div class="left"></div>
            <div class="right">
              {{ item.content }}
            </div>
          </div>
        </div>
      </div>
      <div class="noComment" v-else>
        <div class="text">没有更多评论了:(</div>
      </div>
    </mscroll>
    <send-comment
      @successComment="successComment"
      :type="type"
      :id="$route.params.id"
      class="SendComment"
    ></send-comment>
    <van-share-sheet
      v-model="showShare"
      title="立即分享给好友"
      :options="options"
    />
  </div>
</template>

<script>
import { getSheetComment } from "network/sheetInfo"; // 评论请求
import { getAlbumComment } from "network/album";
import sendComment from "components/context/sendComment/SendComment"; // 发送评论
import mscroll from "components/common/muiScroll/MuiScroll";
import mui from "assets/mui/js/mui.min.js"; // 引入 mui js 文件

export default {
  name: "SheetInfoComment",
  data() {
    return {
      commentTitle: "", // 评论数量
      offset: 0, // 评论页数
      commentList: [], // 评论内容
      likedImg: require("assets/img/commentList/zan.svg"),
      getComMore: null,
      showShare: false, // 分享面板
      options: [
        [
          { name: "微信", icon: "wechat" },
          { name: "微博", icon: "weibo" },
          { name: "QQ", icon: "qq" },
        ],
        [
          { name: "复制链接", icon: "link" },
          { name: "分享海报", icon: "poster" },
          { name: "二维码", icon: "qrcode" },
        ],
      ],
      commentLength: 1, // 判断是否加载评论
      more: true, // 是否有更多评论
      type: 2, // 评论类型
    };
  },
  components: {
    mscroll,
    sendComment,
  },
  methods: {
    successComment(commentDetail) {
      this.commentList.unshift({
        content: commentDetail.content, // 评论内容
        likedCount: 0, // 喜欢数量
        time: commentDetail.time, // 发布时间戳
        userImg: commentDetail.avatarUrl, // 用户头像
        userName: commentDetail.nickname, // 用户昵称
        userId: commentDetail.id, // 用户id
      });
    },

    // 跳转路由
    profile(id) {
      this.$router.push("/Information/" + id);
      this.$store.state.isShowNav = true;
    },

    getIndex(index) {
      this.listIndex = index;
    },

    // 分享组件
    fxiang() {
      this.showShare = true;
    },

    // 监听上拉加载
    loadComment() {
      // 发送请求
      // 判断评论组件是否打开
      if (this.$store.state.isShowNav === false) {
        this.$loading.loadingShow();
        setTimeout(() => {
          this.getCommentList(this.$route.params.id, 100, this.offset * 100);
          this.$loading.loadingNo();
        }, 1000);
      }
    },

    back() {
      this.$emit("toback");
    },

    // 封转方法
    getCommentList(sid, list, offset) {
      if (this.$route.params.isAlbum == "false") {
        this.type = 2;  // 评论类型--歌单
        // 发送网络请求
        if (this.more) {
          getSheetComment(sid, list, offset).then((res) => {
            this.more = res.data.more;
            this.commentTitle = "评论(" + res.data.total + ")";
            // 判断评论数量
            this.commentLength = res.data.comments.length;
            if (res.data.hotComments) {
              for (const item of res.data.hotComments) {
                this.commentList.push({
                  commentId: item.commentId, // 评论楼层id
                  content: item.content, // 评论内容
                  likedCount: item.likedCount, // 喜欢数量
                  time: item.time, // 发布时间戳
                  userImg: item.user.avatarUrl, // 用户头像
                  authStatus: item.user.authStatus, // 用户身份认证  1 表示歌手
                  userName: item.user.nickname, // 用户昵称
                  userId: item.user.userId, // 用户id
                  vipType: item.user.vipType, // 是否开通会员
                });
              }
            }

            for (const itemc of res.data.comments) {
              this.commentList.push({
                commentId: itemc.commentId, // 评论楼层id
                content: itemc.content, // 评论内容
                likedCount: itemc.likedCount, // 喜欢数量
                time: itemc.time, // 发布时间戳
                userImg: itemc.user.avatarUrl, // 用户头像
                authStatus: itemc.user.authStatus, // 用户身份认证  1 表示歌手
                userName: itemc.user.nickname, // 用户昵称
                userId: itemc.user.userId, // 用户id
                vipType: itemc.user.vipType, // 是否开通会员
              });
            }
            this.offset++; // 页数增加
          });
        } else {
          this.$toast.show("没有更多评论了:(", 1900);
        }
      } else if (this.$route.params.isAlbum == "true") {
        this.type = 3;  // 评论类型--歌单
        if (this.more) {
          getAlbumComment(sid, list, offset).then((res) => {
            this.more = res.data.more;
            this.commentTitle = "评论(" + res.data.total + ")";
            // 判断评论数量
            this.commentLength = res.data.comments.length;
            if (res.data.hotComments) {
              for (const item of res.data.hotComments) {
                this.commentList.push({
                  commentId: item.commentId, // 评论楼层id
                  content: item.content, // 评论内容
                  likedCount: item.likedCount, // 喜欢数量
                  time: item.time, // 发布时间戳
                  userImg: item.user.avatarUrl, // 用户头像
                  authStatus: item.user.authStatus, // 用户身份认证  1 表示歌手
                  userName: item.user.nickname, // 用户昵称
                  userId: item.user.userId, // 用户id
                  vipType: item.user.vipType, // 是否开通会员
                });
              }
            }

            for (const itemc of res.data.comments) {
              this.commentList.push({
                commentId: itemc.commentId, // 评论楼层id
                content: itemc.content, // 评论内容
                likedCount: itemc.likedCount, // 喜欢数量
                time: itemc.time, // 发布时间戳
                userImg: itemc.user.avatarUrl, // 用户头像
                authStatus: itemc.user.authStatus, // 用户身份认证  1 表示歌手
                userName: itemc.user.nickname, // 用户昵称
                userId: itemc.user.userId, // 用户id
                vipType: itemc.user.vipType, // 是否开通会员
              });
            }
            this.offset++; // 页数增加
          });
        }
      }
    },
  },
  created() {
    // 获取评论
    this.getCommentList(this.$route.params.id, 100, this.offset * 100);
  },
};
</script>
<style scoped>
.SendComment {
  position: absolute;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 20;
}
.infoComment {
  width: 100%;
  /* height: 100vh; */
  background: #fff;
  position: absolute;
  top: 0;
  bottom: 0;
  z-index: 60;
}
.CommentList {
  width: 100%;
  background: #fff;
  position: absolute;
  top: 0;
  z-index: 60;
}
.nav {
  width: 100%;
  height: 1.198402rem;
  display: flex;
  color: #fff;
  background-color: rgb(212, 81, 74);
  box-shadow: 0 -2px 5px rgba(0, 0, 0, 0.5);
}
.left {
  flex: 1;
}
.left img {
  width: 0.665779rem;
  height: 0.532623rem;
  position: relative;
  top: 12.5px;
  margin-left: 6px;
}
.center {
  flex: 7;
  line-height: 1.198402rem;
  font-size: 0.479361rem;
  /* font-size: .399467rem; */
}
.rightbox {
  flex: 1;
}
.rightbox img {
  width: 0.585885rem;
  height: 0.585885rem;
  position: relative;
  top: 11px;
  margin-left: 9px;
}
.comment {
  height: 13.848202rem;
}
.content {
  padding: 0.426099rem 0.319574rem;
}
.topbox {
  width: 100%;
  height: 1.065246rem;
  display: flex;
}
.userImg {
  flex: 1.2;
}
.userName {
  flex: 5;
  font-size: 0.346205rem;
  margin-top: 0.053262rem;
}
.name {
  position: relative;
}
.name img {
  height: 0.639148rem;
  margin-left: 3px;
  position: absolute;
  top: -3px;
}
.liked {
  flex: 2.2;
}
.addtime {
  font-size: 0.319574rem;
  color: #8b8b8b;
  margin-top: -4px;
}
.userImg img {
  width: 1.065246rem;
  height: 1.065246rem;
  border-radius: 50%;
  overflow: hidden;
}
.count {
  color: #8b8b8b;
  margin-top: 0.213049rem;
  font-size: 0.372836rem;
  float: right;
}
.count img {
  position: relative;
  top: 1px;
}
.bottombox {
  width: 100%;
  display: flex;
  padding-bottom: 0.186418rem;
}
.left {
  flex: 1.2;
}
.right {
  flex: 7;
  font-size: 0.364847rem;
  padding-bottom: 0.186418rem;
  border-bottom: 1px solid rgb(230, 230, 230);
}
.conscroll {
  top: 1.198402rem;
  background-color: #fff;
  height: calc(100vh - 1.065246rem - 1.065246rem);
}
.noComment {
  width: 100%;
  height: 100vh;
  text-align: center;
  color: #8b8b8b;
}
.text {
  position: relative;
  font-size: 0.559254rem;
  font-weight: 550;
  top: 45%;
  left: 50%;
  transform: translate(-50%, -50%);
}
</style>