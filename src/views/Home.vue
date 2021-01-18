<template>
  <section class="main">
      <!--Header-->
    <header class="header">
        <!--Logotipo-->
      <img class="logo" src="../../public/images/techno.svg" alt="Techno">
      <div class="carrinho_menu" @click="carrinhoAtivo = true">{{carrinhoTotal | numeroPreco}} | {{carrinho.length}}</div>
    </header>

    <section class="produtos">
      <div v-for="(item, index) in produtos.data" :key="index" @click.prevent="abrirModal(item.id)" class="produto">
        <img :src="item.path" :alt="item.nome" class="produto_img" />
        <span class="produto_preco">{{ item.preco | numeroPreco }}</span>
        <h2 class="produto_titulo">{{ item.nome }}</h2>
      </div>
    </section>

  <!-- Modal -->
    <section class="modal" v-if="produto" @click.prevent="fecharModal">
      <div class="modal_container">
        <div class="modal_img">
          <img :src="produto.data.pathImg" alt="produto.nome">
        </div>

        <div class="modal_dados">
          <button @click="produto = false" class="modal_fechar">X</button>
          <span class="modal_preco">{{ produto.data.preco | numeroPreco }}</span>
          <h2 class="modal_titulo">{{ produto.data.nome }}</h2>
          <p>{{ produto.data.descricao }}</p>

          <!-- Adicionar Carrinho -->
          <button v-if="produto.data.estoque > 0" class="modal_btn" @click="adicionarItem">Adicionar Item</button>
          <button v-else class="modal_btn esgotado" disabled>Produto Esgotado</button>
          <p>{{ produto.data.estoque }}</p>
        </div>

        <div class="avaliacoes">
          <h2 class="avaliacoes_subtitulo">Avaliações</h2>
          <ul>
            <li v-for="(avaliacao, index) in produto.data.avaliacoes" :key="index" class="avaliacao">
              <p class="avaliacao_descricao">{{ avaliacao.descricao }}</p>
              <p>{{ avaliacao.nome }} | {{ avaliacao.estrelas }} estrelas</p>
            </li>
          </ul>
        </div>
      </div>
    </section>

    <!--Carrinho Modal-->
    <section class="carrinho_modal" :class="{ativo: carrinhoAtivo}" @click="clickForaCarrinho">
      <div class="carrinho_container">
        <button class="carrinho_fechar" @click="carrinhoAtivo = false">X</button>
        <h2 class="carrinho_titulo">Carrinho</h2>
        <div v-if="carrinho.length > 0">
          <ul class="carrinho_lista">
            <li v-for="(item, index) in carrinho" :key="index" class="carrinho_item">
              <p>{{item.nome}}</p>
              <p class="carrinho_preco">{{item.preco | numeroPreco}}</p>
              <button class="carrinho_remover" @click="removerItem(index)">X</button>
            </li>
          </ul>
          <p class="carrinho_total">{{carrinhoTotal | numeroPreco}}</p>
          <button class="carrinho_finalizar">Finalizar Compra</button>
        </div>
        <p v-else>O carrinho está vazio.</p>
      </div>
    </section>


    <div class="alerta" :class="{ativo: alertaAtivo}">
      <p class="alerta_mensagem">{{mensagemAlerta}}</p>
    </div>
  </section>
</template>

<script>

export default {
  name: 'Home',
  data() {
    return {
      produtos: [],
      produto: false,
      carrinho: [],
      carrinhoAtivo: false,
      mensagemAlerta: "Item adicionado",
      alertaAtivo: false,
    };
  },
  created() {
    /*Momento da criação - ativa os seguintes métodos*/
    this.axiosProducts();
    this.router();
    this.checarLocalStorage();
  },
  filters: {
    numeroPreco(valor) {
      return valor.toLocaleString("pt-BR", { style: "currency", currency: "BRL" });
    }
  },
  computed: {
    carrinhoTotal() {
      let total = 0;
      if (this.carrinho.length) {
        this.carrinho.forEach(item => {
          total += item.preco;
        })
      }
      return total;
    }
  },
  methods: {
    axiosProducts() {
      /*CHAMADA API FAKE
      * é necessário instalar o json-server para poder consumir essa api fake.
      * Funciona com fetch e axios*/
      this.axios.get("http://localhost:3000/produtos")
          .then(response => (this.produtos = response))
    },
    axiosProduct(id) {
      this.axios.get(`http://localhost:3000/produtos/${id}`)
          .then(response => (this.produto = response))
    },
    abrirModal(id) {
      this.axiosProduct(id);
      window.scrollTo({
        top: 0,
        behavior: "smooth"
      })
    },
    fecharModal({ target, currentTarget }) {
      if (target === currentTarget) this.produto = false;
    },
    clickForaCarrinho({ target, currentTarget }) {
      if (target === currentTarget) this.carrinhoAtivo = false;
    },
    adicionarItem() {
      this.produto.data.estoque--;
      const { id, nome, preco } = this.produto.data;
      this.carrinho.push({ id, nome, preco });
      this.alerta(`${nome} adicionado ao carrinho.`);
    },
    removerItem(index) {
      this.carrinho.splice(index, 1);
    },
    /*Se o meu localStorage tiver carrinho
    * meu carrinho será tranformado de volta no que era antes, em uma array, fazemos isso com parse*/
    checarLocalStorage() {
      if (window.localStorage.carrinho)
        this.carrinho = JSON.parse(window.localStorage.carrinho);
    },
    compararEstoque() {
      const items = this.carrinho.filter(({ id }) => id === this.produto.id);
      this.produto.data.estoque -= items.length;
    },
    alerta(mensagem) {
      this.mensagemAlerta = mensagem;
      this.alertaAtivo = true;
      setTimeout(() => {
        this.alertaAtivo = false;
      }, 1500);
    },
    router() {
      const hash = document.location.hash;
      if (hash)
        this.axiosProduct(hash.replace("#", ""));
    }
  },
  watch: {
    produto() {
      document.title = this.produto.data.nome || "Techno";
      const hash = this.produto.data.id || "";
      history.pushState(null, null, `#${hash}`);
      /*if (this.produto) {
        this.compararEstoque();
      }*/
    },
    /*Transforma minha array carrinho em uma string com stringfy*/
    carrinho() {
      window.localStorage.carrinho = JSON.stringify(this.carrinho);
    }
  },
}
</script>

<style scoped>
/* HEADER */

.header {
  display: flex;
  justify-content: space-between;
  max-width: 900px;
  padding: 20px 0;
  margin: 0 auto;
}

.logo {
  width: 80px;
}

.carrinho_menu::after {
  content: "";
  display: inline-block;
  background: url("../assets/img/carrinho.svg") no-repeat center center;
  width: 25px;
  height: 25px;
  margin-left: 10px;
}

.carrinho_menu {
  display: flex;
  align-items: center;
  cursor: pointer;
}

/* LISTA PRODUTOS */

.produtos {
  max-width: 900px;
  margin: 100px auto 0 auto;
}

.produto {
  display: flex;
  align-items: center;
  margin-bottom: 40px;
  background: #ffffff;
  box-shadow: 0 0 2rem rgba(0,0,0,.1);
  cursor: pointer;
}

.produto_img {
  max-width: 300px;
  margin-right: 40px;
}

.produto_titulo {
  font-size: 3rem;
  line-height: 2;
}

.produto_preco {
  color: rgba(0,0,0,.5);
}

/* MODAL */

.modal::before {
  content: "";
  position: fixed;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100vh;
  background: rgba(0,0,0,.5);
}

.modal {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: absolute;
  top: 0px;
  left: 0px;
  width: 100%;
  padding: 80px;
}

.modal_container {
  position: relative;
  background: linear-gradient(to right, transparent 250px, white 250px);
  z-index: 1;
  display: grid;
  align-items: end;
  grid-gap: 50px;
  padding: 50px 50px 50px 0;
  animation: fadeIn .3s forwards;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translate3d(50px, 0, 0);
  }
  to {
    opacity: 1;
    transform: translate3d(0, 0, 0);
  }
}

.modal_fechar {
  border-radius: 50%;
  border: 2px solid #000;
  width: 40px;
  height: 40px;
  position: absolute;
  top: -10px;
  right: -10px;
  font-size: 1rem;
  box-shadow: 0px 3px 4px rgba(0,0,0,.1), 0px 4px 10px rgba(0,0,0,.2);
  cursor: pointer;
}

.modal_img {
  grid-column: 1;
  box-shadow: 0px 3px 4px rgba(0,0,0,.1), 0px 4px 10px rgba(0,0,0,.2);
}

.modal_img img {
  max-width: 300px;
  display: block;
}

.modal_dados {
  grid-column: 2;
  max-width: 600px;
}

.modal_titulo {
  font-size: 3rem;
}

.modal_btn {
  margin-top: 80px;
  background: #000;
  cursor: pointer;
  color: #ffffff;
  border: none;
  font-size: 1rem;
  padding: 10px 25px;
  font-family: "Noto Serif", sans-serif;
}

.modal_btn.esgotado {
  background: #808080;
}

.modal_btn:active {
  background: #808080;
}

/* AVALIACOES */

.avaliacoes {
  grid-column: 2;
}

.avaliacoes_subtitulo {
  font-size: 1.75rem;
}

.avaliacao {
  border-bottom: 1px solid rgba(0,0,0,.1);
  padding-bottom: 20px;
}

.avaliacao_descricao {
  color: rgba(0,0,0,.7);
  margin: 20px 0 5px 0;
}

.avaliacao_usuario {
  font-weight: bold;
}

/* ALERTA */

.alerta {
  position: absolute;
  top: 20px;
  left: 0px;
  z-index: 10;
  width: 100%;
  text-align: center;
  display: none;
}

.alerta.ativo {
  display: block;
  animation: fadeInDown .3s forwards;
}

@keyframes fadeInDown {
  from {
    transform: translate3d(0, -30px, 0);
    opacity: 0;
  }
  to {
    opacity: 1;
    transform: translate3d(0, 0px, 0);
  }
}

.alerta_mensagem {
  background: #ffffff;
  border-radius: 5px;
  display: inline-block;
  padding: 20px 40px;
  border: 2px solid black;
  box-shadow: 0px 3px 4px rgba(0,0,0,.1), 0px 4px 10px rgba(0,0,0,.2);
}

/* CARRINHO */

.carrinho_modal::before {
  content: "";
  position: fixed;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100vh;
  background: rgba(0,0,0,.5);
}

.carrinho_modal {
  position: absolute;
  display: flex;
  flex-direction: column;
  top: 0px;
  left: 0px;
  width: 100%;
  padding: 20px;
  display: none;
}

.carrinho_modal.ativo {
  display: block;
}

.carrinho_container {
  position: relative;
  margin: 80px auto;
  background: #ffffff;
  padding: 40px;
  max-width: 800px;
  animation: fadeInDown .3s forwards;
}

.carrinho_item {
  display: grid;
  grid-template-columns: 1fr 1fr 50px;
  border-bottom: 1px solid rgba(0,0,0,.1);
  padding: 10px 0;
}

.carrinho_titulo {
  margin-bottom: 10px;
  padding-bottom: 20px;
  border-bottom: 2px solid #000000;
}

.carrinho_remover {
  border: none;
  font-size: 1rem;
  cursor: pointer;
  margin-left: 5px;
}

.carrinho_preco {
  text-align: right;
}

.carrinho_total {
  text-align: right;
  padding: 10px 50px 10px 0;
  margin-bottom: 20px;
  border-bottom: 2px solid #000;
}

.carrinho_fechar {
  border-radius: 50%;
  border: 2px solid #000;
  width: 40px;
  height: 40px;
  position: absolute;
  top: -10px;
  right: -10px;
  font-size: 1rem;
  box-shadow: 0px 3px 4px rgba(0,0,0,.1), 0px 4px 10px rgba(0,0,0,.2);
  cursor: pointer;
}

.carrinho_finalizar {
  display: block;
  margin-left: auto;
  background: #000;
  cursor: pointer;
  color: #ffffff;
  font-size: 1rem;
  padding: 10px 25px;
  border: none;
  font-family: "Noto Serif";
}

/* RESPONSIVO */

@media screen and (max-width: 900px) {
  #app {
    padding: 0 10px;
  }
  .produtos {
    margin-top: 40px;
  }
  .produto {
    flex-direction: column;
    align-items: flex-start;
    max-width: 300px;
    margin: 30px auto;
  }
  .produto_info {
    padding: 20px;
  }
  .produto_img {
    max-width: 100%;
  }
  .produto_titulo {
    font-size: 1.5rem;
  }
  .modal {
    padding: 10px;
  }
  .modal_container {
    grid-gap: 20px;
    background: #ffffff;
    padding: 10px 0;
  }
  .modal_img {
    grid-row: 2;
  }
  .modal_img img {
    width: 100%;
    max-width: initial;
  }
  .modal_dados {
    grid-column: 1;
    padding: 10px;
  }
  .modal_btn {
    margin-top: 20px;
  }
  .avaliacoes {
    grid-column: 1;
    padding: 10px;
  }
  .carrinho_modal {
    padding: 10px;
  }
}
</style>
