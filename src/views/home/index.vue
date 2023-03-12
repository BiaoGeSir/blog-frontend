<!--
 * @author: Tusi
 * @description: 首页
-->
<template>
    <base-layout>
        <a-skeleton :loading="loading" active :paragraph="{ rows: 12 }">
            <template v-if="articleList.length > 0">
                <section class="article-list">
                    <card-article v-for="article in articleList" :key="article.id" :article="article" />
                </section>

                <a-pagination
                    class="pagination-common"
                    v-model:current="pageInfo.page"
                    v-model:pageSize="pageInfo.pageSize"
                    :total="total"
                    show-less-items
                    simple
                    @change="onPageNoChange"
                />
            </template>

            <template v-else>
                <a-empty />
            </template>
        </a-skeleton>
    </base-layout>
</template>

<script lang="ts">
import { defineComponent, reactive, ref, watch } from "vue";
import { LocationQuery, useRoute, useRouter } from "vue-router";
import { Pagination } from "ant-design-vue";
import { ArticleDTO } from "@/bean/dto";
import { articleService } from "@/services/article";
import { useAsyncLoading } from "@/hooks/async";
import CardArticle from "@/components/card/card-article.vue";
import { setScrollTop } from "@/utils/dom";

export default defineComponent({
    name: "Home",
    components: {
        CardArticle,
        [Pagination.name]: Pagination,
    },
    setup() {
        // route
        const route = useRoute();
        const router = useRouter();

        const articleList = ref<ArticleDTO[]>([]);

        const pageInfo = reactive({
            page: 1,
            pageSize: 6,
        });

        const total = ref(0);

        const handleGetArticleList = async (isChangePage: boolean) => {
            const res = await articleService.page(pageInfo);
            articleList.value = res.data;
            total.value = res.total;
            if (isChangePage) {
                setScrollTop({
                    useAnimation: true,
                    duration: 0.3,
                });
            }
        };

        const { trigger: getPageList, loading } = useAsyncLoading(handleGetArticleList);

        // 监听路由变化调用查询
        watch(
            () => route.query,
            (val: LocationQuery, oldVal) => {
                const { page } = val;
                if (page) {
                    pageInfo.page = Number(page);
                } else {
                    pageInfo.page = 1;
                }
                const isChangePage = page !== oldVal?.page;
                getPageList(isChangePage);
            },
            {
                immediate: true,
            }
        );

        // 分页改变
        const onPageNoChange = (page: number) => {
            router.push({ query: { page: String(page) } });
        };

        return {
            loading,
            articleList,
            pageInfo,
            total,
            onPageNoChange,
        };
    },
});
</script>
