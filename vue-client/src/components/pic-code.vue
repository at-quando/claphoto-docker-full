<template>
  <div id="cp-content-wrap" class="cp-content-wrap">
    <loading-time id="loadingpos"/>
    <picture-box id="pictureBox" :dataPic="picView" @closeBox="getCloseBox" @nextImg="getNextImg" @prevImg="getPrevImg"/>
    <div class="container view-container">
      <div class="row"> 
        <div class="cp-pagination col-md-10 col-sm-10 view-pic">
          <div v-if="contract">
            <h2>{{contract.group}} - {{contract.school}}</h2>
            <p>{{contract.town}}</p>
            <p>Niên Khóa: {{contract.school_year}}</p>
            <p>Tổng: <b>{{totalPic()}} tấm</b></p>
            <p>Link ảnh gốc </p>
            <div v-if="raws != null">
            <p v-for="(item, index) of raws.drive_link.split(',')" :key="index"><a target="_blank" :href="item" >{{item}}</a></p>
            </div>
            <hr>
          </div>
          <nav>
            <ul class="pagination">
              <li>
                <a href="#" aria-label="Previous">
                  <span aria-hidden="true">&laquo;</span>
                </a>
              </li>
              <li v-for="(item,index) in pages" :key="index">
                <router-link :to="{name: 'PicCode', params: {id: $route.params.id}, query: {page: (index +1)}}" :class="{active: $route.query.page == (index + 1) }" replace>{{index + 1}}</router-link>
              </li>
              <li>
                <a href="#" aria-label="Next">
                  <span aria-hidden="true">&raquo;</span>
                </a>
              </li>
            </ul>
          </nav>
        </div>
        <div class="col-md-2 col-sm-2"></div>
        <div class="col-md-10 col-sm-10 view-pic">
          <transition name="fade">
            <div class="cp-grid-isotope gallery" v-if="pictures && ($route.query.page ? $route.query.page : 1)  == (index + 1)" v-for="(itemPics, index) of pictures" :key="itemPics">
              <p style="text-align: center">{{itemPics.pictureId.split(',').length}} tấm trong thư mục này</p>
              <div class="isotope items">
                <div v-lazy-container="{ selector: 'img' }" class="item" v-for="(item, ind) of itemPics.pictureId.split(',')" :key="ind">
                  <figure class="cp-hover-eff"> 
                    <img class="product-photo" alt="img02" :data-src="item | smallGoogleImage" keep-alive/>
                    <figcaption>
                      <h3>{{itemPics.name.split(',')[ind]}}</h3>
                      <a class="open-image" @click="openImage(item, ind)"><i class="fa fa-search"></i> View Large</a> 
                    </figcaption>
                  </figure>
                </div>
              </div>
            </div>
          </transition>
          <div class="cp-pagination">
            <nav>
              <ul class="pagination">
                <li>
                  <a href="#" aria-label="Previous">
                    <span aria-hidden="true">&laquo;</span>
                  </a>
                </li>
                <li v-for="(item,index) in pages" :key="index">
                  <router-link :to="{name: 'PicCode', params: {id: $route.params.id}, query: {page: (index +1)}}" :class="{active: $route.query.page == (index + 1) }" replace>{{index + 1}}</router-link>
                </li>
                <li>
                  <a href="#" aria-label="Next">
                    <span aria-hidden="true">&raquo;</span>
                  </a>
                </li>
              </ul>
            </nav>
          </div>
        </div>
        <div class="col-md-3 col-sm-3 side-bar-pic">
          <side-bar/>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import * as types from '../store/types'

var count = 0;
export default {
  name: 'PicCode',
  data () {
    return {
      pos: null,
      currentPage: null,
      show: false,
      pictures: null,
      picArr: null,
      nameArr: null,
      data: [],
      busy: false,
      contract: null,
      intervalId: null,
      raws: null,
      pages: 0,
      picView: null
    }
  },
  created () {
    $('#loadingpos').show()
    window.addEventListener('load', () => {$('#loadingpos').hide();})
    this.$http.get(`${types.SHOW_VIEWER}/${this.$route.params.code}`).then(res => {
      this.contract = res.body.contract
      this.pictures = res.body.meta.pic
      this.raws = res.body.meta.raw
      console.log(12, res.body)
      this.pages = res.body.meta.count
    })
    // this.$Lazyload.$on('loaded', function (listener) {
    //   console.log(222123123)
    // })
    this.intervalId = window.setInterval(() => {
        $(window).trigger("resize")
        console.log('resize')
    },15000)
  },
  mounted () {
    // if (this.$route.query.pictureId && this.$route.query.pos) {
    //   this.openImage(this.$route.query.pictureId, this.$route.query.pos)
    // }
  },
  updated() {
    if (this.$route.query.pictureId && this.$route.query.pos) {
      this.openImage(this.$route.query.pictureId, this.$route.query.pos)
    }
    if ($(".cp-grid-isotope .isotope").length) {
      var $container = $('.cp-grid-isotope .isotope');
      $container.isotope({
        itemSelector: '.item',
        transitionDuration: '0.6s',
        masonry: {
            columnWidth: $container.width() / 12
        },
        layoutMode: 'masonry'
      });
      $(window).resize(function() {
        $container.isotope({
          masonry: {
            columnWidth: $container.width() / 12
          }
        });
      });
    }
    if ($('.gallery').length) {
      $("area[data-rel^='prettyPhoto']").prettyPhoto()
      $(".gallery:first a[data-rel^='prettyPhoto']").prettyPhoto({
        animation_speed:'normal',
        theme:'light_square',
        slideshow:3000, 
        autoplay_slideshow: false
      });
    }
    setTimeout(() => {$(window).trigger("resize")},1500)
  },
  methods: {
    convertPicId (index) {
      index = index? index -1 : 0
      return this.pictures[index].pictureId.split(',')
    },
    loadMore () {
      this.busy = true;
      console.log(1332)
      setTimeout(() => {
        for (var i = 0, j = 10; i < j; i++) {
          this.data.push({ name: count++ });
        }
        this.busy = false;
      }, 1000);
    },
    openImage (value, index) {
      // this.show = true
      var page = this.$route.query.page ? this.$route.query.page : 1
      window.history.pushState(null, '', `?page=${page}&pictureId=${value}&pos=${index}`)
      $('#pictureBox').show()
      $('.pic-view').attr('src', this.$options.filters.mediumGoogleImage(value))
      $('.pic-view').attr('pos', index)
      $('.pic-view').attr('page', page)
      $('.pic-download').attr('href', this.$options.filters.fullGoogleImage(value))
      // this.pos = index
      // this.currentPage = this.$route.query.page ? this.$route.query.page : 1
      // this.picView = value
    },
    convertPicName (index) {
      index = index? index -1 : 0
      return this.pictures[index].name.split(',')
    },
    getCloseBox (value) {
      window.history.pushState(null, '', `?page=${this.$route.query.page ? this.$route.query.page : 1}`)
      $('#pictureBox').hide()
    },
    getNextImg (value) {
      let page = +$('.pic-view').attr('page') - 1
      let range = this.pictures[page].pictureId.split(',').length
      let pos = +$('.pic-view').attr('pos') === (range - 1) ? 0 : (+$('.pic-view').attr('pos') + 1)
      this.setImage(pos, page)
    },
    getPrevImg (value) {
      let page = +$('.pic-view').attr('page') - 1
      let range = this.pictures[page].pictureId.split(',').length
      let pos = +$('.pic-view').attr('pos') === 0 ? (range - 1) : (+$('.pic-view').attr('pos') - 1)
      this.setImage(pos, page)
    },
    setImage (pos, page) {
      var picView = this.pictures[page].pictureId.split(',')[pos]
      window.history.replaceState(null, '', `?page=${page + 1}&pictureId=${picView}&pos=${pos}`)
      $('.pic-view').attr('pos', pos)
      $('.pic-view').attr('page', this.$route.query.page ? this.$route.query.page : 1)
      $('.pic-view').attr('src', this.$options.filters.mediumGoogleImage(picView))
      $('.pic-view').on('load', function(){
        $('#loadingpos').hide();
      });
     $('.pic-download').attr('href', this.$options.filters.fullGoogleImage(picView))
    },
    totalPic() {
      return this.pictures.map(x=> x.pictureId.split(',').length).reduce((acc,curr) => {
        return acc + curr
      }, 0)
    }
  }
}
</script>
<style>
.product-photo {
  width: 100%;
  height: 100%;
  object-fit: cover;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
#loadingpos {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 1200;
}

#pictureBox {
  display: none;
}
</style>
