<template>
  <div class="sw-product-details">
    <div class="product-details__mobile-top">
      <SwProductHeading class="product-details__heading" :product="product" />
    </div>

    <SwPluginSlot name="product-page-description" :slot-context="product">
      <p class="product-details__description" v-html="description" />
    </SwPluginSlot>

    <!-- <div class="product-details__action">
      <button v-if="sizes.length > 0" class="sf-action">Size guide</button>
    </div>-->
    <div
      class="product-details-wrapper"
      :class="{ 'product-details-wrapper__loaded': hasChildren }"
    >
      <div v-if="hasChildren" class="product-details__section">
        <div
          v-for="productType in getAllProductOptionsTypes"
          :key="productType"
        >
          <SwProductColors
            v-if="
              getAllProductOptions[productType].find(({ color }) => !!color)
            "
            :colors="getAllProductOptions[productType]"
            :value="selected[productType]"
            :label="productType"
            @input="handleChange(productType, $event)"
          />
          <SwProductSelect
            v-else
            :value="selected[productType]"
            :options="getAllProductOptions[productType]"
            :label="productType"
            @change="handleChange(productType, $event)"
          />
        </div>
      </div>
    </div>
    <div class="product-details__section">
      <SfAlert
        message="Low in stock"
        type="warning"
        class="product-details__alert mobile-only"
      />
      <SfAddToCart
        v-model="quantity"
        :stock="stock"
        class="product-details__add-to-cart"
        @click="addToCart"
      />
      <SwPluginSlot
        name="product-page-add-to-cart-button-after"
        :slot-context="product"
      />
      <div
        v-if="getProductNumber(product)"
        class="product-details__product-number"
      >
        <p>
          Product number: <span>{{ getProductNumber(product) }}</span>
        </p>
      </div>
    </div>
    <SwProductTabs
      :properties="properties"
      :reviews="reviews"
      :manufacturer="manufacturer"
    />
  </div>
</template>
<script>
import { SfAlert, SfAddToCart } from "@storefront-ui/vue"
import {
  getProductNumber,
  getProductOptions,
  getProductOptionsUrl,
  getProductProperties,
  getProductReviews,
} from "@shopware-pwa/helpers"
import { useAddToCart } from "@shopware-pwa/composables"
import SwProductHeading from "@/components/SwProductHeading"
import SwProductSelect from "@/components/SwProductSelect"
import SwProductColors from "@/components/SwProductColors"
import SwPluginSlot from "sw-plugins/SwPluginSlot"

import SwProductTabs from "@/components/SwProductTabs"
export default {
  name: "SwProductDetails",

  components: {
    SfAlert,
    SfAddToCart,
    SwProductHeading,
    SwProductSelect,
    SwProductTabs,
    SwProductColors,
    SwPluginSlot,
  },
  props: {
    product: {
      type: Object,
      default: () => ({}),
    },
    page: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      selected: {},
    }
  },
  setup({ page }, { root }) {
    const { addToCart, quantity } = useAddToCart(root, page && page.product)

    return {
      quantity,
      addToCart,
    }
  },

  computed: {
    description() {
      return (
        this.product &&
        (this.product.description ||
          (this.product.translated && this.product.translated.description))
      )
    },
    hasChildren() {
      return (
        this.product && this.product.children && this.product.children.length
      )
    },

    properties() {
      return getProductProperties({ product: this.product })
    },

    // TODO: move to helpers
    getAllProductOptions() {
      const options = getProductOptions({
        product: this.product,
      })
      return options
    },

    getAllProductOptionsTypes() {
      return this.getAllProductOptions && Object.keys(this.getAllProductOptions)
    },

    manufacturer() {
      return this.product && this.product.manufacturer
    },
    reviews() {
      return getProductReviews({ product: this.product })
    },

    stock() {
      return this.product && this.product.stock
    },

    selectedOptions() {
      return this.selected
    },
  },
  watch: {
    selectedOptions(selected, selectedOld) {
      if (
        Object.keys(this.getAllProductOptions).length !==
        Object.keys(selected).length
      ) {
        return
      }

      const options = []
      for (const attribute of Object.keys(selected)) {
        options.push(selected[attribute])
      }
      const url = getProductOptionsUrl({
        product: this.product,
        options,
      })

      this.$router.push(this.$i18n.path(url))
    },
  },

  mounted() {
    this.product.options?.forEach((option) => {
      this.selected = Object.assign({}, this.selected, {
        [option.group.name]: option.id,
      })
    })
  },

  methods: {
    toggleWishlist(index) {
      this.products[index].isOnWishlist = !this.products[index].isOnWishlist
    },

    handleChange(attribute, option) {
      this.selected = Object.assign({}, this.selected, { [attribute]: option })
    },

    getProductNumber,
  },
}
</script>

<style lang="scss" scoped>
@import "@/assets/scss/variables";

@mixin for-iOS {
  @supports (-webkit-overflow-scrolling: touch) {
    @content;
  }
}
.product-details-wrapper {
  @include for-desktop {
    height: 0;
    transition: height 0.66s ease-out;
  }

  &__loaded {
    @include for-desktop {
      height: auto;
    }
  }
}

.product-details {
  &__action {
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    margin: var(--spacer-base) 0 calc(var(--spacer-base) / 2);
  }
  &__action-button {
    padding: var(--spacer-xs) 0;
  }
  &__add-to-cart {
    margin: 1.5rem 0;
    @include for-desktop {
      margin: var(--spacer-xl) 0;
    }
  }
  &__alert {
    margin-top: 1.5rem;
  }
  &__attribute {
    margin-bottom: var(--spacer-base);
  }
  &__description {
    margin: var(--spacer-xl) 0;
    font-family: var(--font-family-secondary);
    font-size: var(--font-sm);
  }
  &__heading {
    margin: var(--spacer-base) 0 0 0;
    @include for-desktop {
      margin: var(--spacer-lg) 0 0 0;
    }
  }
  &__mobile-bar {
    display: none;
    padding: var(--spacer-sm) 0;
    box-sizing: border-box;
    .product--is-active & {
      display: block;
      @include for-desktop {
        display: none;
      }
    }
    @include for-desktop {
      display: none;
    }
  }
  &__mobile-top {
    display: flex;
    align-items: center;
    @include for-desktop {
      display: block;
    }
  }
  &__product-number {
    p {
      font-size: var(--font-sm);
      font-weight: bold;

      span {
        font-weight: var(--font-light);
      }
    }
  }
  &__section {
    padding-bottom: 10px;
    padding-top: 20px;
    @include for-desktop {
      padding-bottom: 0;
    }
    // &-attributes {
    //   height: 50px;
    // }
  }
  &__review {
    padding-bottom: var(--spacer-base);
    border-bottom: var(--c-light) solid 1px;
    margin-bottom: var(--spacer-base);
    &:last-of-type {
      border: none;
      padding-bottom: 0;
      margin-bottom: 0;
    }
    @include for-desktop {
      padding-bottom: var(--spacer-xl);
    }
  }
  &__product-property {
    padding: var(--spacer-2xs) 0;
  }
}
</style>
