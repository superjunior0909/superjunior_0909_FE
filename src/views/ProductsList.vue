<template>
  <main class="products-page">
    <!-- HERO -->
    <section class="page-hero">
      <div class="container">
        <div>
          <p class="eyebrow">공동구매 마켓</p>
          <h1>공동구매 상품</h1>
          <p class="subtitle">
            진행 상태와 카테고리별로 공동구매 상품을 확인하세요.
          </p>
        </div>
      </div>
    </section>

    <!-- 맞춤형 추천 -->
    <section v-if="recommendedLoading || recommendedProducts.length" class="product-grid-section recommend-section">
      <div class="container">
        <div class="section-header">
          <h2 class="section-title">
            맞춤형 추천
            <span v-if="recommendedReason" class="section-reason">{{ recommendedReason }}</span>
          </h2>
        </div>
        <div v-if="recommendedLoading" class="product-grid">
          <article
            v-for="n in 3"
            :key="`skeleton-${n}`"
            class="product-card skeleton-card"
          >
            <div class="image-wrapper skeleton-block"></div>
            <div class="card-body">
              <div class="skeleton-line short"></div>
              <div class="skeleton-line"></div>
              <div class="skeleton-line"></div>
              <div class="skeleton-line short"></div>
              <div class="skeleton-line"></div>
            </div>
          </article>
        </div>
        <div v-else class="product-grid">
          <article
            v-for="product in recommendedProducts"
            :key="product.id"
            class="product-card"
            @click="goToDetail(product.id)"
          >
            <div class="image-wrapper">
              <img :src="product.image" :alt="product.title" @error="(e) => handleProductImageError(e, product)" />

              <div class="badge-group">
                <span
                  v-for="badge in product.badges"
                  :key="badge"
                  class="badge"
                >
                  {{ badge }}
                </span>
              </div>

              <button
                class="bookmark"
                :class="{ active: wishlist.has(product.id) }"
                @click.stop="toggleWishlist(product.id)"
              >
                {{ wishlist.has(product.id) ? '★' : '☆' }}
              </button>
            </div>

            <div class="card-body">
              <p class="category">{{ product.category }}</p>
              <h2>{{ product.title }}</h2>
              <p class="subtitle">{{ product.subtitle }}</p>

              <div class="price-row">
                <p class="current-price">
                  ₩{{ product.currentPrice.toLocaleString() }}
                </p>
                <p class="meta">
                  <span class="discount">{{ product.discountRate }}% OFF</span>
                  <span class="original">
                    ₩{{ product.originalPrice.toLocaleString() }}
                  </span>
                </p>
              </div>

              <div class="progress">
                <div class="progress-head">
                  <span>{{ product.currentCount }}명 참여</span>
                  <span>목표 {{ product.targetCount }}명</span>
                </div>
                <div class="progress-bar">
                  <div
                    class="progress-fill"
                    :style="{ width: `${Math.min(product.currentCount / product.targetCount * 100, 100)}%` }"
                  ></div>
                </div>
              </div>

              <div class="card-footer">
                <span class="time">⏰ {{ product.timeLeft }}</span>
                <div class="footer-actions">
                  <!-- <button 
                    class="btn btn-outline btn-sm"
                    @click.stop="addToCart(product)"
                  >
                    장바구니
                  </button> -->
                  <button 
                    class="btn btn-primary btn-sm"
                    @click.stop="goToDetail(product.id)"
                  >
                    참가하기
                  </button>
                </div>
              </div>
            </div>
          </article>
        </div>
      </div>
    </section>

    <!-- FILTER -->
    <section class="filters">
      <div class="container">

        <!-- STATUS + SEARCH -->
        <div class="filter-row">
          <div class="chips">
            <button
              v-for="section in primarySections"
              :key="section.id"
              class="chip"
              :class="{ active: selectedStatus === section.id }"
              @click="setStatus(section.id)"
            >
              {{ section.label }}
            </button>
          </div>

          <div class="filter-actions">
            <div class="search">
              <input
                v-model.trim="keyword"
                type="search"
                placeholder="상품명을 입력하세요"
                @keyup.enter="search"
              />
              <button class="btn btn-outline" @click="search">검색</button>
            </div>
          </div>
        </div>

        <!-- CATEGORY + SORT -->
        <div class="filter-row secondary">
          <div class="category-wrapper">
            <!-- 기본 카테고리 (전체 + 3개) -->
            <div class="categories primary">
              <button
                v-for="cat in primaryCategories"
                :key="cat.value"
                class="category-btn"
                :class="{ active: selectedCategory === cat.value }"
                @click="filterByCategory(cat.value)"
              >
                <span>{{ cat.icon }}</span>
                <span>{{ cat.label }}</span>
              </button>

              <!-- 더보기 버튼 -->
              <button
                class="category-btn more"
                @click="toggleMoreCategories"
              >
                {{ showMoreCategories ? '접기' : '+ 더보기' }}
              </button>
            </div>

            <!-- 더보기 카테고리 (아래로 펼쳐짐) -->
            <div v-show="showMoreCategories" class="categories secondary">
              <button
                v-for="cat in secondaryCategories"
                :key="cat.value"
                class="category-btn"
                :class="{ active: selectedCategory === cat.value }"
                @click="filterByCategory(cat.value)"
              >
                <span>{{ cat.icon }}</span>
                <span>{{ cat.label }}</span>
              </button>
            </div>
          </div>

          <!-- ✅ 정렬 드롭다운 -->
          <div class="sort">
            <label>
              정렬
              <select v-model="sortBy">
                <option value="popular">인기순</option>
                <option value="discountRate">할인율 높은순</option>
                <option value="priceLow">가격 낮은순</option>
                <option value="priceHigh">가격 높은순</option>
                <option value="deadline">마감 임박순</option>
              </select>
            </label>
          </div>
        </div>
      </div>
    </section>

    <!-- GRID -->
    <section class="product-grid-section">
      <div class="container">
        <div v-if="products.length === 0 && !loading" class="empty-state">
          <p>조건에 맞는 상품이 없습니다.</p>
        </div>

        <div v-else class="product-grid">
          <article
            v-for="product in products"
            :key="product.id"
            class="product-card"
            @click="goToDetail(product.id)"
          >
            <div class="image-wrapper">
              <img :src="product.image" :alt="product.title" @error="(e) => handleProductImageError(e, product)" />

              <div class="badge-group">
                <span
                  v-for="badge in product.badges"
                  :key="badge"
                  class="badge"
                >
                  {{ badge }}
                </span>
              </div>

              <button
                class="bookmark"
                :class="{ active: wishlist.has(product.id) }"
                @click.stop="toggleWishlist(product.id)"
              >
                {{ wishlist.has(product.id) ? '★' : '☆' }}
              </button>
            </div>

            <div class="card-body">
              <p class="category">{{ product.category }}</p>
              <h2>{{ product.title }}</h2>
              <p class="subtitle">{{ product.subtitle }}</p>

              <div class="price-row">
                <p class="current-price">
                  ₩{{ product.currentPrice.toLocaleString() }}
                </p>
                <p class="meta">
                  <span class="discount">{{ product.discountRate }}% OFF</span>
                  <span class="original">
                    ₩{{ product.originalPrice.toLocaleString() }}
                  </span>
                </p>
              </div>

              <div class="progress">
                <div class="progress-head">
                  <span>{{ product.currentCount }}명 참여</span>
                  <span>목표 {{ product.targetCount }}명</span>
                </div>
                <div class="progress-bar">
                  <div
                    class="progress-fill"
                    :style="{ width: `${Math.min(product.currentCount / product.targetCount * 100, 100)}%` }"
                  ></div>
                </div>
              </div>

              <div class="card-footer">
                <span class="time">⏰ {{ product.timeLeft }}</span>
                <div class="footer-actions">
                  <!-- <button 
                    class="btn btn-outline btn-sm"
                    @click.stop="addToCart(product)"
                  >
                    장바구니
                  </button> -->
                  <button 
                    class="btn btn-primary btn-sm"
                    @click.stop="goToDetail(product.id)"
                  >
                    참가하기
                  </button>
                </div>
              </div>
            </div>
          </article>
        </div>
      </div>
    </section>
    <div v-if="totalPages > 1" class="pagination">
      <button
        class="page-btn"
        :disabled="page === 0"
        @click="goToPage(page - 1)"
      >
        이전
      </button>

      <span class="page-info">
        {{ page + 1 }} / {{ totalPages }}
      </span>

      <button
        class="page-btn"
        :disabled="page + 1 >= totalPages"
        @click="goToPage(page + 1)"
      >
        다음
      </button>
    </div>
  </main>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { groupPurchaseApi } from '@/api/axios'
import { authAPI as auth } from '@/api/auth'

const route = useRoute()
const router = useRouter()

/* ======================
 * 상태
 * ====================== */
const products = ref([])
const loading = ref(false)
const recommendedProducts = ref([])
const recommendedReason = ref('')
const recommendedLoading = ref(false)

const keyword = ref('')
const selectedStatus = ref('OPEN')
const selectedCategory = ref('')

/* ✅ 정렬 상태 추가 */
const sortBy = ref('popular')

/* ======================
 * 위시리스트
 * ====================== */
const wishlist = ref(new Set())

const toggleWishlist = (id) => {
  const next = new Set(wishlist.value)
  next.has(id) ? next.delete(id) : next.add(id)
  wishlist.value = next
}

/* ======================
 * STATUS
 * ====================== */
const primarySections = [
  { id: 'OPEN', label: '진행중' },
  { id: 'SCHEDULED', label: '진행전' },
  { id: 'SUCCESS', label: '완료' },
  { id: 'FAILED', label: '실패' }
]

/* ======================
 * CATEGORY (더보기)
 * ====================== */
const allCategories = [
  { value: '', label: '전체', icon: '✨' },
  { value: 'HOME', label: '생활·주방', icon: '🏠' },
  { value: 'FOOD', label: '식품·간식', icon: '🍎' },
  { value: 'HEALTH', label: '건강·헬스', icon: '💊' },
  { value: 'BEAUTY', label: '뷰티', icon: '💄' },
  { value: 'FASHION', label: '패션·의류', icon: '👟' },
  { value: 'ELECTRONICS', label: '전자·디지털', icon: '📱' },
  { value: 'KIDS', label: '유아·어린이', icon: '🧸' },
  { value: 'HOBBY', label: '취미', icon: '🎮' },
  { value: 'PET', label: '반려동물', icon: '🐶' }
]

const showMoreCategories = ref(false)

const toggleMoreCategories = () => {
  showMoreCategories.value = !showMoreCategories.value
}

// 기본: 전체 + 3개
const primaryCategories = computed(() => allCategories.slice(0, 4))
// 더보기 영역
const secondaryCategories = computed(() => allCategories.slice(4))

const filterByCategory = (value) => {
  selectedCategory.value = value
  loadProducts()
  fetchRecommendedProducts()
}

// 카테고리별 기본 이미지
const categoryImages = {
  'HOME': 'https://images.unsplash.com/photo-1513694203232-719a280e022f?w=400',
  'FOOD': 'https://images.unsplash.com/photo-1504674900247-0877df9cc836?w=400',
  'HEALTH': 'https://images.unsplash.com/photo-1571019614242-c5c5dee9f50b?w=400',
  'BEAUTY': 'https://images.unsplash.com/photo-1596462502278-27bfdc403348?w=400',
  'FASHION': 'https://images.unsplash.com/photo-1445205170230-053b83016050?w=400',
  'ELECTRONICS': 'https://images.unsplash.com/photo-1468495244123-6c6c332eeece?w=400',
  'KIDS': 'https://images.unsplash.com/photo-1515488042361-ee00e0ddd4e4?w=400',
  'HOBBY': 'https://images.unsplash.com/photo-1452860606245-08befc0ff44b?w=400',
  'PET': 'https://images.unsplash.com/photo-1450778869180-41d0601e046e?w=400'
}

// 카테고리 한글 변환 맵
const categoryMap = {
  'HOME': '생활 & 주방',
  'FOOD': '식품 & 간식',
  'HEALTH': '건강 & 헬스',
  'BEAUTY': '뷰티',
  'FASHION': '패션 & 의류',
  'ELECTRONICS': '전자 & 디지털',
  'KIDS': '유아 & 어린이',
  'HOBBY': '취미',
  'PET': '반려동물'
}

/* ======================
 * ✅ 정렬 매핑 (서버 sort 파라미터)
 * ====================== */
const toApiSort = (key) => {
  switch (key) {
    case 'discountRate':
      return 'discountRate,desc'
    case 'priceLow':
      return 'discountedPrice,asc'
    case 'priceHigh':
      return 'discountedPrice,desc'
    case 'deadline':
      return 'endDate,asc'
    default:
      // 인기순(참여자수 기준)
      return 'currentQuantity,desc'
  }
}

/* ======================
 * UTIL
 * ====================== */
const getTimeLeft = (endDate) => {
  const diff = new Date(endDate) - new Date()
  if (diff <= 0) return '종료됨'
  const h = Math.floor(diff / 36e5)
  const d = Math.floor(h / 24)
  return d > 0 ? `${d}일 ${h % 24}시간 남음` : `${h}시간 남음`
}

const getCategoryFallbackImage = (product) => {
  const key = product?.rawCategory || ''
  return categoryImages[key] || categoryImages['PET']
}

const handleProductImageError = (e, product) => {
  const img = e?.target
  if (!img) return
  img.onerror = null
  img.src = getCategoryFallbackImage(product)
}

/* ======================
 * TRANSFORM
 * ====================== */

//형식 맞추기
const mapToProductCard = (gp) => {
  const p = gp.productSearchInfo || {}
  const categoryCode = p.category || 'PET'

  // ✅ 이미지 우선순위: 공동구매 이미지 → 상품 이미지 → 카테고리 기본 이미지
  let image = gp.imageUrl
    || p.imageUrl
    || p.image
    || p.thumbnailUrl
    || categoryImages[categoryCode]
  if (!image || image.trim() === '') {
    image = categoryImages[categoryCode]
  }

  const originalPrice = p.price || 0
  const discountedPrice = gp.discountedPrice || originalPrice
  const discountRate =
    originalPrice > 0
      ? Math.round(((originalPrice - discountedPrice) / originalPrice) * 100)
      : 0

  // 카테고리를 한글로 변환
  const categoryKorean = categoryMap[categoryCode] || categoryCode

  return {
    id: gp.groupPurchaseId,
    title: gp.title,
    subtitle: '',
    category: categoryKorean,
    rawCategory: categoryCode,
    image,
    originalPrice,
    currentPrice: discountedPrice,
    discountRate,
    currentCount: gp.currentQuantity || 0,
    targetCount: gp.maxQuantity || 1,
    timeLeft: getTimeLeft(gp.endDate),
    badges: [gp.status]
  }
}

const fetchRecommendedProducts = async () => {
  recommendedLoading.value = true
  try {
    const res = await auth.searchAIRecommendPurchases({
      keyword: keyword.value,
      category: selectedCategory.value
    })
    const items = res?.groupPurchase ?? []
    recommendedProducts.value = items.slice(0, 3).map(mapToProductCard)
    recommendedReason.value = res?.reason ?? ''
    console.log('추천 상품 검색 완료,{}', recommendedProducts.value)
  } catch (e) {
    console.error('추천 상품 조회 실패', e)
    recommendedProducts.value = []
    recommendedReason.value = ''
  } finally {
    recommendedLoading.value = false
  }
}

/* ======================
 * ES SEARCH
 * ====================== */
const loadProducts = async () => {
  loading.value = true
  try {
    const res = await groupPurchaseApi.searchGroupPurchases({
                         keyword: keyword.value,
                         status: selectedStatus.value,
                         category: selectedCategory.value,
                         page: page.value,
                         size: size.value,
                         sort: toApiSort(sortBy.value)
                       })

    const pageData = res.data?.data

    products.value = (pageData?.content ?? []).map(mapToProductCard)
    totalPages.value = pageData?.totalPages ?? 0
  } catch (e) {
    console.error('공동구매 검색 실패', e)
    products.value = []
  } finally {
    loading.value = false
  }
}

/* ======================
 * PAGING
 * ====================== */
const page = ref(0)          // 현재 페이지 (0부터 시작)
const size = ref(9)         // 페이지 사이즈
const totalPages = ref(0)

const goToPage = (nextPage) => {
  if (nextPage < 0 || nextPage >= totalPages.value) return
  page.value = nextPage
  loadProducts()
}

const resetPageAndReload = () => {
  page.value = 0
  loadProducts()
}

watch(sortBy, resetPageAndReload)
watch(selectedStatus, resetPageAndReload)
watch(selectedCategory, resetPageAndReload)

/* ======================
 * EVENTS
 * ====================== */
const setStatus = (status) => {
  selectedStatus.value = status
  loadProducts()
}

const search = () => {
  loadProducts()
  fetchRecommendedProducts()
}

const goToDetail = (id) => {
  router.push({ name: 'group-purchase-detail', params: { id } })
}

// const addToCart = async (product) => {
//   try {
//     const response = await cartApi.addToCart({
//       groupPurchaseId: product.id,
//       quantity: 1
//     })
//     console.log('장바구니 추가 성공:', response.data)
//     alert('장바구니에 담았습니다.')
//     // FloatingCart 업데이트 이벤트 발생
//     window.dispatchEvent(new CustomEvent('cart-updated'))
//   } catch (error) {
//     console.error('장바구니 담기 실패:', error)
//     const message = error.response?.data?.message || '장바구니 담기에 실패했습니다.'
//     alert(message)
//   }
// }

watch(
  () => route.query,
  (query) => {
    // 🔍 검색어
    keyword.value = query.q ?? ''

    // 🗂 카테고리
    selectedCategory.value = query.category ?? ''

    // 📄 페이지 초기화
    page.value = 0

    // 🔥 실제 검색 실행
    loadProducts()
    fetchRecommendedProducts()
  },
  { immediate: true }
)

/* ======================
 * INIT
 * ====================== */
onMounted(() => {
  loadProducts()
  fetchRecommendedProducts()
})
</script>

<style scoped>
.products-page {
  background: var(--bg);
  min-height: 100vh;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.page-hero {
  padding: 48px 0 32px;
}

.page-hero .eyebrow {
  color: var(--text);
  font-weight: 600;
  letter-spacing: 0.04em;
  text-transform: uppercase;
  margin-bottom: 8px;
}

.page-hero h1 {
  font-size: 36px;
  margin-bottom: 12px;
  color: var(--text);
}

.page-hero .subtitle {
  color: var(--muted);
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 24px;
}

.section-title {
  font-size: 24px;
  font-weight: 700;
  margin: 0;
  color: var(--text);
}

.section-reason {
  margin-left: 12px;
  font-size: 13px;
  font-weight: 500;
  color: var(--muted);
}

.skeleton-card {
  pointer-events: none;
}

.skeleton-block {
  background: linear-gradient(90deg, #1f1f1f 0%, #2a2a2a 50%, #1f1f1f 100%);
  background-size: 200% 100%;
  animation: skeleton-shimmer 1.2s ease-in-out infinite;
}

.skeleton-line {
  height: 12px;
  border-radius: 8px;
  margin-bottom: 10px;
  background: linear-gradient(90deg, #1f1f1f 0%, #2a2a2a 50%, #1f1f1f 100%);
  background-size: 200% 100%;
  animation: skeleton-shimmer 1.2s ease-in-out infinite;
}

.skeleton-line.short {
  width: 60%;
}

@keyframes skeleton-shimmer {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}

.recommend-section {
  padding: 24px 0 40px;
}

.filters {
  background: var(--bg);
  border-top: 1px solid #2a2a2a;
  border-bottom: 1px solid #2a2a2a;
  padding: 24px 0;
}

.filter-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 16px;
}

.filter-row.secondary {
  margin-top: 12px;
  justify-content: space-between;
  align-items: flex-start;
}

.chips {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
  flex: 1;
}

.chip {
  padding: 8px 16px;
  border-radius: 999px;
  border: 1px solid var(--border);
  background: var(--surface);
  color: var(--text);
  cursor: pointer;
  font-weight: 500;
  font-size: 14px;
  transition: all 0.2s;
}

.chip.active {
  background: var(--text);
  color: var(--bg);
  border-color: var(--text);
  font-weight: 600;
}

.filter-actions {
  display: flex;
  gap: 12px;
  align-items: center;
}

.search {
  display: flex;
  gap: 8px;
}

.search input {
  padding: 10px 14px;
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 8px;
  min-width: 260px;
  color: var(--text);
}

.search input::placeholder {
  color: var(--muted);
}

.search input:focus {
  outline: none;
  border-color: var(--text);
  background: #151515;
}

.category-wrapper {
  display: flex;
  flex-direction: column;
}

.categories {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.categories.primary {
  margin-bottom: 8px;
}

.categories.secondary {
  margin-top: 4px;
}

.category-btn {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 10px 14px;
  border-radius: 999px;
  border: 1px solid var(--border);
  background: var(--surface);
  color: var(--text);
  cursor: pointer;
}

.category-btn.active {
  background: var(--text);
  color: var(--bg);
  border-color: var(--text);
  font-weight: 600;
}

.category-btn.more {
  opacity: 0.8;
}

/* ✅ 정렬 스타일 */
.sort {
  color: var(--text);
  display: flex;
  align-items: center;
  gap: 8px;
  padding-top: 4px;
}

.sort select {
  margin-left: 8px;
  padding: 10px 14px;
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 10px;
  color: var(--text);
  cursor: pointer;
  min-width: 160px;
}

.sort select:focus {
  outline: none;
  border-color: var(--text);
  background: #151515;
}

.product-grid-section {
  padding: 40px 0 80px;
}

.product-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 24px;
}

.product-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: 20px;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.3);
  cursor: pointer;
  transition: all 0.3s;
}

body.theme-light .products-page,
body.theme-light .filters,
body.theme-light .recommend-section,
body.theme-light .product-grid-section {
  background: #ffffff;
  color: #111111;
  border-color: #e2e2e7;
}

body.theme-light .chip,
body.theme-light .category-btn {
  background: #ffffff;
  color: #111111;
  border-color: #d0d0d6;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
}

/* 라이트 모드에서 선택된 버튼 - 최고 우선순위 */
:global(body.theme-light) .chip.active,
:global(body.theme-light) .category-btn.active {
  background: #111111 !important;
  color: #ffffff !important;
  border-color: #111111 !important;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15) !important;
  font-weight: 600 !important;
}

/* 모든 자식 요소에도 하얀색 강제 적용 */
:global(body.theme-light) .chip.active,
:global(body.theme-light) .chip.active *,
:global(body.theme-light) .chip.active span,
:global(body.theme-light) .category-btn.active,
:global(body.theme-light) .category-btn.active *,
:global(body.theme-light) .category-btn.active span {
  color: #ffffff !important;
}

body.theme-light .search input,
body.theme-light .sort select {
  background: #ffffff;
  color: #111111;
  border-color: #d0d0d6;
}

body.theme-light .search input:focus,
body.theme-light .sort select:focus {
  border-color: #111111;
  background: #ffffff;
}

body.theme-light .sort {
  color: #111111;
}

body.theme-light .btn-primary {
  background: #111111;
  color: #ffffff;
  border-color: #111111;
}

body.theme-light .btn-primary:hover {
  background: #2a2a2a;
  border-color: #2a2a2a;
}

body.theme-light .product-card {
  background: #ffffff;
  color: #111111;
  border-color: #e2e2e7;
  box-shadow: 0 12px 30px rgba(0, 0, 0, 0.08);
}

body.theme-light .product-category,
body.theme-light .product-title,
body.theme-light .price-row,
body.theme-light .original-price,
body.theme-light .discount-rate,
body.theme-light .current-price,
body.theme-light .progress-text,
body.theme-light .progress-target,
body.theme-light .time,
body.theme-light .section-title {
  color: #111111;
}

body.theme-light .original-price,
body.theme-light .discount-rate {
  color: #666666;
}

body.theme-light .progress-bar {
  background: #e6e6ea;
}

body.theme-light .progress-fill {
  background: #111111;
}

.product-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 16px 40px rgba(0, 0, 0, 0.5);
  border-color: #3a3a3a;
}

.image-wrapper {
  position: relative;
  padding-top: 72%;
}

.image-wrapper img {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.badge-group {
  position: absolute;
  top: 12px;
  left: 12px;
  display: flex;
  gap: 6px;
}

.badge {
  background: rgba(0, 0, 0, 0.65);
  color: white;
  padding: 4px 10px;
  border-radius: 12px;
  font-size: 12px;
}

body.theme-light .badge {
  background: rgba(0, 0, 0, 0.65);
  color: #ffffff;
}

.bookmark {
  position: absolute;
  top: 12px;
  right: 12px;
  background: rgba(255, 255, 255, 0.8);
  border: none;
  border-radius: 50%;
  width: 36px;
  height: 36px;
  font-size: 18px;
  cursor: pointer;
}

.bookmark.active {
  color: #f59f00;
}

.card-body {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.card-body h2 {
  font-size: 20px;
  margin: 0;
  color: var(--text);
}

.card-body .subtitle {
  color: var(--muted);
  font-size: 14px;
}

.category {
  color: var(--text);
  font-weight: 600;
  font-size: 13px;
}

.price-row {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  gap: 12px;
}

.current-price {
  font-size: 24px;
  font-weight: 700;
  color: var(--text);
}

.meta {
  font-size: 13px;
  color: var(--muted);
}

.discount {
  color: var(--text);
  font-weight: 600;
  margin-right: 8px;
}

.original {
  text-decoration: line-through;
  color: var(--muted);
}

.progress-head {
  display: flex;
  justify-content: space-between;
  font-size: 13px;
  color: var(--text);
}

.progress-bar {
  height: 8px;
  background: var(--bg);
  border-radius: 999px;
  overflow: hidden;
  margin-top: 8px;
}

.progress-fill {
  height: 100%;
  background: #ffffff;
}

.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.footer-actions {
  display: flex;
  gap: 8px;
}

.time {
  font-size: 13px;
  color: var(--muted);
}

.btn {
  border: none;
  border-radius: 10px;
  padding: 10px 16px;
  cursor: pointer;
  font-weight: 600;
}

.btn-primary {
  background: #ffffff;
  color: #0a0a0a;
  border: 1px solid var(--border);
}

.btn-primary:hover {
  background: #f0f0f0;
  border-color: var(--text);
}

.btn-outline {
  border: 1px solid #3a3a3a;
  background: transparent;
  color: var(--text);
}

.btn-outline:hover {
  background: var(--hover);
  border-color: #4a4a4a;
}

.btn-sm {
  padding: 8px 16px;
  font-size: 14px;
}

.empty-state {
  text-align: center;
  padding: 80px 0;
  color: var(--text);
}

.empty-state p {
  color: var(--muted);
  margin-bottom: 16px;
}

/* ===== Pagination ===== */
  .pagination {
    margin-top: 48px;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 16px;
  }

  /* 버튼 */
  .page-btn {
    padding: 10px 18px;
    border-radius: 999px;
    border: 1px solid var(--border);
    background: var(--surface);
    color: var(--text);
    font-weight: 600;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .page-btn:hover:not(:disabled) {
    background: #ffffff;
    color: #0a0a0a;
    border-color: var(--text);
  }

  .page-btn:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }

  /* 페이지 정보 */
  .page-info {
    min-width: 80px;
    text-align: center;
    font-weight: 600;
    color: var(--text);
  }

@media (max-width: 768px) {
  .filter-row {
    flex-direction: column;
  }

  .filter-row.secondary {
    align-items: stretch;
  }

  .sort {
    width: 100%;
  }

  .sort select {
    width: 100%;
    margin-left: 0;
  }

  .search {
    width: 100%;
  }

  .search input {
    flex: 1;
    min-width: auto;
  }

  .price-row {
    flex-direction: column;
    align-items: flex-start;
  }
}
</style>

<style>
/* 라이트 모드에서 선택된 버튼 - 전역 스타일로 최고 우선순위 적용 */
body.theme-light .chip.active,
body.theme-light .category-btn.active {
  background: #111111 !important;
  color: #ffffff !important;
  border-color: #111111 !important;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15) !important;
  font-weight: 600 !important;
}

/* 모든 자식 요소에도 하얀색 강제 적용 */
body.theme-light .chip.active,
body.theme-light .chip.active *,
body.theme-light .chip.active span,
body.theme-light .category-btn.active,
body.theme-light .category-btn.active *,
body.theme-light .category-btn.active span {
  color: #ffffff !important;
}
</style>
