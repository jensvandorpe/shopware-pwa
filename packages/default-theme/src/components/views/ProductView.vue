<template>
  <div v-if="product" id="product">
    <SwPluginSlot name="product-page-details-before" :slot-context="product" />
    <SwGoBackArrow class="product-page-back" />
    <div class="product">
      <SwProductGallery :product="product" class="product__gallery" />
      <div class="product__description">
        <SwProductDetails :product="product" :page="page" />
      </div>
    </div>
    <SwPluginSlot name="product-page-details-after" :slot-context="product" />
    <div class="products__recomendations">
      <div class="products-recomendations__section">
        <SwProductCarousel />
      </div>
    </div>
    <SfSection
      title-heading="Share Your Look"
      subtitle-heading="#YOURLOOK"
      class="section"
    >
      <div class="images-grid">
        <div class="images-grid__row">
          <div class="images-grid__col">
            <SfImage src="/img/imageA.png">katherina_trn</SfImage>
          </div>
          <div class="images-grid__col">
            <SfImage src="/img/imageB.png">katherina_trn</SfImage>
          </div>
          <div class="images-grid__col">
            <SfImage src="/img/imageC.png">katherina_trn</SfImage>
          </div>
        </div>
        <div class="images-grid__row">
          <div class="images-grid__col">
            <SfImage src="/img/imageC.png">katherina_trn</SfImage>
          </div>
          <div class="images-grid__col">
            <SfImage src="/img/imageD.png">katherina_trn</SfImage>
          </div>
          <div class="images-grid__col">
            <SfImage src="/img/imageA.png">katherina_trn</SfImage>
          </div>
        </div>
      </div>
    </SfSection>
    <div class="product__advertisement">
      <SwProductAdvertisement />
    </div>
  </div>
</template>
<script>
import { SfImage, SfSection } from "@storefront-ui/vue"
import { useProduct } from "@shopware-pwa/composables"
import SwGoBackArrow from "@/components/atoms/SwGoBackArrow"
import SwProductGallery from "@/components/SwProductGallery"
import SwProductDetails from "@/components/SwProductDetails"
import SwProductCarousel from "@/components/SwProductCarousel"
import SwProductAdvertisement from "@/components/SwProductAdvertisement"
import SwPluginSlot from "sw-plugins/SwPluginSlot"

export default {
  name: "Product",
  components: {
    SwGoBackArrow,
    SfImage,
    SfSection,
    SwProductGallery,
    SwProductDetails,
    SwProductCarousel,
    SwProductAdvertisement,
    SwPluginSlot,
  },
  props: {
    page: {
      type: Object,
      default: () => ({}),
    },
  },
  data() {
    return {
      productWithChildren: null,
      relatedProducts: [],
      selectedSize: null,
      selectedColor: null,
    }
  },
  computed: {
    product() {
      return this.productWithChildren
        ? this.productWithChildren.value
        : this.page.product
    },
  },
  // load children association from the parent - variants loading
  async mounted() {
    if (!this.page.product.parentId) {
      return
    }

    try {
      const { loadAssociations, product } = useProduct(this, this.page.product)
      this.productWithChildren = product
      await loadAssociations()
    } catch (e) {
      console.error("ProductView:mounted:loadAssociations", e)
    }
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

#product {
  position: relative;

  @include for-desktop {
    max-width: 1240px;
    margin: auto;
  }
}

.section {
  padding: 0 var(--spacer-base);
  @include for-desktop {
    padding: 0;
  }
}

.images-grid {
  &__row {
    display: flex;
    & + & {
      margin-top: calc(var(--spacer-base) / 2);
      @include for-desktop {
        margin-top: var(--spacer-base);
      }
    }
  }
  &__col {
    margin: 0;
    & + & {
      margin-left: calc(var(--spacer-base) / 2);
      @include for-desktop {
        margin-left: var(--spacer-base);
      }
    }
  }
}
.product {
  @include for-desktop {
    display: flex;
  }
  &__gallery,
  &__description {
    flex: 1;
  }
  &__description {
    padding: 0 var(--spacer-sm);
    @include for-desktop {
      margin-left: calc(var(--spacer-base) * 3);
    }
  }
}
.product-page-back {
  left: 0.5rem;
  position: absolute;
  top: 1.5rem;
  z-index: 4;
}
</style>
