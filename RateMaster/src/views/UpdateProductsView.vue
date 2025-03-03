<script setup>
import CustomButton from '@/components/CustomButton.vue';
import DAOService from '@/services/DAOService';
import { ref, onMounted } from 'vue';
import { useRoute, useRouter } from 'vue-router';

const route = useRoute();
const router = useRouter();

const alertMessage = ref(null);
const alertType = ref('success');
const showAlert = ref(false);

const daoProducts = new DAOService('products');
const daoBrands = new DAOService('brands');

const brands = ref([]);

const loadBrands = async () => {
  brands.value = await daoBrands.getAll();
};

const product = ref({
  name: '',
  description: '',
  brand: '',
  price: '',
  type: '',
  image: null,
});

const triggerAlert = (message, type = 'success') => {
    alertMessage.value = message;
    alertType.value = type;
    showAlert.value = true;
    setTimeout(() => (showAlert.value = false), 3000);
};

const fetchProduct = async () => {
  const productId = route.params.id;

  if (!productId) {
    triggerAlert('Produto não encontrado!','warning');
    router.push({ name: 'listProducts' });
    return;
  }

  try {
    const fetchedProduct = await daoProducts.get(productId);
    product.value = { ...fetchedProduct };
  } catch (error) {
    console.error(error);
    triggerAlert('Erro ao carregar o produto.', 'danger');
    router.push({ name: 'listProducts' });
  }
};


const submit = async () => {
  if (!product.value.name || !product.value.description || !product.value.price || !product.value.brand || !product.value.type) {
    triggerAlert('Por favor, preencha todos os campos obrigatórios!','warning');
    return;
  }

  try {
    await daoProducts.update(product.value.id, product.value);
    triggerAlert('Produto atualizado com sucesso!', 'success');
    router.push("/myProfile")
  } catch (error) {
    console.error(error);
    triggerAlert('Erro ao atualizar o produto.', 'danger');
  }
};

const image = ref('');

const handleImageUpload = (event) => {
  const file = event.target.files[0];
  if (file) {
    const reader = new FileReader();
    reader.onload = () => {
      image.value = reader.result;
      product.value.image = image.value;
    };
    reader.readAsDataURL(file);
  }
};

const formatPrice = (event) => {
  let value = event.target.value.replace(/\D/g, '');
  if (value.length > 2) {
    value = value.slice(0, value.length - 2) + ',' + value.slice(value.length - 2);
  }
  product.value.price = value ? 'R$ ' + value : '';
};

onMounted(() => {
  loadBrands();
  fetchProduct();
});
</script>

<template>
   <div class="content container-fluid mt-5">
        <div v-if="showAlert" :class="`alert alert-${alertType} alert-dismissible fade show custom-alert`" role="alert">
            <i v-if="alertType === 'success'" class="bi bi-check-circle-fill"></i>
            <i v-if="alertType === 'warning'" class="bi bi-exclamation-triangle-fill"></i>
            <i v-if="alertType === 'danger'" class="bi bi-x-circle-fill"></i>
            {{ alertMessage }}
            <button type="button" class="btn-close" @click="showAlert = false"></button>
        </div>
    </div>
    <div class="container mt-5">
      <button type="button" class="btn pb-3" @click="router.push('/Myprofile')">
      ← Voltar para os produtos
    </button>
      <div class="row">
        <div class="col-md-6">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">Anexar imagem</h5>
              <input type="file" @change="handleImageUpload" class="form-control" />
              <div v-if="image" class="mt-3">
                <img :src="image" alt="Product Preview" class="img-fluid" />
              </div>
            </div>
          </div>
        </div>

        <div class="col-md-6">
          <div class="card">
            <div class="card-body">
              <h5 class="card-title">Editar Produto</h5>
              <form @submit.prevent="submit">

                <div class="mb-3">
                  <label for="productName" class="form-label">Nome do produto</label>
                  <input v-model="product.name" type="text" class="form-control" placeholder="Digite o nome do produto."
                    required />
                </div>

                <div class="mb-3">
                  <label for="description" class="form-label">Descrição</label>
                  <input v-model="product.description" type="text" class="form-control"
                    placeholder="Digite a descrição do seu produto" required />
                </div>

                <div class="mb-3">
                  <label for="price" class="form-label">Preço</label>
                  <input v-model="product.price" type="text" class="form-control" placeholder="R$ 0,00" required
                    @input="formatPrice" />
                </div>

                <div class="mb-3">
                  <label for="brand" class="form-label">Marca</label>
                  <div class="d-flex align-items-center">
                    <select v-model="product.brand" class="form-select me-2" required>
                      <option v-for="brand in brands" :key="brand.id" :value="brand.id">
                        {{ brand.name }}
                      </option>
                    </select>
                    <RouterLink to="/brand">
                      <CustomButton class="btn btn-primary">
                        <i class="bi bi-plus-square-fill"></i>
                      </CustomButton>
                    </RouterLink>
                  </div>
                </div>

                <div class="mb-3">
                  <label for="type" class="form-label">Tipo</label>
                  <select v-model="product.type" class="form-select" required>
                    <option value="Produto">Produto</option>
                    <option value="Serviço">Serviço</option>
                  </select>
                </div>
                <CustomButton class="button container-fluid">Salvar</CustomButton>
              </form>
            </div>
          </div>
        </div>
      </div>
    </div>
</template>

<style scoped>
.card {
  margin-bottom: 20px;
}

.card-title {
  font-size: 1.25rem;
  font-weight: bold;
}

img {
  max-height: 200px;
  object-fit: cover;
  border-radius: 5px;
}

button:hover {
  transform: scale(1.01);
}
.custom-alert {
    display: flex;
    align-items: center;
    gap: 10px;
    animation: fadeIn 0.5s;
    margin-bottom: 20px;
    font-size: 1rem;
}

@keyframes fadeIn {
    from {
        opacity: 0;
    }
    to {
        opacity: 1;
    }
}
</style>
