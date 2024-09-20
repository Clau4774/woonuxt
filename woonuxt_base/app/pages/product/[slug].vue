<script lang="ts" setup>
import jsonData from '../../data/data.json'


const route = useRoute();
const { storeSettings } = useAppConfig();
const { arraysEqual, formatArray, checkForVariationTypeOfAny } = useHelpers();
const { addToCart, isUpdatingCart } = useCart();
const { t } = useI18n();
const slug = route.params.slug as string;

const {data} = jsonData;

// const { data } = {
//   "data": {
//     "product": {
//       "name": "Teléfono Inteligente",
//       "type": "VARIABLE",
//       "databaseId": 102,
//       "id": "cHJvZHVjdDoxMDI=",
//       "metaData": [
//         {
//           "id": 1,
//           "key": "_wc_average_rating",
//           "value": "4.8"
//         },
//         {
//           "id": 2,
//           "key": "_wc_review_count",
//           "value": "25"
//         }
//       ],
//       "slug": "telefono-inteligente",
//       "sku": "TEL-INT-102",
//       "description": "<p>Un teléfono inteligente de última generación con múltiples características.</p>",
//       "rawDescription": "Un teléfono inteligente de última generación con múltiples características.",
//       "shortDescription": "<p>Teléfono inteligente con pantalla OLED y cámara de alta resolución.</p>",
//       "attributes": {
//         "nodes": [
//           {
//             "variation": true,
//             "name": "Capacidad",
//             "id": "YXR0cmlidXRlOjE=",
//             "options": ["64GB", "128GB"],
//             "label": "Capacidad",
//             "scope": "global",
//             "slug": "capacidad",
//             "terms": {
//               "nodes": [
//                 {
//                   "name": "64GB",
//                   "slug": "64gb",
//                   "taxonomyName": "pa_capacidad",
//                   "databaseId": 201
//                 },
//                 {
//                   "name": "128GB",
//                   "slug": "128gb",
//                   "taxonomyName": "pa_capacidad",
//                   "databaseId": 202
//                 }
//               ]
//             }
//           }
//         ]
//       },
//       "productCategories": [
//         {
//           "name": "Electrónicos",
//           "slug": "electronicos"
//         }
//       ],
//       "terms": [
//         {
//           "taxonomyName": "Etiqueta",
//           "name": "Oferta",
//           "slug": "oferta",
//           "count": 120
//         }
//       ],
//       "price": "299.99",
//       "regularPrice": "349.99",
//       "salePrice": "299.99",
//       "onSale": true,
//       "variations": [
//         {
//           "id": "cHJvZHVjdFZhcmlhdGlvbjoxMDIwMQ==",
//           "name": "Teléfono Inteligente - 64GB",
//           "price": "299.99"
//         },
//         {
//           "id": "cHJvZHVjdFZhcmlhdGlvbjoxMDIwMg==",
//           "name": "Teléfono Inteligente - 128GB",
//           "price": "349.99"
//         }
//       ],
//       "related": {
//         "nodes": [
//           {
//             "name": "Auriculares Inalámbricos",
//             "type": "SIMPLE",
//             "databaseId": 103,
//             "id": "cHJvZHVjdDoxMDM=",
//             "price": "59.99",
//             "regularPrice": "79.99",
//             "salePrice": "59.99",
//             "onSale": true
//           },
//           {
//             "name": "Cargador Rápido",
//             "type": "SIMPLE",
//             "databaseId": 104,
//             "id": "cHJvZHVjdDoxMDQ=",
//             "price": "19.99",
//             "regularPrice": "24.99",
//             "salePrice": "19.99",
//             "onSale": true
//           }
//           // Puedes agregar más productos relacionados si lo deseas
//         ]
//       },
//       "reviews": {
//         "averageRating": "4.8",
//         "edges": [
//           {
//             "rating": 5,
//             "node": {
//               "content": "Excelente teléfono, muy rápido y con buena batería.",
//               "id": "Y29tbWVudDox",
//               "date": "2023-10-01",
//               "author": {
//                 "node": {
//                   "name": "Juan Pérez",
//                   "avatar": {
//                     "url": "https://ejemplo.com/avatar/juanperez.jpg"
//                   }
//                 }
//               }
//             }
//           },
//           {
//             "rating": 4,
//             "node": {
//               "content": "Buena relación calidad-precio.",
//               "id": "Y29tbWVudDoy",
//               "date": "2023-09-25",
//               "author": {
//                 "node": {
//                   "name": "María Gómez",
//                   "avatar": {
//                     "url": "https://ejemplo.com/avatar/mariagomez.jpg"
//                   }
//                 }
//               }
//             }
//           }
//           // Puedes agregar más reseñas si lo deseas
//         ]
//       }
//     }
//   }
// }

// if (!data.product) {
//   throw showError({ statusCode: 404, statusMessage: t('messages.shop.productNotFound') });
// }
if (!data.products.nodes[0]?.simpleProduct) {
  throw showError({ statusCode: 404, statusMessage: t('messages.shop.productNotFound') });
}

const product = ref<Product>(data.products.nodes.filter(elem => elem.slug === slug)[0]?.simpleProduct);
const quantity = ref<number>(1);
//const activeVariation = ref<Variation | null>(null);
const variation = ref<VariationAttribute[]>([]);
const indexOfTypeAny = computed<number[]>(() => checkForVariationTypeOfAny(product));
const attrValues = ref();
const isSimpleProduct = computed<boolean>(() => product.type === "SIMPLE");
const isVariableProduct = computed<boolean>(() => product.type === "VARIABLE");
const isExternalProduct = computed<boolean>(() => product.type === "EXTERNAL");

const type = computed(() => null || product);
console.log(type);
const selectProductInput = computed<any>(() => ({ productId: type.databaseId, quantity: quantity.value })) as ComputedRef<AddToCartInput>;

const mergeLiveStockStatus = (payload: Product): void => {
  product.stockStatus = payload.stockStatus ?? product.stockStatus;

  payload.variations?.nodes?.forEach((variation: Variation, index: number) => {
    if (product.variations?.nodes[index]) {
      product.variations.nodes[index].stockStatus = variation.stockStatus;
    }
  });
};

onMounted(async () => {
  try {
    const { product } = await GqlGetStockStatus({ slug });
    if (product) mergeLiveStockStatus(product as Product);
  } catch (error: any) {
    const errorMessage = error?.gqlErrors?.[0].message;
    if (errorMessage) console.error(errorMessage);
  }
});

const updateSelectedVariations = (variations: VariationAttribute[]): void => {
  if (!product.value.variations) return;

  attrValues.value = variations.map((el) => ({ attributeName: el.name, attributeValue: el.value }));
  const clonedVariations = JSON.parse(JSON.stringify(variations));
  const getActiveVariation = product.value.variations?.nodes.filter((variation: any) => {
    // If there is any variation of type ANY set the value to ''
    if (variation.attributes) {
      // Set the value of the variation of type ANY to ''
      indexOfTypeAny.value.forEach((index) => (clonedVariations[index].value = ''));

      return arraysEqual(formatArray(variation.attributes.nodes), formatArray(clonedVariations));
    }
  });

  if (getActiveVariation[0]) activeVariation.value = getActiveVariation[0];
  selectProductInput.value.variationId = activeVariation.databaseId ?? null;
  selectProductInput.value.variation = activeVariation.value ? attrValues.value : null;
  variation.value = variations;
};

const stockStatus = computed(() => {
  if (isVariableProduct.value) return activeVariation.stockStatus || "OUT_OF_STOCK";
  return type.stockStatus || "OUT_OF_STOCK";
});
const disabledAddToCart = computed(() => {
  if (isSimpleProduct.value) return !type.value || stockStatus.value === "OUT_OF_STOCK" || isUpdatingCart.value;
  return !type.value || stockStatus.value === "OUT_OF_STOCK" || !activeVariation.value || isUpdatingCart.value;
});
</script>

<template>
  <main class="container relative py-6 xl:max-w-7xl">
    <div v-if="product">
      <SEOHead :info="product" />
      <Breadcrumb :product class="mb-6" v-if="storeSettings.showBreadcrumbOnSingleProduct" />

      <div class="flex flex-col gap-10 md:flex-row md:justify-between lg:gap-24">
        <ProductImageGallery
          v-if="product.image"
          class="relative flex-1"
          :main-image="product.image"
          :gallery="product.galleryImages!"
          :node="type"
          :activeVariation="activeVariation || {}" />
        <NuxtImg v-else class="relative flex-1 skeleton" src="/images/placeholder.jpg" :alt="product?.name || 'Product'" />

        <div class="lg:max-w-md xl:max-w-lg md:py-2 w-full">
          <div class="flex justify-between mb-4">
            <div class="flex-1">
              <h1 class="flex flex-wrap items-center gap-2 mb-2 text-2xl font-sesmibold">
                {{ type.name }}
                <LazyWPAdminLink :link="`/wp-admin/post.php?post=${product.databaseId}&action=edit`">Edit</LazyWPAdminLink>
              </h1>
              <StarRating :rating="product.averageRating || 0" :count="product.reviewCount || 0" v-if="storeSettings.showReviews" />
            </div>
            <ProductPrice class="text-xl" :sale-price="type.salePrice" :regular-price="type.regularPrice" />
          </div>

          <div class="grid gap-2 my-8 text-sm empty:hidden">
            <div v-if="!isExternalProduct" class="flex items-center gap-2">
              <span class="text-gray-400">{{ $t('messages.shop.availability') }}: </span>
              <StockStatus :stockStatus @updated="mergeLiveStockStatus" />
            </div>
            <div class="flex items-center gap-2" v-if="storeSettings.showSKU && product.sku">
              <span class="text-gray-400">{{ $t('messages.shop.sku') }}: </span>
              <span>{{ product.sku || 'N/A' }}</span>
            </div>
          </div>

          <div class="mb-8 font-light prose" v-html="product.shortDescription || product.description" />

          <hr />

          <form @submit.prevent="addToCart(selectProductInput)">
            <AttributeSelections
              v-if="isVariableProduct && product.attributes && product.variations"
              class="mt-4 mb-8"
              :attributes="product.attributes.nodes"
              :defaultAttributes="product.defaultAttributes"
              :variations="product.variations.nodes"
              @attrs-changed="updateSelectedVariations" />
            <div
              v-if="isVariableProduct || isSimpleProduct"
              class="fixed bottom-0 left-0 z-10 flex items-center w-full gap-4 p-4 mt-12 bg-white md:static md:bg-transparent bg-opacity-90 md:p-0">
              <input
                v-model="quantity"
                type="number"
                min="1"
                aria-label="Quantity"
                class="bg-white border rounded-lg flex text-left p-2.5 w-20 gap-4 items-center justify-center focus:outline-none" />
              <AddToCartButton class="flex-1 w-full md:max-w-xs" :disabled="disabledAddToCart" :class="{ loading: isUpdatingCart }" />
            </div>
            <a
              v-if="isExternalProduct && product.externalUrl"
              :href="product.externalUrl"
              target="_blank"
              class="rounded-lg flex font-bold bg-gray-800 text-white text-center min-w-[150px] p-2.5 gap-4 items-center justify-center focus:outline-none">
              {{ product?.buttonText || 'View product' }}
            </a>
          </form>

          <div v-if="storeSettings.showProductCategoriesOnSingleProduct && product.productCategories">
            <div class="grid gap-2 my-8 text-sm">
              <div class="flex items-center gap-2">
                <span class="text-gray-400">{{ $t('messages.shop.category', 2) }}:</span>
                <div class="product-categories">
                  <NuxtLink
                    v-for="category in product.productCategories.nodes"
                    :key="category.slug"
                    :to="`/product-category/${decodeURIComponent(category.slug)}`"
                    class="hover:text-primary"
                    :title="category.name"
                    >{{ category.name }}<span class="comma">, </span>
                  </NuxtLink>
                </div>
              </div>
            </div>
            <hr />
          </div>

          <div class="flex flex-wrap gap-4">
            <WishlistButton :product />
            <ShareButton :product />
          </div>
        </div>

      </div>
      <div v-if="product.description || product.reviews" class="my-32">
        <ProductTabs :product />
      </div>
      <div class="my-32" v-if="product.related && storeSettings.showRelatedProducts">
        <div class="mb-4 text-xl font-semibold">{{ $t('messages.shop.youMayLike') }}</div>
        <ProductRow :products="product.related.nodes" class="grid-cols-2 md:grid-cols-4 lg:grid-cols-5" />
      </div>
    </div>
  </main>
</template>

<style scoped>
.product-categories > a:last-child .comma {
  display: none;
}

input[type='number']::-webkit-inner-spin-button {
  opacity: 1;
}
</style>
