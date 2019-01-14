<template>
  <div class="container">
    <form v-on:submit.prevent="onSubmit">
      <input
        type="text" 
        v-model="query" 
        @keyup="onKeyup" 
        placeholder="검색어를 입력하세요" 
        autofocus
      >
      <button 
        @click.prevent="onReset"
        v-show="query.length" 
        type="reset"
        class="btn-reset"
      >
      </button>

      <div v-if="submitted">
        <div v-if="searchResult.length">
          <ul>
            <li 
              v-for="(item, index) in searchResult" 
              :key="index"
            >
              <img 
                v-bind:src="item.image" 
                alt=""
              >
              {{item.name}}
            </li>
          </ul>
        </div>
        <div v-else>
          {{query}} 검색어로 찾을 수 없습니다.
        </div>
      </div>
      <div v-else>
        <ul class="tabs">
          <li
            v-for="(tab, index) in tabs" 
            :key="index"
            v-bind:class="{active: tab === selectedTab}"
            v-on:click="onClickTab(tab)"
          >
            {{tab}}
          </li>
        </ul>
        <div v-if="selectedTab === tabs[0]">
          <div v-if="keywords.length">
            <ul class="list">
              <li 
                v-for="(item, index) in keywords" 
                :key="index"
                @click="onClickKeyword(item.keyword)"
              >
                <span class="number">{{index + 1}}</span>
                {{item.keyword}}
              </li>
            </ul>
          </div>
          <div v-else>
            추천 검색어가 없습니다. 
          </div>
        </div>
        <div v-else>
          <div v-if="history.length">
            <ul class="list">
              <li 
                v-for="(item, index) in history" 
                :key="index"
                v-on:click="onClickKeyword(item.keyword)"
              >
                {{item.keyword}}
                <span class="date">{{item.date}}</span>
                <button 
                  class="btn-remove"
                  @click.stop="onClickRemoveHistory(item.keyword)"
                ></button>
              </li>
            </ul>

          </div>

          <div v-else>
            최근 검색어가 없습니다.
          </div>
        </div>
      </div>

    </form>
  </div>
</template>


<script>



import SearchModel from '../models/SearchModel.js'
import KeywordModel from '../models/KeywordModel.js'
import HistoryModel from '../models/HistoryModel.js'


export default {
  data() {
    return {
      query: '',
      submitted: false,
      searchResult: [],
      selectedTab: '',
      history: [],
      tabs: ['추천 검색어', '최근 검색어'],
      keywords:[]
    }
  },

  created() {
    this.selectedTab = this.tabs[0]
    this.fetchKeyword()
    this.fetchHistory()
  },

  methods: {
    onSubmit(e) {
      this.search()
    },

    onKeyup() {
      if ( !this.query.length ) this.resetForm()
    },

    onReset(e) {
      this.resetForm()
    },

    onClickTab(tabName) {
      this.selectedTab = tabName
    },

    onClickKeyword(keyword) {
      this.query = keyword
      this.search() 
    },

    onClickRemoveHistory(keyword) {
      HistoryModel.remove(keyword)
      this.fetchHistory()
    },

    search() {
      SearchModel.list().then(data => {
        this.submitted = true
        this.searchResult = data
      })
      HistoryModel.add(this.query)
      this.fetchHistory()
    },

    resetForm() {
      this.query = ''
      this.submitted = false
      this.searchResult = []
    },
    
    fetchKeyword() {
      KeywordModel.list().then(data => {
        this.keywords = data
      })
    },
    fetchHistory() {
      HistoryModel.list().then(data => {
        this.history = data
      })
    }
  }
}
</script>