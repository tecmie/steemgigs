<template>
  <page :pageClasses="['new_steemgig__view', 'row']">
    <ul class="sections hide-on-med-and-down center">
      <li v-for="(section, index) in sections" :key="index"><a v-text="section" :class="{active: index === currentSection}" @click="switchTo(index)"></a></li>
    </ul>
    <div class="container" @keypress.tab="nextSection">
      <div class="col s12 m7 l9 row">
        <form class="card-panel row" v-if="currentSection === 0">
          <div class="container gigForm">
            <p class="flow-text title">Gig Title</p>
            <div class="input-field col s12">
              <span class="title-before">#STEEMGIGS: I will</span>
              <textarea @keypress.enter.prevent @keyup.enter="''" @input="search" v-model="newGigData.title" type="text" placeholder="do this (what you can do well) just the way they want it" row="2" maxlength="90" minlength="5" required>
              </textarea>
              <p class="word-count right" v-text="wordCount"></p>
              <div v-if="newGigData.title.length > 5" class="col s12 mb-2">
                <span class="simple-card">
                  <span v-if="duplicateTitle" class="green-text">You have already used this <router-link :to="`/@${$store.state.username}/${duplicateTitle.permlink}`" target="_blank">title</router-link>, you can still choose to proceed</span>
                  <span v-if="!duplicateTitle" class="green-text" v-text="validTitle" />
                </span>
              </div>
              <div class="tutorial_guide hide-on-small-only center-align">
                <div class="card">
                  <div class="card-content">
                    <span>Make your title short, describing exactly what service(s) you offer as this is the likeliest way to boost viewership, potential &amp; eventual sales</span>
                  </div>
                </div>
              </div>
            </div>
            <p class="flow-text title">STEEMGIGS Category</p>
            <div class="col s12 row">
              <div class="input-field col s12 m6 l4">
                <select class="browser-default my-select category_select" @change="refreshSubCategory" v-model="newGigData.category">
                  <option value="" disabled selected>Select Category</option>
                  <option v-for="(category, index) in categories" :key="index" :value="category.name" v-text="category.name"></option>
                </select>
              </div>
              <div class="input-field col s12 m6 l4" v-show="newGigData.category">
                <select class="my-select browser-default subCategory_select" v-model="newGigData.subcategory">
                  <option value="" disabled selected>Select Subcategory</option>
                  <option v-for="(subcategory, index) in categories[selectedCategoryIndex].subcategories" :key="index" :value="subcategory.name" v-text="subcategory.name"></option>
                </select>
              </div>
              <div class="col input-field s12">
                <input-tag limit="3" :read-only="true" :tags="defaultTags" />
                <input-tag limit="2" class="editable" placeholder="add tags" @update:tags="getTags" :tags="userTags" />
              </div>
              <div class="form-navs">
                <button @click.prevent="nextSection" class="right btn indigo waves-effect">Save and Proceed</button>
              </div>
              <div class="tutorial_guide guide-2 hide-on-small-only">
                <div class="card">
                  <div class="card-content">
                    <p>Select the most accurately describing category and subcategory</p>
                    <p class="mb-3">Choose the best follow up tags</p>
                    <span>"This will enhance the visibility of your gigs on our website&rsquo;s search engine and on the steem blockchain".</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </form>
        <form class="card-panel row" v-if="currentSection === 1">
          <div class="container gigForm">
            <p class="flow-text title">Describe your STEEMGIG</p>
            <div class="input-field col s12">
              <vue-editor v-model="newGigData.description" :upload="uploadConfig"></vue-editor>
              <div v-if="descError" class="col s12 my-3">
                <span class="simple-card">
                  <span class="red-text" v-text="descError" />
                </span>
              </div>
              <div class="tutorial_guide hide-on-small-only center-align">
                <div class="card">
                  <div class="card-content">
                    <span class="card-title">Describe your STEEMGIG</span>
                    <p>Be as expressive as possible as this is your chance to tell postential clients and steemians what you offer, why you gig is special and why they should give you opportunities etc.</p>
                    <p>Set up a viable list of potential FAQs and to try to answer each one.</p>
                  </div>
                </div>
              </div>
            </div>
            <div class="col s12 row">
                <button @click.prevent="prevSection" class="btn indigo accent-2 waves-effect">back</button>
                <button class="right btn indigo waves-effect" @click.prevent="descNext = true; nextSection()">Save and Proceed</button>
            </div>
          </div>
        </form>
        <form class="card-panel row" v-if="currentSection === 2">
          <div class="container gigForm">
            <p class="flow-text title">Pricing</p>
            <div class="input-field col s12">
              <vue-editor v-model="newGigData.pricing" :upload="uploadConfig"></vue-editor>
              <div v-if="pricingError" class="col s12 my-3">
                <span class="simple-card">
                  <span class="red-text" v-text="pricingError" />
                </span>
              </div>
              <div class="tutorial_guide hide-on-small-only center-align">
                <div class="card">
                  <div class="card-content">
                    <p class="mb-2">Give your PACKAGE an appealing name.
                    <p class="mb-2">Describe what will be contained in your package for each price.</p>
                    <p class="mb-2">Tell us if you are willing to do revisions. If so, "how many revisions?"; "will it cost extra?" etc</p>
                    <p class="mb-2">Can you render more "speed of delivery", if offered extra payment?</p>
                    <p>Are there aspects of your GIG that you are will to offering lovingly for free etc?</p>
                  </div>
                </div>
              </div>
            </div>
            <p class="flow-text title">Delivery</p>
            <div class="input-field col s12 m3 l3">
              <select class="browser-default my-select category_select" v-model="newGigData.hours">
                <option value="0" disabled selected>Less than an hour</option>
                <option v-for="(hour, index) in 24" :key="index" :value="hour">{{ hour }} hr(s)</option>
              </select>
            </div>
            <div class="input-field col s12 m3 l8">
              <select class="browser-default my-select category_select" v-model="newGigData.days">
                <option value="0">Less than a day</option>
                <option v-for="(day, index) in 30" :key="index" :value="day">{{ day }} day(s)</option>
              </select>
            </div>
            <div class="input-field col s12 m3 l3">
              <p class="flow-text title left col s12" style="margin-bottom: 1em">Price</p>
              <select class="browser-default my-select category_select" v-model="newGigData.currency">
                <option value="" disabled selected>currency</option>
                <option>STEEM</option>
                <option>SBD</option>
                <option>SP</option>
              </select>
            </div>
            <div class="input-field col s12 m3 l3 push-down">
              <input type="number" v-model="newGigData.price" class="price" placeholder="price">
            </div>
            <div class="col s12 row" style="margin-top: 3em;">
                <button @click.prevent="prevSection" class="btn indigo accent-2 waves-effect">back</button>
                <button class="right btn indigo waves-effect" @click.prevent="priceNext = true; nextSection()">Save and Proceed</button>
            </div>
          </div>
        </form>
        <form class="card-panel row" v-if="currentSection === 3">
          <div class="container gigForm">
            <p class="flow-text title">Requirements</p>
            <div class="input-field col s12">
              <vue-editor v-model="newGigData.requirements" :upload="uploadConfig"></vue-editor>
              <div v-if="requirementError" class="col s12 my-3">
                <span class="simple-card">
                  <span class="red-text" v-text="requirementError" />
                </span>
              </div>
              <div class="tutorial_guide hide-on-small-only center-align">
                <div class="card">
                  <div class="card-content">
                    <span class="card-title"></span>
                    <p class="mb-2">Explain what you would want your clients to provide you with, to help you start the GIG. e.g if your GIG is SEO-related, you may want them to drop "the URL of their website; If your GIG is "graphics-related", you may request images, GIFs etc to guide you during your creation; If your GIG is "video-editing", you may require them to drop you "video files" etc.</p>
                    <p>Tell them if your "requirements" are optional or compulsory.</p>
                  </div>
                </div>
              </div>
            </div>
            <div class="col s12 row">
                <button @click.prevent="prevSection" class="btn indigo accent-2 waves-effect">back</button>
                <button class="right btn indigo waves-effect" @click.prevent="reqNext = true; nextSection()">Save and Proceed</button>
            </div>
          </div>
        </form>
        <form class="card-panel row" v-show="currentSection === 4">
          <dismissible-notice>
            <span>Please ensure all images are yours or give credit to source (URL) where required.<br>Remember that your posts appear on the steem blochain and are subject to curation. Please read style guide on right.</span>
          </dismissible-notice>
          <div class="container gigForm">
            <p class="flow-text title">Portfolio</p>
            <div class="input-field col s12 row">
              <div class="col s12 m4 l3 mb-3" v-for="(image, index) in newGigData.portfolio" :key="image.key">
                <img-upload :id="index" :img="image.url" />
              </div>
              <div class="col s12 m4 l3 mb-3" v-if="newGigData.portfolio.length < 8">
                <div class="add-box center center-align pt-5" @click.prevent="morePics">
                  <button class="btn-floating indigo">
                    <i class="icon ion-android-add"></i>
                  </button><br><br>
                  <span>Click to add more images</span>
                </div>
              </div>
              <div class="tutorial_guide hide-on-small-only center-align">
                <div class="card">
                  <div class="card-content">
                    <span class="card-title"></span>
                    <p class="mb-2">Attach images, videos, GIFs etc showcasing your past work. Add any additional media that describes your SteemGig or tells potential client what they can do to help you serve them better.</p>
                    <p class="mb-3">Consider attaching a video telling potential clients as well as steemians, why your service is outstanding.</p>
                    <span class="ml-1">Note: The first uploaded media is what will appear in the thumbnail of your SteemGig both here and on the steem blockchain.</span>
                  </div>
                </div>
              </div>
            </div>
            <p class="flow-text title">Reward</p>
            <div class="input-field col s12 m3 l3">
              <select class="browser-default my-select category_select" v-model="newGigData.reward">
                <option>100% STEEM POWER</option>
                <option>50% SBD 50% SP</option>
                <option>Declined</option>
              </select>
            </div>
            <div class="input-field col s12 m6 row">
              <div class="col s5">
                <p>
                  <label>
                    <input type="checkbox" :checked="newGigData.liked ? 'checked' : ''" v-model="newGigData.liked" />
                    <span>Like your post</span>
                  </label>
                </p>
              </div>
              <div class="col s7 mt-4">
                <slider-range v-if="newGigData.liked" :min="1" v-model="newGigData.upvoteRange" />
              </div>
            </div>
            <div v-if="portfolioError" class="col s12 my-3">
              <span class="simple-card">
                <span class="red-text" v-text="portfolioError" />
              </span>
            </div>
            <div class="col s12 row">
                <button @click.prevent="prevSection" class="btn indigo accent-2 waves-effect">back</button>
                <button class="right btn indigo waves-effect" @click.prevent="nextSection">Save and Proceed</button>
            </div>
          </div>
        </form>
        <div class="row" v-if="currentSection === 5">
          <div class="col s12 preview">
            <div class="card">
              <div class="card-content">
                <span class="card-title">#STEEMGIGS: I will {{ newGigData.title }}</span>
                <p><router-link :to="'/categories/' + this.newGigData.category">{{ this.newGigData.category }}</router-link> / <router-link :to="'/categories/' + this.newGigData.category + '/' + this.newGigData.subcategory">{{ this.newGigData.subcategory }}</router-link></p>
              </div>
              <div class="card-image">
                <carousel :navigationEnabled="false" :autoplay="true" :autoplayHoverPause="true" :perPage="1">
                  <slide v-for="(image, index) in portfolio" :key="index">
                    <img :src="image" class="responsive-img" :alt="newGigData.title">
                  </slide>
                </carousel>
              </div>
              <div class="card-content">
                <vue-markdown :source="previewData" />
              </div>
            </div>
            <div class="tutorial_guide hide-on-small-only">
              <div class="card">
                <div class="card-content">
                  <span class="card-title">How Nice?</span>
                  <p class="mt-1">Take a look at your Steemgigs Post to see if you have made errors</p>
                  <p class="mb-3">If error free, hit "publish", else, correct errors</p>
                  <span>Note: Your post will also appear on the Steem Blockchain</span>
                </div>
              </div>
            </div>
            <div v-if="errorr" class="simple-card card-panel">
              <p v-if="!validTitle" class="red-text mt-1 mb-0" >Title must be at least 5 characters</p>
              <p v-if="descError" class="red-text mt-1 mb-0" v-text="descError" />
              <p v-if="pricingError" class="red-text mt-1 mb-0" v-text="pricingError" />
              <p v-if="requirementError" class="red-text mt-1 mb-0" v-text="requirementError" />
              <p v-if="portfolioError" class="red-text mt-1 mb-0" v-text="portfolioError" />
              <p v-if="subcatError" class="red-text mt-1 mb-0" v-text="subcatError" />
            </div>

          </div>
          <div class="col s12 row">
            <button @click.prevent="prevSection" class="btn indigo accent-2 waves-effect">back</button>
            <button :disabled="errorr" class="right btn indigo waves-effect" @click.prevent="submit">
              <i class="fa fa-spinner fa-pulse" v-if="isPosting"></i>
              POST #STEEMGIG
            </button>
            <p class="red-text" v-if="errorText">Error: {{ errorText }}</p>
            <p class="indigo-text" v-if="successText">{{ successText }}</p>
          </div>
        </div>
      </div>
    </div>
  </page>
</template>

<script>
import Api from '@/services/api'
import Page from '@/components/page'
import CatNav from '@/components/layout/catNav'
import ImgUpload from '@/components/snippets/imgUpload'
import { MarkdownEditor } from 'markdown-it-editor'
import VueMarkdown from 'vue-markdown'
import { VueEditor } from 'vue2-editor'
import { Carousel, Slide } from 'vue-carousel'
import DismissibleNotice from '@/components/snippets/dismissibleNotice'
import InputTag from 'vue-input-tag'
import SliderRange from 'vue-slider-component'
import debounce from '@/plugins/debounce'
import Util from '@/services/util'

export default {
  components: {
    Page,
    CatNav,
    MarkdownEditor,
    VueMarkdown,
    Carousel,
    Slide,
    InputTag,
    ImgUpload,
    VueEditor,
    SliderRange,
    DismissibleNotice
  },
  data () {
    return {
      successText: '',
      errorText: '',
      isPosting: false,
      sections: ['Overview', 'Description', 'Pricing', 'Requirements', 'Portfolio', 'Publish'],
      currentSection: 0,
      descNext: false,
      reqNext: false,
      priceNext: false,
      portNext: false,
      userTags: [],
      totalPics: 1,
      existingTitle: false,
      checkingTitle: false,
      duplicateTitle: '',
      newGigData: this.$store.state.newPosts.steemgig,
      customToolbar: [
        ['bold', 'italic', 'underline'],
        [{'list': 'ordered'}, {'list': 'bullet'}],
        ['image', 'code-block']
      ],
      uploadConfig: {
        name: 'file',
        accept: 'image/jpg,image/jpeg,image/png',
        url: this.$imgUploadURL
      }
    }
  },
  methods: {
    vote () {
      if (!this.newGigData.liked) {
        this.newGigData.liked = true
      } else {
        this.newGigData.liked = false
      }
    },
    switchTo (index) {
      if (index === 5) {
        this.descNext = this.priceNext = this.reqNext = this.portNext = true
      } else if (this.currentSection >= 3) {
        this.reqNext = true
      } else if (this.currentSection >= 2) {
        this.priceNext = true
      } else if (this.currentSection >= 1) {
        this.descNext = true
      }
      this.currentSection = index
    },
    nextSection () {
      if (this.currentSection < this.sections.length) this.currentSection++
    },
    prevSection () {
      if (this.currentSection > 0) this.currentSection--
    },
    refreshSubCategory () {
      this.newGigData.subcategory = ''
    },
    getTags (entries) {
      this.userTags = entries
    },
    morePics () {
      if (this.newGigData.portfolio.length < 8) {
        this.newGigData.portfolio.push({url: '', key: Math.floor(Math.random() * 1000)})
      }
    },
    search: debounce(function () {
      this.checkingTitle = true
      let searchTerm = this.steemedTitle
      console.log('search term:', searchTerm)
      Api.checkTitleExistence({username: this.$store.state.username, title: this.steemedTitle}).then(result => {
        this.checkingTitle = false
        this.duplicateTitle = result.data
        console.log(result)
      }).catch(e => {
        this.checkingTitle = false
        this.errorText = 'there was an error with search'
        console.log('error:', e)
      })
    }, 1000),
    closeSearch (cb) {
      this.searchTerm = ''
      this.searchResults = []
      this.searchActive = false
      cb()
    },
    goto (where) {
      this.closeSearch(this.$router.push(where))
      // this.$router.push(where)
    },
    submit () {
      if (!this.errorr) {
        if (this.isPosting) return
        let that = this
        this.errorText = ''
        this.successText = ''
        this.isPosting = true
        let jsonMetadata = {
          app: 'steemgig',
          users: ['steemgigs', this.$store.state.username],
          tags: [...this.userTags, ...this.defaultTags],
          format: 'Markdown',
          timestamp: new Date().getTime(),
          price: this.newGigData.price,
          currency: this.newGigData.currency,
          authorPic: this.$store.state.profile.profileImage,
          category: this.slugify(this.newGigData.category),
          subcategory: this.slugify(this.newGigData.subcategory),
          images: this.portfolio,
          type: 'steemgigs_post',
          generated: true
        }
        let textifiedPics = '\n<h2>Portfolio</h2>\n<hr />\n'
        this.portfolio.forEach(url => {
          textifiedPics += `<img src="${url}"> <br />`
        })
        let username = this.$store.state.username
        let permlink = this.slugify(this.newGigData.title)
        let steemGigsTag = `
  <i>this post was made on <a href="https://steemgigs.org/@${username}/${permlink}">STEEMGIGS Where everyone has something to offer</a></i>
        `
        let contentToHide = textifiedPics + steemGigsTag
        let hiddenContainer = this.htmlHide(contentToHide)
        let body = this.previewData + hiddenContainer

        const imagesFromBody = Util.getImagesFromBody(this.previewData)
        if (imagesFromBody.length) {
          jsonMetadata.images = jsonMetadata.images.concat(imagesFromBody)
        }

        let token = this.$store.state.accessToken
        let title = this.steemedTitle
        // if (this.duplicateTitle) {
        //   let splittedTitle = title.split(' ')
        //   let lastNumber = parseInt(splittedTitle[splittedTitle.length - 1])
        //   title = lastNumber ? title + ' ' + lastNumber++ : title + ' 1'
        // })
        if (this.duplicateTitle) {
          let modifiedTitle = this.newGigData.title + Math.floor(Math.random() * 1000)
          permlink = this.slugify(modifiedTitle)
          title = '#STEEMGIGS: I will ' + modifiedTitle
        }
        let liked = this.newGigData.liked
        let upvoteRange = this.newGigData.upvoteRange
        // username, permlink, title, body, jsonMetadata, token
        Api.post({username, permlink, title, body, liked, upvoteRange, jsonMetadata}, token).then((err, res) => {
          console.log('err', err, 'res', res)
          that.isPosting = false
          this.$notify({
            group: 'foo',
            title: 'Success',
            text: 'Successfully pushed to steem!',
            type: 'success'
          })
          that.successText = 'Successfully pushed to steem!'
          that.$store.commit('RESET_NEW_STEEMGIG')
        }).catch((e) => {
          console.log(e)
          that.isPosting = false
          this.$notify({
            group: 'foo',
            title: 'Error',
            text: 'Error pushing post to steem',
            type: 'error'
          })
          that.errorText = 'Error pushing post to steem'
        })
      }
    }
  },
  computed: {
    validTitle () {
      if (this.newGigData.title.length > 5 && this.checkingTitle) {
        return 'Wait a sec...'
      } else if (this.newGigData.title.length > 5) {
        return 'Title is valid, your\'re cool!'
      } else {
        return ''
      }
    },
    steemedTitle () {
      return '#STEEMGIGS: I will ' + this.newGigData.title
    },
    portfolio () {
      return this.newGigData.portfolio.filter(image => image.url).map(image => image.url)
    },
    descError () {
      if (this.descNext && this.newGigData.description.length < 300) {
        return 'Your description should be 300 Characters or more, please read style guide for clarification'
      } else {
        return ''
      }
    },
    pricingError () {
      if (this.priceNext && this.newGigData.pricing.length < 100) {
        return 'Your pricing description should be 100 Characters or more, please read style guide for clarification'
      } else {
        return ''
      }
    },
    requirementError () {
      if (this.reqNext && this.newGigData.requirements.length < 100) {
        return 'Your requirments description should be 100 Characters or more, please read style guide for clarification'
      } else {
        return ''
      }
    },
    subcatError () {
      if (this.descNext && !this.newGigData.subcategory) {
        return 'You must select a category/subcategory'
      } else {
        return ''
      }
    },
    portfolioError () {
      if (this.portNext && this.newGigData.portfolio.length < 1) {
        return 'You must add at least one image to your portfolio'
      } else {
        return ''
      }
    },
    errorr () {
      return Boolean(this.descError || this.requirementError || this.pricingError || this.subcatError || this.portfolioError || !this.validTitle)
    },
    selectedCategoryIndex () {
      let catIndex = 0
      this.categories.forEach((category, index) => {
        if (category.name === this.newGigData.category) catIndex = index
      })
      return catIndex
    },
    defaultTags () {
      return ['steemgigs', this.slugify(this.newGigData.category), this.slugify(this.newGigData.subcategory)]
    },
    wordCount () {
      if (this.newGigData.title.length > 0) {
        return `${this.newGigData.title.length} / 90 Characters`
      } else {
        return `90 Characters`
      }
    },
    previewData () {
      return `
<h2 class="headline">Description</h2>
<hr />
${Util.convertImageUrlToHTML(this.newGigData.description)}
<h2 class="headline">Pricing</h2>
<hr />
${Util.convertImageUrlToHTML(this.newGigData.pricing)}

<h5>Price: Starting at ${this.newGigData.price} ${this.newGigData.currency}</h5>
<h5>Delivery: ${this.newGigData.days} day(s) ${this.newGigData.hours} hour(s)</h5>
<hr />
<h2 class="headline">Requirements</h2>
<hr />
${Util.convertImageUrlToHTML(this.newGigData.requirements)}
      `
    }
  },
  watch: {
    newGigData: {
      handler (val) { this.$store.commit('SET_NEW_STEEMGIG', val) },
      deep: true
    }
  },
  mounted () {
    this.$eventBus.$on('img-uploaded', payload => {
      console.log(payload)
      this.newGigData.portfolio[payload.index].url = payload.url
    })
    this.$eventBus.$on('delete-image-url', payload => {
      this.newGigData.portfolio.splice(payload, 1)
    })
  },
  deforeDestroy () {
    this.$eventBus.$off('img-uploaded')
    this.$eventBus.$off('delete-image-url')
  }
}
</script>

<style lang="scss" scoped>
form .input-field {
  position: relative;
}
.add-box {
  box-shadow: 0 1px 1px;
  background: #f9f9f9;
  color: dimgray;
  padding: 10px 10px;
  height: 200px; /* minimum height */
  position: relative;
  cursor: pointer;
  span {
    font-size: .9em;
  }
}
select.my-select {
  width: initial !important;
  pointer-events: initial !important;
  height: 2.5em !important;
  min-width: 10em;
  position: static;
  opacity: 1 !important;
}
.sections {
  background: white;
  border-bottom: 1px solid #ccc;
  border-top: 1px solid #f8f8f8;
  top: 41px;
  position: fixed;
  width: 100%;
  z-index: 2;
  li {
    &:not(:first-child) {
      &>a::before {
        font-family: "Ionicons";
        content: "\f125";
        position: absolute;
        left: -0.5em;
      }
    }
    display: inline-block;
    position: relative;
    &:first-child>a::before {
      content: ''
    }
    a {
      padding: 15.5px 1em;
      line-height: 50px;
      font-weight: 500;
      color: #757575;
      cursor: pointer;
      margin-left: 1em;
      position: relative;
      box-sizing: border-box;
      transition: all ease-in-out .3s;
      &.active, &:hover {
        color: black;
        font-weight: 500;
      }
      &::after, &.active::after {
        content: ' ';
        height: 2px;
        width: 0%;
        background: black;
        display: inline-block;
        position: absolute;
        left: 0;
        bottom: 0;
        transition: all ease-in-out .3s;
      }
      &:hover::after, &.active::after {
        width: 100%;
      }
      &.active::after {
        height: 1px;
      }
      &:hover::after {
        height: 2px;
      }
    }
  }
}
.container {
  min-width: 95%;
  padding-top: 1.5em;
  &>.col>.card {
    // padding: 1em;
    padding-bottom: 40px;
  }
}
// .input-field>label:not(.label-icon).active {
//     -webkit-transform: translateY(-14px) scale(0.8);
//     transform: translateY(-14px) translateX(-38px) scale(1);
//     -webkit-transform-origin: 0 0;
//     transform-origin: 0 0;
//     color: black;
// }
.input-field {
  margin-bottom: 1em;
}
p.title {
  margin-left: 0.5em;
  margin-bottom: 0;
  margin-top: 0
}
.select-wrapper {
  position: relative;
  outline: 0px solid;
  border: 1px solid #BDBDBD;
  margin-left: -0.5em;
  padding-left: 5px;
  input.select-dropdown {
    margin-bottom: 0;
  }
}
.gigForm {
  position: relative;
  .title-before {
    position: absolute;
    color: #757575;
    top: 0.9px;
    left: 0.6em;
    font-size: 28px;
  }
  textarea {
    font-family: 'Source Sans Pro', Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    line-height: 41.5px;
    text-indent: 7.8em;
    font-size: 28px;
    padding: 0px 6px;
    min-height: 4.7em;
    resize: none;
    color: #757575;
    overflow-y: auto;
    border: 2px solid #bdbdbd5e;
    box-sizing: border-box;
    border-radius: 4px;
    &:focus {
      border: 2px solid #9FA8DA;
      outline: 0px solid;
    }
  }
}
.word-count {
  margin-top: 0;
}
.form-navs {
  display: block;
  margin-top: 5em;
  button {
    line-height: 3px;
    text-transform: initial;
    font-weight: 500;
  }
}
.tutorial_guide {
  position: absolute;
  right: -28vw;
  width: 23.5vw;
  top: 0em;
  &::before {
    content: ' ';
    width: 0;
    height: 0;
    border-top: 10px solid transparent;
    border-bottom: 10px solid transparent;
    border-right: 10px solid #FFFFFC;
    position: absolute;
    left: -10px;
    top: 20%;
    z-index: 3;
    box-shadow: 0px 0px 0px black;
  }
  .card-content {
    p {
      display: list-item;
      margin-left: 1em;
    }
  }
}
.push-down {
  margin-top: 4.2em;
}
</style>
