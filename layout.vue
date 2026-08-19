<template>
    <div class="vector" :style="{ '--logo-url': logoUrl }">
        <div id="mw-page-base" class="noprint"></div>
        <div id="mw-head-base" class="noprint"></div>
        <div id="content" class="mw-body" role="main">
            <a id="top"></a>
            <div v-if="$store.state.config['wiki.sitenotice']" id="siteNotice" class="mw-body-content" v-html="$store.state.config['wiki.sitenotice']"></div>
            <div class="mw-indicators mw-body-content"></div>
            <h1 id="firstHeading" class="firstHeading" lang="ko">{{ $store.state.page.title }}</h1>
            <div id="bodyContent" class="mw-body-content">
                <div id="siteSub" class="noprint"></div>
                <div id="contentSub">
                <div v-if="$store.state.page.viewName === 'wiki' && $store.state.page.data.rev" class="mw-revision">
                    <div id="mw-revision-info"><local-date :date="$store.state.page.data.date" />에 작성된 r{{ $store.state.page.data.rev }} 판</div>
                    <!-- <div id="mv-revision-nav"></div> -->
                </div>
                </div>
                <div v-if="$store.state.session.user_document_discuss" class="usermessage theseed-user-document-discuss-alert" :data-id="$store.state.session.account.name + '-' + $store.state.session.user_document_discuss">현재 진행중인
                    <nuxt-link :to="doc_action_link(user_doc($store.state.session.account.name), 'discuss')">사용자토론</nuxt-link>이 있습니다.
                </div>
                <div id="mw-content-text" class="mw-content-ltr wiki-article" dir="ltr">
                    <nuxt />
                </div>
                <div class="visualClear"></div>
            </div>
        </div>
        <div id="mw-navigation">
            <h2>둘러보기 메뉴</h2>
            <div id="mw-head">
                <div id="p-personal" role="navigation" aria-labelledby="p-personal-label">
                    <h3 id="p-personal-label">개인 도구</h3>
                    <ul v-if="$store.state.session.account.type === 1">
                        <li id="pt-userpage">
                            <nuxt-link :to="doc_action_link(user_doc($store.state.session.account.name), 'w')" title="내 사용자 문서 [Alt+Shift+.]" accesskey=".">{{ $store.state.session.account.name }}</nuxt-link>
                        </li>
                        <li id="pt-mytalk">
                            <nuxt-link :to="doc_action_link(user_doc($store.state.session.account.name), 'discuss')" title="내 토론 [Alt+Shift+n]" accesskey="n">토론</nuxt-link>
                        </li>
                        <li id="pt-preferences">
                            <nuxt-link to="/member/mypage" title="계정설정">설정</nuxt-link>
                        </li>
                        <li id="pt-mycontris">
                            <nuxt-link :to="contribution_link($store.state.session.account.uuid)" title="내 기여 목록 [Alt+Shift+y]" accesskey="y">기여</nuxt-link>
                        </li>
                        <li id="pt-watchlist">
                            <nuxt-link to="/member/starred_documents" title="주시문서 목록 [Alt+Shift+l]" accesskey="l">주시문서 목록</nuxt-link>
                        </li>
                        <li id="pt-logout">
                            <nuxt-link :to="{path:'/member/logout',query:{redirect:$route.fullPath}}" title="로그아웃">로그아웃</nuxt-link>
                        </li>
                    </ul>
                    <ul v-else>
                        <li id="pt-anonuserpage">로그인하지 않음</li>
                        <li v-if="$store.state.session.account.uuid" id="pt-anoncontribs">
                            <nuxt-link :to="contribution_link($store.state.session.account.uuid)" title="이 IP 주소의 편집 목록 [Alt+Shift+y]" accesskey="y">기여</nuxt-link>
                        </li>
                        <li id="pt-createaccount">
                            <nuxt-link to="/member/signup" title="계정을 만들고 로그인하는 것이 좋습니다; 하지만, 필수는 아닙니다">계정 만들기</nuxt-link>
                        </li>
                        <li id="pt-login">
                            <nuxt-link :to="{path:'/member/login',query:{redirect:$route.fullPath}}" :title="$store.state.config['wiki.site_name'] + '에 로그인하면 여러가지 편리한 기능을 사용할 수 있습니다. [Alt+Shift+o]'" accesskey="y">로그인</nuxt-link>
                        </li>
                    </ul>
                </div>
                <div id="left-navigation">
                    <div id="p-namespaces" class="vectorTabs" role="navigation" aria-labelledby="p-personal-label">
                        <h3 id="p-namespaces-label">이름공간</h3>
                        <ul v-if="$store.state.page.data.document">
                            <li :class="{ 'selected': $store.state.page.viewName === 'wiki' }">
                                <span>
                                    <nuxt-link :to="doc_action_link($store.state.page.data.document, 'w')">문서</nuxt-link>
                                </span>
                            </li>
                            <li :class="{ 'selected': $store.state.page.viewName.startsWith('thread') }">
                                <span>
                                    <nuxt-link :to="doc_action_link($store.state.page.data.document, 'discuss')">토론</nuxt-link>
                                </span>
                            </li>
                            <li v-if="$store.state.page.data.user">
                                <span>
                                    <nuxt-link :to="contribution_link($store.state.page.data.user.uuid)">기여내역</nuxt-link>
                                </span>
                            </li>
                        </ul>
                        <ul v-else>
                            <li>
                                <span>
                                    <nuxt-link :to="$route.fullPath">특수문서</nuxt-link>
                                </span>
                            </li>
                        </ul>
                    </div>
                </div>
                <div id="right-navigation">
                    <template v-if="$store.state.page.data.document">
                        <div id="p-views" role="navigation" class="vectorTabs" aria-labelledby="p-views-label">
                            <h3 id="p-views-label">보기</h3>
                            <ul>
                                <li id="ca-view" class="collapsible" :class="{ selected: $store.state.page.viewName === 'wiki' }">
                                    <span>
                                        <nuxt-link :to="doc_action_link($store.state.page.data.document, 'w')">읽기</nuxt-link>
                                    </span>
                                </li>
                                <li id="ca-edit" class="collapsible" :class="{ selected: $store.state.page.viewName === 'edit' }">
                                    <span>
                                        <nuxt-link :to="doc_action_link($store.state.page.data.document, 'edit')" title="이 문서 편집하기 [Alt+Shift+e]" accesskey="e">편집</nuxt-link>
                                    </span>
                                </li>
                                <li id="ca-history" class="collapsible" :class="{ selected: $store.state.page.viewName === 'history' }">
                                    <span>
                                        <nuxt-link :to="doc_action_link($store.state.page.data.document, 'history')" title="이 문서의 과거 편집 내역입니다. [Alt+Shift+h]" accesskey="h">역사 보기</nuxt-link>
                                    </span>
                                </li>
                                <!-- 원본은 주시문서 설정시 빙글빙글 돌아가는 등의 애니메이션이 있지만 귀찮아서 구현 안함. -->
                                <li v-if="$store.state.page.data.star_count >= 0" :id="$store.state.page.data.starred ? 'ca-unwatch' : 'ca-watch'" class="collapsible icon mw-watchlink">
                                    <span>
                                        <nuxt-link :to="$store.state.page.data.starred ? doc_action_link($store.state.page.data.document, 'member/unstar') : doc_action_link($store.state.page.data.document, 'member/star')" data-mw="interface" :title="'이 문서를 주시문서 목록에 추가 [Alt+Shift+w] (' + $store.state.page.data.star_count + '명이 주시중)'" accesskey="w">주시{{ $store.state.page.data.starred ? ' 해제' : '' }}</nuxt-link>
                                    </span>
                                </li>
                                <span class="placeholder" style="display: none;"></span>
                                <span class="placeholder" style="display: none;"></span>
                            </ul>
                        </div>
                        <div id="p-cactions" role="navigation" class="vectorMenu" aria-labelledby="p-cactions-label">
                            <h3 id="p-cactions-label" tabindex="0" style="width: 74px;">
                                <span>더 보기</span>
                            </h3>
                            <div class="menu">
                                <ul>
                                    <li class="collapsible" style="display: list-item;">
                                        <span>
                                            <nuxt-link :to="doc_action_link($store.state.page.data.document, 'delete')" title="이 문서를 삭제합니다.">삭제</nuxt-link>
                                        </span>
                                    </li>
                                    <li class="collapsible" style="display: list-item;">
                                        <span>
                                            <nuxt-link :to="doc_action_link($store.state.page.data.document, 'move')" title="이 문서를 이동합니다.">이동</nuxt-link>
                                        </span>
                                    </li>
                                    <li class="collapsible" style="display: list-item;">
                                        <span>
                                            <nuxt-link :to="doc_action_link($store.state.page.data.document, 'acl')" title="ACL을 봅니다.">ACL</nuxt-link>
                                        </span>
                                    </li>
                                </ul>
                            </div>
                        </div>
                    </template>
                    <div id="p-search" role="search">
                        <h3>
                            <label for="searchInput">검색</label>
                        </h3>
                        <search-form />
                    </div>
                </div>
            </div>
            <div id="mw-panel">
                <div id="p-logo" role="banner">
                    <nuxt-link to="/" class="mw-wiki-logo" href="/" title="대문으로 가기"></nuxt-link>
                </div>
                <div class="portal" role="navigation" id="p-navigation" aria-labelledby="p-navigation-label">
                    <h3 id="p-navigation-label">둘러보기</h3>
                    <div class="body">
                        <ul>
                            <li>
                                <nuxt-link to="/" title="대문으로 가기 [Alt+Shift+z]" accesskey="z">대문</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/RecentChanges" title="위키의 최근 바뀐 목록 [Alt+Shift+r]" accesskey="r">최근 바뀜</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/RecentDiscuss" title="위키의 최근 토론 목록">최근 토론</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/random" title="임의 문서 불러오기 [Alt+Shift+x]" accesskey="x">임의 문서로</nuxt-link>
                            </li>
                        </ul>
                    </div>
                </div>
                <div class="portal" role="navigation" id="p-tb" aria-labelledby="p-tb-label">
                    <h3 id="p-tb-label">도구</h3>
                    <div class="body">
                        <ul>
                            <li>
                                <nuxt-link :to="doc_action_link($store.state.page.data.document, 'backlink')" title="여기를 가리키는 모든 위키 문서의 목록 [Alt+Shift+j]" accesskey="j">여기를 가리키는 문서</nuxt-link>
                            </li>
                            <li v-if="$store.state.page.data.user">
                                <nuxt-link :to="contribution_link($store.state.page.data.user.uuid)" title="이 사용자의 기여 목록">사용자 기여</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/Upload" title="파일 올리기 [Alt+Shift+u]" accesskey="u">파일 올리기</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/NeededPages" title="필요한 문서">필요한 문서</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/OrphanedPages" title="고립된 문서">고립된 문서</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/OldPages" title="방치된 문서">방치된 문서</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/ShortestPages" title="내용이 짧은 문서">내용이 짧은 문서</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/LongestPages" title="내용이 긴 문서">내용이 긴 문서</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/BlockHistory" title="차단 기록">차단 기록</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/RandomPage" title="임의 문서들">임의 문서들</nuxt-link>
                            </li>
                            <li>
                                <nuxt-link to="/License" title="라이선스">라이선스</nuxt-link>
                            </li>
                        </ul>
                    </div>
                </div>
            </div>
            <div id="footer" role="contentinfo">
                <ul v-if="$store.state.page.viewName === 'wiki' && $store.state.page.data.date" id="footer-info">
                    <li id="footer-info-lastmod">이 문서는 <local-date :date="$store.state.page.data.date" />에 마지막으로 편집되었습니다.</li>
                    <li id="footer-info-copyright" v-html="$store.state.config['wiki.copyright_text']"></li>
                </ul>
                <ul id="footer-places">
                    <li>
                        <nuxt-link to="/License" class="extiw">엔진 라이선스</nuxt-link>
                    </li>
                </ul>
                <ul id="footer-icons" class="noprint">
                    <li id="footer-poweredbyico">
                        <a href="//theseed.io/">Powered by the seed Engine</a>
                    </li>
                </ul>
            </div>
        </div>
        <setting />
    </div>
</template>

<style>
@import './static/css/bootstrap.min.css';
@import './static/css/jquery-ui.min.css';
@import './static/css/layout.css';

.mw-wiki-logo {
    background-image: var(--logo-url);
}
</style>

<script>
import Common from '~/mixins/common';
import LocalDate from '~/components/localDate';
import Setting from '~/components/setting';
import searchForm from './searchForm';

if (typeof window !== 'undefined') {
    try {
        require("./static/js/jquery.min.js");
        require("./static/js/jquery-ui.min.js");
        require("./static/js/tether.min.js");
        require("./static/js/bootstrap.min.js");
        require("./static/js/collapsibleTabs.js");
        require("./static/js/vector.js");
    } catch(e) {}
}
export default {
    mixins: [Common],
    components: {
        LocalDate,
        Setting,
        searchForm
    },
    computed: {
        logoUrl() {
            return this.$store.state.config['wiki.logo_url'] ? `url(${this.$store.state.config['wiki.logo_url']})`: `url(${require('./static/images/wiki-logo.svg')})`;
        }
    }
}
</script>
