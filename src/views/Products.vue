<template>
  <div>
    <h1>Products</h1>
    <input type="text" v-model="searchString" @input="pushQueryToUrl({search: searchString})">
    <div v-for="c in categories.data" :key="c.id">
      <button @click="pushQueryToUrl({categoryId: c.id})">{{ c.title }}</button>
    </div>
    <select name="sort" v-model="sort" @change="asd($event)">
      <option value="updatedAt:desc">Newest</option>
      <option value="price:asc">Price: Ascending</option>
      <option value="price:desc">Price: Descending</option>
    </select>
    <div v-for="p in products.data" :key="p.id">
      <p>{{ p.title }}</p>
      <p>{{ p.price }}</p>
      <p>{{ p.Dualsim }}</p>
      <p>{{ p.Lidar }}</p>
    </div>
    <div v-for="i in products?.meta?.pagination?.pageCount" :key="i">
      <button @click="pushQueryToUrl({page: i})">{{ i }}</button>
    </div>

<div id="solution_finder">
  <h1 style="font-size:27px; margin-top:45px; margin-left:40px; font-weight:normal;"> Product Filter </h1>
  <ul id="filters" style="list-style:none; margin-top:25px; line-height:30px; ">
    <li>
      <input type="checkbox" value="all" checked="checked" id="all" style="display:none;">
    </li>
     <li>
      <input type="checkbox" value="Dualsim" id="Dualsim">
      <label for="filter-category">Dualsim</label>
    </li>
    <li>
      <input type="checkbox" value="Lidar" id="Lidar">
      <label for="filter-category">Lidar</label>
    </li>
    </ul>
</div>
    </div>
</template>

<script>
import { useECommerceStore } from "@/stores/e-commerce";
import { useRoute, useRouter } from "vue-router";
import { ref, onMounted, watchEffect } from "vue";
import { storeToRefs } from 'pinia'
import qs from 'qs'
export default {
  setup() {
    const eCommerceStore = useECommerceStore();
    const { products } = storeToRefs(eCommerceStore);
    const { categories } = storeToRefs(eCommerceStore);
    const route = useRoute();
    const router = useRouter();
    const sort = ref(route.query.sort || "updatedAt:desc");
    const searchString = ref('')
    let ezQuery = {};
    function asd(event) {
      pushQueryToUrl({ sort: event.target.value });
    }
    function pushQueryToUrl(queryParam) {
      Object.entries(queryParam).forEach(([key, value]) => {
        if (value) {
          ezQuery[key] = value;
        }else{
          ezQuery[key] = undefined;
        }
      });
      router.push({query: ezQuery})
    }

function filter(array = [], filters = {}) {
    const keys = Object.keys(filters).filter(key => filters.hasOwnProperty(key));
    return array.filter(elem => {
        const commonKeys = keys.filter(key => elem.hasOwnProperty(key));
        return commonKeys.reduce((flag, key) => (flag && filters[key].includes(elem[key])), true);
    });
}


    async function createProductQuery() {
      ezQuery = {
        page: route.query.page,
        gte: route.query.gte,
        lte: route.query.lte,
        sort: route.query.sort,
        categoryId: route.query.categoryId,
        search: route.query.search != '' || route.query.search ? route.query.search : undefined
      }
      const pagination = {page: route.query.page || 1, pageSize: 5};
      const populate = ["category"];
      const sort = route.query.sort ? [route.query.sort] : ["updatedAt:desc"];
      const search = route.query.search != '' && route.query.search ? route.query.search : undefined
      const filters = {
          $and: [
            {
              price: {
                $gte: route.query.price_gte,
              },
            },
            {
              price: {
                $lte: route.query.price_lte,
              },
            },
            {
              category:{
                id: {
                  $eq: route.query.categoryId || categories[0]?.id,
                }
              }
            },
            {
              title: {
                $startsWith: search,
              }
            }
          ],
        };
      const query = {
        populate,
        pagination,
        sort,
        filters,
        search,
      }
      await eCommerceStore.fetchCategories();
      await eCommerceStore.fetchProducts(query);
    }
    watchEffect(() => {      
      createProductQuery();
    });
    onMounted(() => {});
    return {
      products,
      eCommerceStore,
      pushQueryToUrl,
      categories,
      sort,
      asd
    };
  },
  name: "Products",
};
</script>

<style scoped>
</style>