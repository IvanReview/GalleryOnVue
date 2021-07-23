<template>
  <div class="container">
    <div class="gallery__wrapper">

      <!--//форма загрузки изображений-->
      <load-image  
          @loadData="addImageToGallery" 
          @giveFile="getFileFromChildAndAdd"  
          ref="child" 
      />

      <div class="gallery gallery__container"  
          ref="gallery__container_ref" 
      >
        <div class="gallery__item" v-for="(image, index) in gallery_images" :key="index" @click="deleteImage(index)">
          <img :src="image.url" class="gallery__item-img">
        </div>

        <div class="gallery__item empty_container" 
             v-for="(cont, i) in container_count" 
             :key="i"
             ref="elements"
             :ref="el => { if (el) empty_containers[i] = el }" 
        > </div>
      </div>
   
      
    </div>
  </div>
</template>

<script >
import LoadImage from "./LoadImage";

export default {
  name: 'GalleryImage',
  components: {LoadImage},

  data: ()=> ({
      container_count: [1,2,3,4],
      empty_containers: [],
      gallery_images: [],
  }),

  methods: {
    //добавление изображение по URL и из файла- JSON
    addImageToGallery(data) {
      this.gallery_images = this.gallery_images.concat(data)
      this.gallery_images.length ? this.deleteEmptyContainer() : false
    },

    // удалить по клику
    deleteImage(index) {
      const ok = confirm('Точно хотите удалить изображение??')
      if (ok) {
        this.gallery_images.splice(index, 1)
      }
    },

    //удалить пустые контейнеры при загрузки картики из файла или по URL 
    deleteEmptyContainer() {

        let emptyConteiners = document.querySelectorAll('.empty_container')
      
        for (let index in emptyConteiners) {
          if (emptyConteiners.hasOwnProperty(index)) {

              emptyConteiners[index].remove()
          }
        }
      
    },

    //получить файл из дочернего компонента при загрузке DragAndDrop или просто файл изображения
    getFileFromChildAndAdd(file) {

        //создаем ноый контейнер для загрузки картинки и добавляем родителю
        let galleryContainer = this.$refs.gallery__container_ref
        let emptyConteiners = galleryContainer.querySelectorAll('.empty_container')
   
        if (emptyConteiners.length < 1) {

          const el = document.createElement('div')
          el.classList.add('gallery__item', 'empty_container')
          galleryContainer.prepend(el)
          /*this.empty_containers.push(el)*/

          emptyConteiners = galleryContainer.querySelectorAll('.empty_container')
        }                

        // вызываем метод отображения картинки
        this.showImage(file, emptyConteiners[0])

        //удаление добавленной через DragAndDrop картинки по клику
        emptyConteiners[0].addEventListener('click', () => {
          emptyConteiners[0].remove()
        })
    },

    //перетащить картинку
    dragAndDrop(areaWhenDragFile, inputFileField) {
      ['dragenter', 'dragover', 'dragleave', 'drop'].forEach((eventName, index) => {
        areaWhenDragFile.addEventListener(eventName, (e) => {
          e.preventDefault()
          e.stopPropagation()

          if (index < 2) {
            areaWhenDragFile.style.background = '#000'
          } else {
            areaWhenDragFile.style.background = '#ffa726'

            if (index === 3 && inputFileField.files) {
              // dataTransfer хранит объекты перетаскиваемые мышью
              inputFileField.files = e.dataTransfer.files

              // вызвать событие change, а при событии change срабатывает уже другая функц загрузки изобр
              inputFileField.dispatchEvent(new Event('change'))
            }
          }
        })
      })
    },

    // показать картинку.jpg сразу после загрузки
    showImage(file, container) {
      const reader = new FileReader()

      // содержимое контейнера удаляем
      container.innerHTML = ''

      reader.readAsDataURL(file)

      reader.onload = function (e) {
        // внутри контейнера создаем тег img
        container.innerHTML = '<img class="gallery__item-img" src="">'

        // вставляем в img файл
        container.querySelector('img').setAttribute('src', reader.result)

        container.classList.remove('empty_container')

      }
    }

  },

  mounted () {

    console.log(this.$refs.elements)

    const inputFileFieldGallery = document.querySelector('.form__input-file')
    const areaWhenDragFile = this.$refs.gallery__container_ref

    this.dragAndDrop(areaWhenDragFile, inputFileFieldGallery)
  }
}
</script>


<style lang="scss">

  .container {
    max-width: 860px;
    background-color:  #ffa726 ;
    margin: 40px auto;
    .gallery__wrapper{
      border: 2px solid black;
      .gallery {
        display: flex;
        justify-content: flex-start;
        margin: 20px 10px;
        flex-wrap: wrap;
        .empty_container{
          max-width: 190px;
          min-width: 180px;
          min-height: 100px;
          max-height: 100%;
          margin: 8px 10px;
          border: 1px dotted #000;
        }
        .gallery__item{
          background-color: rgba(128,0,128,.1);
          border: 1px dotted #000;
          margin: 8px 8px;
          display: flex;
          align-items: center;
          cursor: pointer;

          .gallery__item-img{
            max-width: 190px;
            min-width: 140px;
          }
        }
      }
    }
    .load {
      margin: 20px;
    }
    .form__wrapper {
      padding: 30px 20px;
      background-color: black;
      margin: auto;

      .form__load-image {
        display: flex;
        justify-content: space-between;
        flex-wrap: wrap;
        .form__input-url {
          min-width: 250px;
          max-width: 600px;
          padding: 14px;
          border: none;
          box-shadow: 0 0 15px 4px rgba(0, 0, 0, 0.6);
          font-size: 16px;
        }

        .form__input-file {
          width: 0.01px;
          height: 0.01px;
          opacity: 0;
          overflow: hidden;
          position: absolute;
          z-index: -1;
        }

        .form__input-file + label {
          font-size: 16px;
          text-transform: uppercase;
          color: white;
          background-color: #ffa726;
          box-shadow: 0 0 10px black;
          display: inline-block;
          cursor: pointer;
          padding: 15px 30px;
          transition: all .3s ease;
        }

        .form__input-file:focus + label,
        .form__input-file + label:hover {
          background-color: plum;
        }

        .form__button {
          background-color: teal;
          padding: 15px 30px;
          border: none;
          color: #fff;
          text-transform: uppercase;
          transition: all .5s ease;
          cursor: pointer;
          font-size: 16px;

          &:hover {
            background-color: #d84315;
            transition: all .5s ease;
          }

          &:active, &:focus {
            outline: none;
          }
        }
      }
    }
    .no-image{
      text-align: center;
      font-size: 25px;
    }
  }
  @media (max-width: 873px)  {
    .container {
      .gallery__wrapper {
        color: black;
        .gallery {
          justify-content: space-evenly !important;
        }
      }
    }
  }

  @media (max-width: 775px)  {
    .container {
      .gallery__wrapper {
        color: black;
        .gallery {
          justify-content: space-evenly !important;
        }
      }
    }
  }

  @media (max-width: 545px)  {
    .container {
      .form__load-image {
        display: flex;
        justify-content: center;
        flex-wrap: wrap;

        .form__load-inputs {
          display: flex;
          flex-wrap: wrap;
        }
        .form__button {
          margin: 20px 0;
          padding: 15px 40px !important;
        }
      }
    }
  }

  @media (max-width: 445px) {
    .container {
      .gallery__wrapper {
        .gallery {
          .gallery__item {
            .gallery__item-img {
              max-width: 220px;
              min-width: 180px;
            }
          }
        }

        .form__button {
          margin: 10px;
        }

        .form__load-image {
          display: flex;
          justify-content: center;
          .form__load-inputs {
            display: flex;
            justify-content: center;
            flex-wrap: wrap;
            .form__input-url {
               min-width: 280px;
               max-width: 300px;
            }
            .form__input-file + label{
              color: black;
              margin: 20px;
            }
          }
        }
      }
    }
  }
  @media (max-width: 379px)  {
    .container {
      .form__load-image {
        .form__load-inputs {
          .form__input-url {
            min-width: 200px !important;
            max-width: 250px;
            font-size: 15px;
          }
        }
      }
    }
  }
</style>
