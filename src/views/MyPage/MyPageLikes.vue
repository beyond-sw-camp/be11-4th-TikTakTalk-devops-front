<template>
  <v-container>
    <v-row>
      <!-- 왼쪽 프로필 카드 -->
      <UserProfileCard
        :avatar="userDetail.profileImageOfAuthor"
        :userName="userDetail.authorNickName"
        :userBio="'내가 좋아요한 글 목록입니다.'"
        selectedMenu="likes"
      />

      <!-- 오른쪽 게시글 목록 -->
      <v-col cols="8">
        <v-card class="pa-4">
          <v-card-title class="title-text">내가 좋아요한 게시글</v-card-title>
          <v-divider></v-divider>
          <v-card-text v-if="likePost.length">
            <v-row>
              <v-col v-for="myLikePost in likePost" :key="myLikePost.postId" cols="12">
                <v-card class="post-card" @click="goToDetailPost(myLikePost.postId)">
                  <v-card-text>
                    <!-- 게시글 정보 -->
                    <v-row no-gutters class="align-center">
                      <!-- 프로필 이미지 -->
                      <v-col cols="auto">
                        <v-avatar size="40">
                          <img
                            :src="myLikePost.authorImage || require('@/assets/basicProfileImage.png')"
                            class="profile-img"
                          />
                        </v-avatar>
                      </v-col>
                      <v-col class="user-info">
                        <div class="nickname">{{ myLikePost.authorNickName }}</div>
                        <div class="date">{{ formatDate(myLikePost.createdTime) }}</div>
                      </v-col>
                    </v-row>

                    <!-- 제목 -->
                    <v-row no-gutters>
                      <v-col>
                        <h3 class="post-title">
                          {{ myLikePost.title }}
                        </h3>
                      </v-col>
                    </v-row>

                    <!-- 게시글 내용 미리보기 -->
                    <v-row no-gutters>
                      <v-col>
                        <p class="text-preview">
                          {{ truncatedContent(removeHtmlTags(myLikePost.contents), 100) }}
                        </p>
                      </v-col>
                    </v-row>

                    <!-- 좋아요 및 댓글 정보 -->
                    <v-row no-gutters class="post-meta mt-2">
                      <span>👍 {{ myLikePost.likesCount }}</span>
                      <span class="ml-3">💬 {{ myLikePost.countOfComment }}</span>
                    </v-row>
                  </v-card-text>
                </v-card>
              </v-col>
            </v-row>
          </v-card-text>
          <v-card-text v-else>
            좋아요한 게시글이 없습니다.
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
import UserProfileCard from "@/components/UserProfileCard.vue"; // 프로필 카드 컴포넌트 import

export default {
  components: {
    UserProfileCard,
  },
  data() {
    return {
      userDetail: {}, // 유저 정보 저장
      likePost: [],
    };
  },
  async created() {
    try {
      const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/user/myLikeList`);
      this.likePost = response.data.result.content;

      console.log("저장된 likePost 데이터:", this.likePost);
      // 유저 정보 가져오기 (게시글 리스트에서 첫 번째 게시글의 유저 정보 활용)
      if (this.likePost.length > 0) {
        this.userDetail = {
          AuthorImage: this.likePost[0].authorImage || "",
          authorNickName: this.likePost[0].authorNickName || "익명",
        };
      }
    } catch (error) {
      console.error("Axios 요청 오류:", error);
    }
  },
  methods: {
    goToDetailPost(postId) {
      console.log("선택된 postId:", postId);
      if (!postId) {
        console.error("postId가 정의되지 않았습니다.");
        return;
      }
      this.$router.push(`/ttt/post/${postId}`);
    },
    truncatedContent(text, length = 100) {
      if (!text) return "";
      return text.length > length ? text.slice(0, length) + "..." : text;
    },
    removeHtmlTags(text) {
      if (text) {
        const doc = new DOMParser().parseFromString(text, "text/html");
        return doc.body.textContent || "";
      } else {
        return "";
      }
    },
    formatDate(dateArray) {
      if (!dateArray || dateArray.length < 6) return "";
      return `${dateArray[0]}-${String(dateArray[1]).padStart(2, "0")}-${String(dateArray[2]).padStart(2, "0")}`;
    },
  },
};
</script>

<style scoped>
/* 제목 스타일 */
.title-text {
  font-size: 20px;
  font-weight: bold;
  margin-bottom: 10px;
}

/* 게시글 카드 스타일 */
.post-card {
  cursor: pointer;
  transition: 0.3s ease-in-out;
  border-radius: 12px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 15px;
  padding: 16px;
}

/* 마우스 호버 시 카드 효과 */
.post-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 6px 12px rgba(98, 0, 234, 0.3);
}

/* 게시글 리스트 아이템 */
.post-item {
  padding: 16px;
  transition: background-color 0.3s ease-in-out, transform 0.2s ease-in-out;
  border-bottom: 1px solid #eee;
}

/* 마우스 호버 시 효과 */
.post-item:hover {
  background-color: #f3f3f3;
  transform: translateX(5px);
}

/* 프로필 이미지 */
.profile-img {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  object-fit: cover;
}

/* 사용자 정보 */
.user-info {
  margin-left: 10px;
}

.nickname {
  font-weight: bold;
  font-size: 16px;
}

.date {
  font-size: 13px;
  color: #666;
}

/* 제목 스타일 */
.clickable-title {
  font-size: 18px;
  font-weight: bold;
  cursor: pointer;
  transition: color 0.2s;
}

.clickable-title:hover {
  color: #6200ea;
  text-decoration: underline;
}

/* 내용 스타일 */
.clickable-content {
  font-size: 14px;
  color: #666;
  cursor: pointer;
  transition: color 0.2s;
}

.clickable-content:hover {
  color: #333;
}

/* 좋아요 및 댓글 정보 */
.post-meta {
  display: flex;
  align-items: center;
  gap: 12px;
  font-size: 14px;
  color: #666;
}

.post-meta i {
  font-size: 16px;
  margin-right: 4px;
}

/* 반응형 설정 */
@media (max-width: 768px) {
  .post-card {
    padding: 10px;
  }
  .clickable-title {
    font-size: 16px;
  }
  .clickable-content {
    font-size: 12px;
  }
  .post-meta {
    font-size: 12px;
  }
}

</style>
