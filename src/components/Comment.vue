<template>
   <div class="comment-block">
    <div class='row between-xs bottom-xs'>
      <dir class="comment-user-block col-xs-6 row bottom-xs">
        <div class="comment-user-img">{{ comment ? comment.username.name[0].toUpperCase() : 'U'}}</div>
        <a href="#" class="comment-user-name">{{ comment ? comment.username.name : '' }}</a>
      </dir>
      <div class="comment-time col-xs-6 end-xs" v-if="comment.created_at">
        {{ [comment.created_at, "YYYY-MM-DD HH:mm:ss"] | moment("from") }}
      </div>
    </div>
    <div class="comment-body col-xs-12 col-sm">
      <div  class="comment-content"
            :class="{ 'comment-content-edited': wasEdited }"
            v-show="!editionMode">
        {{ comment ? comment.content : '' }}
        <!-- <div class='shadow' v-show="isUpdPending"></div> -->
      </div>
      <textarea-autosize
        v-show="editionMode"
        type="text"
        class="comment-content"
        rows="1"
        ref="changeRef"
        v-model="value"
        @blur.native="closeEditing"
        @keydown.native="operateKeyDown"
      ></textarea-autosize>
      <div class="comment-props row">
        <div class="comment-props-answer">
          <span @click="$emit('answer', comment.username.name, comment.username.id)">Ответить</span>
        </div>
        <div class="comment-props-like row">
          <a class="like">Спасибо</a>
          <i class="icon-thump-up like-icon"></i>
          <span class="like-counter">{{ (comment && comment.like) ? comment.like : 0 }}</span>
        </div>
        <div  class='comment-props-change-comment'
              v-show="isMyProfileId(comment.username.id) && !editionMode"
              @click="prepareEditing"
        >Редактировать</div>
        <div  class='comment-props-change-comment'
              v-show="isMyProfileId(comment.username.id) && editionMode"
              @mousedown="applyChanges"
        >Сохранить</div>
      </div>
    </div>
  </div> 
</template>
<script>
import { mapGetters } from 'vuex';
 import { TOPIC_UPD_COMMENT } from '../store/actions';
export default {
  name: 'Comment',
  props: ['comment'],
  data() {
    return {
      value: '',
      editionMode: false,
    };
  },
  computed: {
    ...mapGetters({
      isMyProfileId: 'isMyProfileId',
      isUpdPending: 'isCurrentTopicUpdCommentPending',
    }),
    wasEdited() {
      return this.comment.created_at
          && this.comment.updated_at
          && this.comment.updated_at !== this.comment.created_at
    }
  },
  methods: {
    prepareEditing() {
      if (this.value === '') {
        this.value = this.comment.content;
      }
      this.editionMode = true;
      setTimeout(() => this.$refs.changeRef.$el.focus());
    },
    closeEditing() {
      setTimeout(() => this.editionMode = false, 400);
    },
    operateKeyDown(e) {
      if (e.ctrlKey && e.key === "Enter") {
        this.applyChanges();
      } else if (e.key === "Escape") {
        this.clearValue();  
      }
    },
    async applyChanges() {
      const result = await this.$store.dispatch(TOPIC_UPD_COMMENT, { commentId: this.comment.id, content: this.value });
      if (result) {
        this.clearValue();
      }
      this.editionMode = false;  
    },
    clearValue() {
      this.editionMode = false;
      this.value = '';
    }
  }
}
</script>
<style lang="sass" scoped>
@import "../assets/variables"
.comment-block
  display: flex
  flex-direction: column
  align-items: stretch
  &:not(:last-child)
    margin-bottom: 10px
  &:nth-last-child(2)
    margin-bottom: 17px
  .comment-user-block
    padding-right: 0
    padding-left: 0
  .comment-user-img // заменить на фото
    height: 32px
    width: 32px
    border-radius: 50%
    background-color: $topic_block_background
    margin-right: 8px
    display: flex
    justify-content: center
    align-items: center
    font-weight: 700
    color: #333
  
  .comment-user-name
    font-size: $forun_item_secondary_font_size
    text-decoration: none
    margin-bottom: 6px
    &:hover
      opacity: 0.5
  .comment-time
    margin-bottom: 6px
    font-size: $forun_item_secondary_font_size
  .comment-body
    margin-left: 40px
    padding-right: 0
    padding-left: 0
    @media screen and ( max-width: 420px )
      margin-left: 0
      margin-top: 15px
  // .shadow // доработать
  //   position: absolute
  //   top: 0
  //   right: 0
  //   height: 100%
  //   width: 100%
  //   box-sizing: border-box
  //   font-size: 30px
  //   display: flex
  //   margin-left: 40px
  //   align-items: center
  //   justify-content: center
  //   z-index: 100
  //   border-radius: 4px
  //   background-color: $dark_background_color
  //   color: $text_background_color
  //   &:before
  //     content: "PENDING ... "

  .comment-content
    position: relative
    padding: 15px
    border-radius: 4px
    background-color: $comment_background_color
    word-wrap: break-word
    color: $base_font_color
    font-size: $base_font_size
    width: 100%
    outline: none
    border: 1px solid $comment_background_color
  
  textarea.comment-content
    margin-bottom: -4px // correction
    border-color: $dark_background_color
    background-color: $background-color
  div.comment-content
    padding-bottom: 14px  // correction
  
  .comment-content-edited
    &:before
      content: 'Сообщение отредактировано'
      position: absolute
      top: 2px
      right: 3px
      font-size: 10px
      font-weight: 500
      color: $dark_background_color

  .comment-props
    padding-top: 10px
    font-size: $forun_item_secondary_font_size
    line-height: normal
    color: $base_font_color
    & *
      font-size: inherit
      cursor: default
    &-answer
      padding-left: 15px
      @media screen and ( max-width: 420px )
        padding-left: 0
      span
        text-decoration: none
        color: inherit
    &-change-comment
      margin-left: auto
      color: inherit
    &-answer:hover,
    &-delete:hover,
    &-change-comment:hover,
    .like:hover
      opacity: 0.5
    &-like
      padding-left: 23px
      @media screen and ( max-width: 420px )
        padding-left: 8px
    .like,
    .like-icon
      margin-right: 4px
</style>
