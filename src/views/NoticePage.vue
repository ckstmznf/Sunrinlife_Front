<template>
    <div class="panel page notice-page">
        <div class="page-content">
            <div
                class="notice-content"
                :class="{ 'neu-morphism-card': !$store.state.isMobileWindow }"
            >
                <div class="header">
                    <h3>공지</h3>
                    <div class="search-wrap">
                        <input
                            v-model="searchQueryText"
                            type="text"
                            placeholder="검색"
                            @keyup.enter="search()"
                        />
                        <img
                            src="/img/search.svg"
                            alt="검색 아이콘"
                            class="search-button"
                            @click="search()"
                        />
                    </div>
                </div>

                <ul class="notice-list">
                    <li
                        v-for="(i, n) in loadedNoticeData"
                        :key="n"
                        @click="$router.push({ name: 'noticeId', params: { noticeId: i.id } })"
                    >
                        <NoticeIcon :type="i.type" />

                        <p class="notice-title text-truncated">{{ i.title }}</p>
                    </li>
                </ul>
                <Pagination v-bind:page-count="loadedPageCount" />
            </div>
        </div>
    </div>
</template>

<script>
import NoticeIcon from "../components/NoticeIcon.vue";

import { mapState } from "vuex";

import {
    getNotice,
    getNoticeSearch,
    getNoticePageCountWithSearch,
    getNoticePageCount
} from "../api.js";
import store from "../store.js";
import Pagination from "../components/Pagination.vue";

export default {
    name: "Notice",
    data() {
        return {
            loadedPageCount: null,
            loadedNoticeData: {},
            searchQueryText: "",
        };
    },
    components: {
        NoticeIcon,
        Pagination
    },
    computed: {
        ...mapState(["noticePageCount", "noticePage", "notice"]),
        getAuthToken() {
            return this.$store.getters.getAuthToken;
        },
        pageId: function () {
            return parseInt(this.$route.query.page) || 1;
        },
        searchQuery: function () {
            return this.$route.query.search;
        },
        savedPages: function () {
            return Object.keys(this.$store.getters.getNoticePage || {})
                .map((x) => parseInt(x))
                .filter((x) => x);
        },
    },
    watch: {
        getAuthToken(){
            getNoticePageCount().then(res=>store.commit("setNoticePageCount", res));
        },
        pageId: function () {
            this.loadNotice();
        },
        noticePageCount: function() {
            this.loadedPageCount = this.$store.getters.getNoticePageCount;
        },
    },
    methods: {
        loadNotice: function () {
            if (!this.searchQuery && this.savedPages.includes(this.pageId)) {
                this.loadedNoticeData = this.noticePage[this.pageId].map(
                    (x) => this.notice[x]
                );
            } else {
                this.getNoticeTask().then((res) => {
                    if (!this.searchQuery)
                        store.commit("getNoticePage", {
                            page: this.pageId,
                            ids: res.map((x) => x.id),
                        });
                    for (let data of res) store.commit("getNotice", data);
                    this.loadedNoticeData = res;
                });
            }
        },
        getNoticeTask: function () {
            if (this.searchQuery)
                return getNoticeSearch(this.pageId, this.searchQuery);
            return getNotice(this.pageId);
        },
        search: async function () {
            await this.$router.push({
                path: "notice",
                query: { search: this.searchQueryText },
            });
            await this.updateCount();
            this.loadNotice();
        },
        updateCount: async function () {
            if (this.searchQuery)
                this.loadedPageCount = await getNoticePageCountWithSearch(this.searchQuery);
            else this.loadedPageCount = this.$store.getters.getNoticePageCount
        },
    },
    mounted() {
        this.loadNotice();
        this.updateCount();
        if(this.$store.getters.getNoticePageCount === null) getNoticePageCount().then(res=>store.commit("setNoticePageCount", res));
        if(this.$route.query.search){
            this.searchQueryText = this.$route.query.search
        }
    },
};
</script>

<style scoped>
.page-content {
    
}

.notice-content {
    min-height: 660px;

    padding: 16px;
    position: relative;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    gap: 20px;
    height: 620px;
}

.header {
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.header h3 {
    text-align: center;
    display: flex;
    flex: 1 1 auto;
    margin-right: 12px;
}

.search-wrap {
    height: 40px;
    max-width: 323px;
    flex: 1;

    padding: 8px 16px;
    border-radius: 8px;
    background-color: #f5f6f7;

    display: flex;
    gap: 8px;

    position: relative;
    align-items: center;
}

.search-wrap input {
    height: 100%;
    width: 100%;
    flex: 1;

    font-size: 16px;
    font-weight: 500;

    background-color: transparent;

    border: 0px;
    border-radius: 0px;
}

.search-wrap input::placeholder {
    color: #b9b9b9;
}

.search-wrap .search-button {
    width: 20px;
    height: 20px;

    cursor: pointer;
}

.notice-list {
    flex: 1;

    padding: 4px 24px;

    display: flex;
    flex-direction: column;
    gap: 24px;
}

.notice-list li {
    display: flex;
    gap: 12px;

    cursor: pointer;
}

.notice-list .notice-icon {
    margin: 0px;
}

.notice-list .notice-title {
    color: #3d3d3d;

    font-size: 16px;
    font-weight: 500;
    margin-top: 1.5px;

    flex: 1;
}

.notice-list .writer {
    color: #b9b9b9;

    font-size: 12px;
    font-weight: 500;
}

.pagination-wrap {
    height: 32px;

    /* margin-top : 32px; */

    display: flex;
    justify-content: center;
}

.pagination-wrap img {
    width: 32px;

    padding: 8px;
}

.pagination-wrap .page-button-wrap {
    display: flex;
    gap: 4px;
}

.pagination-wrap .page-btn {
    width: 32px;

    color: #c9c9c9;


    font-size: 14px;
    font-weight: 500;

    border-radius: 50%;

    display: flex;
    align-items: center;
    justify-content: center;

    user-select: none;
}

.pagination-wrap .page-btn:hover {
    background-color: var(--gray1);
}

.pagination-wrap .page-btn.current-page {
    color: white;

    background-color: var(--main-color4);
}

.pagination-wrap * {
    cursor: pointer;
}

@media (max-width: 970px) {
    .header h3 {
        display: none;
    }

    .search-wrap {
        max-width: none;

        background-color: #fff;
    }
}
</style>
