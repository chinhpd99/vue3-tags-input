<template>
  <div @click="focusNewTag()"
       v-click-outside="closeContextMenu"
       :class="{
          'v3ti--focus': isInputActive,
          'v3ti--error': isError
        }"
       class="v3ti">
    <div class="v3ti-content"
         ref="inputBox"
         :class="{ 'v3ti-content--select': select }">
        <span v-for="(tag, index) in innerTags"
              :key="index"
              class="v3ti-tag">
          <slot v-if="isShot('item')"
                name="item" v-bind="{ name: tag, index, tag }"></slot>
          <span v-else class="v3ti-tag-content"> {{ tag }} </span>
          <a v-if="!readOnly" @click.prevent.stop="remove(index)" class="v3ti-remove-tag"></a>
        </span>
      <input
          ref="inputTag"
          :placeholder="placeholder"
          v-model="newTag"
          @keydown.delete.stop="removeLastTag"
          @keydown="addNew"
          @blur="handleInputBlur"
          @focus="handleInputFocus"
          @input="makeItNormal"
          class="v3ti-new-tag"
          :disabled="readOnly"/>
    </div>
    <section v-if="select"
             class="v3ti-context-menu"
             :class="{ 'v3ti-context-menu-no-data': !isShowNoData && selectItems.length === 0 }"
             ref="contextMenu">
      <div v-if="loading"
           class="v3ti-loading">
        <slot v-if="isShot('loading')"></slot>
        <Loading v-else/>
      </div>
      <div v-if="!loading && selectItems.length === 0 && isShowNoData"
           class="v3ti-no-data">
        <slot v-if="isShot('no-data')" name="no-data"></slot>
        <span v-else>
          No data
        </span>
      </div>
      <div v-if="!loading && selectItems.length > 0">
        <div v-for="(item, index) in selectItems"
             :key="index"
             class="v3ti-context-item"
             :class="{'v3ti-context-item--active': isShowCheckmark(item)}"
             @click.stop="handleSelect(item, index)">
          <div class="v3ti-context-item--label">
            <slot name="select-item"
                  v-bind="item"></slot>
          </div>
          <svg v-if="isShowCheckmark(item)"
               class="v3ti-icon-selected-tag"
               width="44" height="44"
               viewBox="0 0 24 24"
               stroke-width="1.5"
               fill="none"
               stroke-linecap="round"
               stroke-linejoin="round">
            <path stroke="none" d="M0 0h24v24H0z"/>
            <path d="M5 12l5 5l10 -10" />
          </svg>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
import vClickOutside from 'click-outside-vue3';
import Loading from './components/Loading.vue';

export default {
  name: "Vue3TagsInput",
  emits: ['update:modelValue', 'update:tags', 'on-limit', 'on-tags-changed', 'on-remove',
    'on-error', 'on-focus', 'on-blur', 'on-select', 'on-select-duplicate-tag', 'on-new-tag'],
  props: {
    readOnly: {
      type: Boolean,
      default: false
    },
    modelValue: {
      type: String,
      default: '',
    },
    validate: {
      type: [String, Function, Object],
      default: ""
    },
    addTagOnKeys: {
      type: Array,
      default: function () {
        return [
          13, // Enter
          ',', // Comma ','
          32, // Space
        ];
      }
    },
    placeholder: {
      type: String,
      default: ''
    },
    tags: {
      type: Array,
      default: () => []
    },
    loading: {
      type: Boolean,
      default: false
    },
    limit: {
      type: Number,
      default: -1
    },
    allowDuplicates: {
      type: Boolean,
      default: false
    },
    addTagOnBlur: {
      type: Boolean,
      default: false
    },
    selectItems: {
      type: Array,
      default: () => []
    },
    select: {
      type: Boolean,
      default: false
    },
    duplicateSelectItem: {
      type: Boolean,
      default: true
    },
    uniqueSelectField: {
      type: String,
      default: 'id'
    },
    addTagOnKeysWhenSelect: {
      type: Boolean,
      default: false
    },
    isShowNoData: {
      type: Boolean,
      default: true
    },
    // multiple: {
    //   type: Boolean,
    //   default: false
    // },
  },
  components: { Loading },
  directives: {
    clickOutside: vClickOutside.directive
  },
  data() {
    return {
      isInputActive: false,
      isError: false,
      newTag: '',
      innerTags: [],
      multiple: false
    }
  },
  computed: {
    isLimit() {
      const isLimit = this.limit > 0 && Number(this.limit) === this.innerTags.length;
      if (isLimit) {
        this.$emit('on-limit');
      }
      return isLimit;
    },

    selectedItemsIds() {
      if (!this.duplicateSelectItem) {
        return this.tags.map(o => o[this.uniqueSelectField] || '');
      }
      return []
    },
  },
  watch: {
    error() {
      this.isError = this.error;
    },
    modelValue: {
      immediate: true,
      handler(value) {
        this.newTag = value;
      }
    },
    tags: {
      deep: true,
      immediate: true,
      handler(tags) {
        this.innerTags = [...tags]
      }
    },
  },
  methods: {
    isShot(name) {
      return !!this.$slots[name]
    },
    makeItNormal(event) {
      this.$emit('update:modelValue', event.target.value)
      this.$refs.inputTag.className = 'v3ti-new-tag';
      this.$refs.inputTag.style.textDecoration = "none";
    },
    resetData() {
      this.innerTags = []
    },
    resetInputValue() {
      this.newTag = '';
      this.$emit('update:modelValue', '');
    },

    setPosition() {
      const el = this.$refs.inputBox;
      const menu = this.$refs.contextMenu;
      if (el && menu) {
        menu.style.display = "block";
        // menu.style.position = 'fixed';
        const ELEMENT_HEIGHT = el.clientHeight || 32;
        const BORDER_HEIGHT = 3;
        menu.style.top = ELEMENT_HEIGHT + BORDER_HEIGHT + "px";
      }
    },

    closeContextMenu() {
      if (this.$refs.contextMenu) {
        this.$refs.contextMenu.style = { display: 'none' };
      }
    },

    handleSelect(tagData) {
      if (this.isShowCheckmark(tagData)) {
        const tags = this.tags.filter(o => tagData.id !== o.id);
        this.$emit('update:tags', tags);
        this.$emit('on-select-duplicate-tag', tagData);
        this.resetInputValue();
      } else {
        this.$emit('on-select', tagData);
      }
      this.$nextTick(() => {
        this.closeContextMenu()
      })
    },

    isShowCheckmark(tag) {
      if (!this.duplicateSelectItem) {
        return this.selectedItemsIds.includes(tag[this.uniqueSelectField]);
      }
      return false
    },

    focusNewTag() {
      if (this.select && !this.disabled) {
        this.setPosition();
      }
      if (this.readOnly || !this.$el.querySelector(".v3ti-new-tag")) {
        return;
      }
      this.$el.querySelector(".v3ti-new-tag").focus();
    },

    handleInputFocus(event) {
      this.isInputActive = true;
      this.$emit('on-focus', event);
    },
    handleInputBlur(e) {
      this.isInputActive = false;
      this.addNew(e);
      this.$emit('on-blur', e);
    },
    addNew(e) {
      if (this.select && !this.addTagOnKeysWhenSelect) {
        return;
      }
      const keyShouldAddTag = e
          ? this.addTagOnKeys.indexOf(e.keyCode) !== -1 || this.addTagOnKeys.indexOf(e.key) !== -1
          : true;
      const typeIsNotBlur = e && e.type !== "blur";
      if (
          (!keyShouldAddTag && (typeIsNotBlur || !this.addTagOnBlur)) ||
          this.isLimit
      ) {
        return;
      }
      if (
          this.newTag &&
          (this.allowDuplicates || this.innerTags.indexOf(this.newTag) === -1) &&
          this.validateIfNeeded(this.newTag)
      ) {
        this.innerTags.push(this.newTag);
        if (this.addTagOnKeysWhenSelect) {
          this.$emit('on-new-tag', this.newTag);
          this.updatePositionContextMenu();
        }
        this.resetInputValue();
        this.tagChange();
        e && e.preventDefault();
      } else {
        if (this.validateIfNeeded(this.newTag)) {
          this.makeItError(true);
        } else {
          this.makeItError(false);
        }
        e && e.preventDefault();
      }
    },
    updatePositionContextMenu() {
      this.$nextTick(() => {
        this.setPosition()
      });
    },
    makeItError(isDuplicatedOrMaxLength) {
      if (this.newTag !== '') {
        this.$refs.inputTag.className = 'v3ti-new-tag v3ti-new-tag--error';
        this.$refs.inputTag.style.textDecoration = "underline";
        this.$emit('on-error', isDuplicatedOrMaxLength);
      }
    },
    validateIfNeeded(tagValue) {
      if (this.validate === "" || this.validate === undefined) {
        return true;
      }
      if (typeof this.validate === "function") {
        return this.validate(tagValue);
      }
      return true;
    },
    removeLastTag() {
      if (this.newTag) {
        return;
      }
      this.innerTags.pop();
      this.tagChange();
      this.updatePositionContextMenu();
    },
    remove(index) {
      this.innerTags.splice(index, 1);
      this.tagChange();
      this.$emit("on-remove", index);
      this.updatePositionContextMenu();
    },
    tagChange() {
      this.$emit("on-tags-changed", this.innerTags);
    }
  }
};
</script>

<style lang="scss">
$blackColor: #000000;
$errorColor: #F56C6C;
$primaryColor: #317CAF;
$successColor: #19be6b;
$paddingItem: 4px 7px;
$marginTag: 3px;
$paddingTag: 5px;
.v3ti {
  border-radius: 5px;
  min-height: 32px;
  line-height: 1.4;
  background-color: #fff;
  border: 1px solid #9ca3af;
  cursor: text;
  text-align: left;
  -webkit-appearance: textfield;
  display: flex;
  flex-wrap: wrap;
  position: relative;
  .v3ti-icon-selected-tag {
    stroke: $successColor;
    width: 1rem;
    height: 1rem;
    margin-left: 4px;
  }

  &--focus {
    outline: 0;
    border-color: $blackColor;
    box-shadow: 0 0 0 1px $blackColor;
  }

  &--error {
    border-color: $errorColor;
  }

  .v3ti-no-data {
    color: #d8d8d8;
    text-align: center;
    padding: $paddingItem;
  }

  .v3ti-loading {
    padding: $paddingItem;
    text-align: center;
  }

  .v3ti-context-menu {
    max-height: 150px;
    min-width: 150px;
    overflow: auto;
    display: none;
    outline: none;
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    margin: 0;
    padding: 5px 0;
    background: #ffffff;
    z-index: 1050;
    color: #475569;
    box-shadow: 0 3px 8px 2px rgba(0, 0, 0, .1);
    border-radius: 0 0 6px 6px;
    .v3ti-context-item {
      padding: $paddingItem;
      display: flex;
      align-items: center;
      &:hover {
        background: #e8e8e8;
        cursor: pointer;
      }
      &--label {
        flex: 1;
        min-width: 1px;
      }
      &--active {
        color: $primaryColor;
      }
    }
  }

  .v3ti-context-menu-no-data {
    padding: 0;
  }

  .v3ti-content {
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    &--select {
      padding-right: 30px;
    }
  }

  .v3ti-tag {
    display: flex;
    font-weight: 400;
    margin: $marginTag;
    padding: 0 $paddingTag;
    background: $primaryColor;
    color: #ffffff;
    height: 27px;
    border-radius: 5px;
    align-items: center;
    max-width: calc(100% - (($marginTag * 2) + ($paddingTag * 2)));
    .v3ti-tag-content {
      flex: 1;
      min-width: 1px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .v3ti-remove-tag {
      color: #ffffff;
      transition: opacity .3s ease;
      opacity: .5;
      cursor: pointer;
      padding: 0 5px 0 7px;

      &::before {
        content: "x";
      }

      &:hover {
        opacity: 1;
      }
    }
  }

  .v3ti-new-tag {
    background: transparent;
    border: 0;
    font-weight: 400;
    margin: 3px;
    outline: none;
    padding: 0 4px;
    flex: 1;
    min-width: 60px;
    height: 27px;

    &--error {
      color: $errorColor;
    }
  }
}
</style>