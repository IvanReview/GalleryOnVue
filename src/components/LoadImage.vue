<template>
  <div class="form__wrapper">
    <div class="form__load-image">
      <div class="form__load-inputs">
        <input class="form__input-url" type="text" v-model="image_url" name="image" placeholder="Введите URL изображения">
        <input class="form__input-file" type="file" v-on:change="loadFile" ref="newFile" id="file">
        <label for="file"><span class="fileName" ref="fileName">File</span></label>
      </div>
      <button class="form__button" @click="LoadImage">Загрузить</button>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'LoadImage',
    components: {},
    data: () => ({
      image_url: '',
      file: '',
      images_from_file: [],
      gallery_images: [],
    }),

    methods: {
    // Загрузка изображения в свойство 
    LoadImage() {

      // для файла
      if (this.images_from_file.length) {

        this.gallery_images = this.images_from_file
        this.images_from_file = []
        this.$emit('loadData', this.gallery_images)

      } else if (this.image_url) {
        // для ссылки 

        //получение размеров изображения
        let newImg = new Image()
        newImg.src = this.image_url

        newImg.onload = () => {
          let height = newImg.height
          let width = newImg.width

          this.gallery_images = [{url: this.image_url, width, height}]

          this.image_url = ''
          this.$emit('loadData', this.gallery_images)
        }

      } else {

        alert('Введите данные!!')
      }
    },


    // обработка после загр в iput картинки или JSON-файла
    async loadFile() {

      const file = this.$refs.newFile.files[0]

      if (file) {
        const fileType = this.$refs.newFile.files[0].type
        const fileName = this.$refs.newFile.files[0].name
        const labelFile = this.$refs.fileName
        labelFile.innerHTML = fileName.substr(-15, 15)

        // если загруж json
        if (fileType === 'application/json') {
          this.file = file
          const reader = new FileReader()
          reader.readAsText(this.file)

          const result = await new Promise((resolve, reject) => {
            reader.onload = function (event) {

              resolve(JSON.parse(reader.result).galleryImages)
            }
          })

          this.images_from_file = result

        } else if (fileType.match(/image/gi)) {// если файл картинки

          this.$emit('giveFile', file)

        } else {

          alert('!!!Загружаемый файл должен быть картинкой или иметь JSON формат!!!')
        }
    }
  },

},
watch: {},

mounted() {

}

}
</script>
