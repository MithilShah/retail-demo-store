<template>
  <Layout :isLoading="isLoading" :previousPageLinkProps="previousPageLinkProps">
    <template #default>
      <div class="container">
        <main class="product-container mb-5 text-left">
          <div class="title-and-rating mb-md-3">
            <h1 class="product-name">{{ product.name }}</h1>
            <FiveStars></FiveStars>
          </div>

          <div class="add-to-cart-and-description">
            <ProductPrice :price="product.price" :discount="discount" class="mb-1"></ProductPrice>

            <div class="mb-2">
              <template v-if="outOfStock">Sorry, this item is currently out of stock.</template>
              <!-- <template v-else>Items currently in stock: {{ product.current_stock }}</template> -->
            </div>

            <div v-if="fenixenablePDP == 'TRUE'" class="fenix-estimates">
              <Fenixmaster
                :currentvariant="fenixcurrentvariant"
              >
              </Fenixmaster>
            </div>

            <div v-if="cartHasMaxAmount" class="mb-2">Sorry, you cannot add more of this item to your cart.</div>

            <div class="mb-5 mb-md-4 d-flex">
              <!-- <button
                v-if="!outOfStock && !cartHasMaxAmount"
                class="quantity-dropdown mr-3 btn btn-outline-secondary dropdown-toggle"
                type="button"
                id="quantity-dropdown"
                data-toggle="dropdown"
                aria-haspopup="true"
                aria-expanded="false"
              >
                Qty: {{ quantity }}
              </button> -->
              <div class="dropdown-menu" aria-labelledby="quantity-dropdown">
                <button
                  v-for="i in Math.max(0, Math.min(9, product.current_stock - quantityInCart))"
                  :key="i"
                  class="dropdown-item"
                  @click="quantity = i"
                >
                  {{ i }}
                </button>
              </div>
              <button class="add-to-cart-btn btn" @click="addProductToCart" :disabled="outOfStock || cartHasMaxAmount">
                Subscribe
              </button>
            </div>

            <p>{{ product.description }}</p>
          </div>

          <div class="product-img">
            <img :src="productImageUrl" class="img-fluid" :alt="product.name" />
          </div>
        </main>

        <RecommendedProductsSection :experiment="experiment" :recommendedProducts="relatedProducts" :feature="feature">
          <template #heading
            >Compare similar items
            <DemoGuideBadge
              v-if="demoGuideBadgeArticle"
              :article="demoGuideBadgeArticle"
              hideTextOnSmallScreens
            ></DemoGuideBadge>
          </template>
        </RecommendedProductsSection>
      </div>
    </template>
  </Layout>
</template>

<script>
import swal from 'sweetalert';
import { mapState, mapActions, mapGetters } from 'vuex';

import { RepositoryFactory } from '@/repositories/RepositoryFactory';
import { AnalyticsHandler } from '@/analytics/AnalyticsHandler';

import { product } from '@/mixins/product';

import Layout from '@/components/Layout/Layout';
import ProductPrice from '@/components/ProductPrice/ProductPrice';
import FiveStars from '@/components/FiveStars/FiveStars';
import RecommendedProductsSection from '@/components/RecommendedProductsSection/RecommendedProductsSection';
import { discountProductPrice } from '@/util/discountProductPrice';
import DemoGuideBadge from '@/components/DemoGuideBadge/DemoGuideBadge';

import { getDemoGuideArticleFromPersonalizeARN } from '@/partials/AppModal/DemoGuide/config';
import Fenixmaster from '@/components/Fenix/Fenixmaster';

const RecommendationsRepository = RepositoryFactory.get('recommendations');
const MAX_RECOMMENDATIONS = 10;
const EXPERIMENT_FEATURE = 'product_detail_related';

export default {
  name: 'ProductDetail',
  components: {
    Layout,
    ProductPrice,
    FiveStars,
    RecommendedProductsSection,
    DemoGuideBadge,
    Fenixmaster,
  },
  mixins: [product],
  props: {
    discount: {
      type: Boolean,
      required: false,
      default: false,
    },
    currentvariant: [Object, Number, String],
  },
  data() {
    return {
      quantity: 1,
      feature: EXPERIMENT_FEATURE,
      relatedProducts: null,
      demoGuideBadgeArticle: null,
      experiment: null,
      fenixcurrentvariant: {},
      fenixenablePDP : process.env.VUE_APP_FENIX_ENABLED_PDP,
    };
  },
  computed: {
    ...mapState({ user: (state) => state.user,
                  cart: (state) => state.cart.cart }),
    ...mapGetters(['personalizeUserID']),
    isLoading() {
      return !this.product;
    },
    previousPageLinkProps() {
      if (!this.product) return null;

      return {
        to: `/category/${this.product.category}`,
        text: this.readableProductCategory,
      };
    },
    cartItem() {
      if (!this.product || !this.cart) return null;

      return this.cart.items.find((item) => item.product_id === this.product.id);
    },
    quantityInCart() {
      return this.cartItem?.quantity ?? 0;
    },
    cartHasMaxAmount() {
      if (!this.product || !this.cartItem) return false;

      return !this.outOfStock && this.cartItem.quantity >= this.product.current_stock;
    },
  },
  watch: {
    $route: {
      immediate: true,
      handler() {
        this.fetchData();
      },
    },
    personalizeUserID() {
      this.getRelatedProducts();
    },
  },
  methods: {
    ...mapActions(['addToCart']),
    resetQuantity() {
      this.quantity = 1;
    },
    async addProductToCart() {
      await this.addToCart({
        product: {
          ...this.product,
          price: this.discount ? discountProductPrice(this.product.price) : this.product.price,
        },
        quantity: this.quantity,
        feature: this.$route.query.feature,
        exp: this.$route.query.exp,
      });

      this.renderAddedToCartConfirmation();

      this.resetQuantity();
      AnalyticsHandler.recordShoppingCart(this.user, this.cart)
    },
    async fetchData() {
      await this.getProductByID(this.$route.params.id);

      this.fenixcurrentvariant = this.product.id;

      this.getRelatedProducts();

      this.recordProductViewed(this.$route.query.feature, this.$route.query.exp, this.$route.query.di);
    },
    async getRelatedProducts() {
      // reset in order to trigger recalculation in carousel - carousel UI breaks without this
      this.relatedProducts = null;

      this.experiment = null;
      this.demoGuideBadgeArticle = null;

      const response = await RecommendationsRepository.getRelatedProducts(
        this.personalizeUserID ?? '',
        this.product.id,
        this.product.category,
        MAX_RECOMMENDATIONS,
        EXPERIMENT_FEATURE,
      );

      if (response.headers) {
        const experimentName = response.headers['x-experiment-name'];
        // For composite use cases such as this one, we may get more than one recipe (comma delimited). Use the first recipe to lookup demo badge.
        const personalizeRecipe = response.headers['x-personalize-recipe'] ? response.headers['x-personalize-recipe'].split(',')[0] : null;

        if (experimentName) this.experiment = `Active experiment: ${experimentName}`;
        if (personalizeRecipe) this.demoGuideBadgeArticle = getDemoGuideArticleFromPersonalizeARN(personalizeRecipe);
      }

      this.relatedProducts = response.data;

      if (this.relatedProducts.length > 0 && 'experiment' in this.relatedProducts[0]) {
        AnalyticsHandler.identifyExperiment(this.user, this.relatedProducts[0].experiment);
      }
    },
    renderAddedToCartConfirmation() {
      swal({
        title: 'Subscribe',
        icon: 'success',
        buttons: {
          cancel: 'Continue Browsing',
          cart: 'View Subscriptions',
        },
      }).then((value) => {
        switch (value) {
          case 'cancel':
            break;
          case 'cart':
            this.$router.push('/cart');
        }
      });
    },
  },
};
</script>

<style scoped>
.product-container {
  display: grid;
  grid-gap: 15px;
  grid-template-columns: 1fr;
  grid-template-rows: auto;
  grid-template-areas:
    'TitleAndRating'
    'ProductImage'
    'AddToCardAndDescription';
}

@media (min-width: 768px) {
  .product-container {
    grid-template-columns: 1fr 1fr;
    grid-column-gap: 30px;
    grid-template-rows: auto;
    grid-row-gap: 00px;
    grid-template-areas:
      'ProductImage TitleAndRating'
      'ProductImage AddToCardAndDescription'
      'ProductImage .';
  }
}

.title-and-rating {
  grid-area: TitleAndRating;
}

.product-name {
  font-size: 1.5rem;
}

.add-to-cart-and-description {
  grid-area: AddToCardAndDescription;
}

.quantity-dropdown {
  border-color: var(--grey-600);
}

.quantity-dropdown:hover,
.quantity-dropdown:focus {
  background: var(--grey-900);
  border-color: var(--grey-900);
}

.add-to-cart-btn {
  background: var(--grey-600);
  color: var(--white);
}

.add-to-cart-btn:hover,
.add-to-cart-btn:focus {
  background: var(--grey-900);
}

.product-img {
  grid-area: ProductImage;
}
</style>
