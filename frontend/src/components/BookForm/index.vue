<template>
  <form class="book-form" @submit.prevent>
    <label class="book-form__field" :class="{ error: isTitleError }">
      Название
      <input class="book-form__input" v-model="form.title.value" @blur="form.title.touched = true" name="title" />
    </label>

    <div class="book-form__group">
      <label class="book-form__field" :class="{ error: isAuthorFirstNameError }">
        Имя
        <input
          class="book-form__input"
          v-model="form.author.firstName.value"
          @blur="form.author.firstName.touched = true"
          name="author-first-name"
        />
      </label>
      <label class="book-form__field" :class="{ error: isAuthorLastNameError }">
        Фамилия
        <input
          class="book-form__input"
          v-model="form.author.lastName.value"
          @blur="form.author.lastName.touched = true"
          name="author-last-name"
        />
      </label>
    </div>

    <label class="book-form__field" :class="{ error: isNumberOfPagesError }">
      Количество страниц
      <input
        class="book-form__input"
        v-model="form.numberOfPages.value"
        @blur="form.numberOfPages.touched = true"
        name="number-of-pages"
      />
    </label>
    <label class="book-form__field" :class="{ error: isPublishingHouseError }">
      Издательство
      <input
        class="book-form__input"
        v-model="form.publishingHouse.value"
        @blur="form.publishingHouse.touched = true"
        name="publishing-house"
      />
    </label>

    <div class="book-form__group">
      <label class="book-form__field" :class="{ error: isPublishingYearError }">
        Год издания
        <input
          class="book-form__input"
          v-model="form.publishingYear.value"
          @blur="form.publishingYear.touched = true"
          name="publishing-year"
        />
      </label>
      <label class="book-form__field" :class="{ error: form.releaseDate.error }">
        Дата выхода в тираж
        <input
          class="book-form__input"
          v-model="form.releaseDate.value"
          @blur="form.releaseDate.touched = true"
          name="release-date"
        />
      </label>
    </div>
    <label class="book-form__field">
      Картинка
      <input class="book-form__input" type="file" name="image" ref="imageFile" @change="handleFileChange" />
    </label>
    <button class="button book-form__button-save" @click="handleSaveClick">Сохранить</button>
    <button class="button book-form__button-cancel" @click="handleCancelClick">Отмена</button>
  </form>
</template>

<script>
import { book as bookValidation } from '@/mixins/validation/'
import { v4 as uuidV4 } from 'node-uuid'
import axios from '../../axios'

export default {
  name: 'BookForm',
  mixins: [bookValidation],
  data() {
    return {
      form: {
        title: { value: '', touched: false },
        author: {
          firstName: { value: '', touched: false },
          lastName: { value: '', touched: false }
        },
        numberOfPages: { value: '', touched: false },
        publishingHouse: { value: '', touched: false },
        publishingYear: { value: '', touched: false },
        releaseDate: { value: '', touched: false },
        image: { value: '', touched: false, file: '' }
      }
    }
  },
  computed: {
    isTitleError() {
      const { value, touched } = this.form.title
      return !this.validateTitle(value) && touched
    },
    isAuthorFirstNameError() {
      const { value, touched } = this.form.author.firstName
      return !this.validateAuthorFirstName(value) && touched
    },
    isAuthorLastNameError() {
      const { value, touched } = this.form.author.lastName
      return !this.validateAuthorLastName(value) && touched
    },
    isNumberOfPagesError() {
      const { value, touched } = this.form.numberOfPages
      return !this.validateNumberOfPages(value) && touched
    },
    isPublishingHouseError() {
      const { value, touched } = this.form.publishingHouse
      return !this.validatePublishingHouse(value) && touched
    },
    isPublishingYearError() {
      const { value, touched } = this.form.publishingYear
      return !this.validatePublishingYear(value) && touched
    },
    isReleaseDateError() {
      const { value, touched } = this.form.releaseDate
      return !this.validateReleaseDate(value) && touched
    },
    newBookData() {
      const bookData = {
        id: '',
        book: {}
      }
      if (!this.editedBookData) {
        bookData.id = uuidV4()
      } else {
        bookData.id = this.editedBookData.id
      }
      for (let field of Object.keys(this.form)) {
        if (field === 'author') {
          bookData.book[field] = {}
          bookData.book[field].firstName = this.form[field].firstName.value
          bookData.book[field].lastName = this.form[field].lastName.value
        } else {
          bookData.book[field] = this.form[field].value
        }
      }
      return bookData
    }
  },
  methods: {
    handleCancelClick() {
      this.$emit('cancel')
    },
    handleFileChange() {
      this.form.image.file = this.$refs.imageFile.files[0]
      this.form.image.touched = true
    },
    async handleSaveClick() {
      if (
        !this.validateTitle(this.form.title.value) ||
        !this.validateAuthorFirstName(this.form.author.firstName.value) ||
        !this.validateAuthorLastName(this.form.author.lastName.value) ||
        !this.validateNumberOfPages(this.form.numberOfPages.value) ||
        !this.validatePublishingHouse(this.form.publishingHouse.value) ||
        !this.validatePublishingYear(this.form.publishingYear.value) ||
        !this.validateReleaseDate(this.form.releaseDate.value)
      ) {
        return
      }
      const formData = new FormData()
      if (this.form.image.touched && this.form.image.file) {
        formData.append('image', this.form.image.file)
        formData.append('id', this.newBookData.id)
      }
      try {
        const response = await axios.post('/images/', formData, {
          headers: {
            'Content-Type': 'multipart/form-data'
          }
        })
        if (response.data && response.data.url) {
          this.form.image.value = response.data.url
        }
      } catch (error) {
        console.log(error)
      }

      this.$emit('save-book', this.newBookData)
    }
  },
  props: {
    editedBookData: {
      type: Object
    }
  },
  mounted() {
    if (this.editedBookData && this.editedBookData.book) {
      for (let key of Object.keys(this.editedBookData.book)) {
        if (key === 'author') {
          this.form[key].firstName.value = this.editedBookData.book[key].firstName
          this.form[key].lastName.value = this.editedBookData.book[key].lastName
        } else {
          this.form[key].value = this.editedBookData.book[key]
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.book-form {
  display: block;

  &__field {
    padding: 10px 0;
    display: block;
  }

  &__field.error {
    color: red !important;
  }

  &__field.error &__input {
    border: 1px solid red !important;
  }

  &__group &__field {
    width: 100%;
  }

  &__group &__field:last-child {
    margin-left: 10px;
  }

  &__group {
    display: flex;
  }

  &__input {
    margin-top: 5px;
    display: block;
    width: 100%;
    height: 20px;
    border: 1px solid #eee;
    border-radius: 5px;
  }

  &__button-save {
    background: #3c82ea;
    &:hover {
      background: #3676d1;
    }
  }

  &__button-cancel {
    background: #f22b2b;
    margin-left: 10px;
    &:hover {
      background: #d82727;
    }
  }
}

.button {
  padding: 10px 20px;
  cursor: pointer;
  border-radius: 8px;
  font-weight: bold;
  font-size: inherit;
  color: white;
}
</style>
