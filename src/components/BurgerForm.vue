<template>
  <div>
    <Message :msg="msg" v-show="msg" />
    <div>
      <form id="burger-form" @submit="createBurger">
        <div class="input-container">
          <label for="">Nome do cliente</label>
          <div>
            <input @blur="v$.nome.$touch" type="text" name="nome" id="nome" v-model="nome"
              placeholder="Digite o seu nome:">
            <div v-for="error of v$.nome.$errors" :key="error.$uid">
              <div class="error">{{ error.$message }}</div>
            </div>
          </div>
        </div>
        <div class="input-container">
          <label for="pao">Selecione o seu pão:</label>
          <select @blur="v$.pao.$touch" name="pao" id="pao" v-model="pao">
            <option value="">Selecione o tipo de pão</option>
            <option v-for="pao in paes" :key="pao.id" :value="pao.tipo">
              {{ pao.tipo }}
            </option>
          </select>
          <div v-for="error of v$.pao.$errors" :key="error.$uid">
            <div class="error">{{ error.$message }}</div>
          </div>
        </div>
        <div class="input-container">
          <label for="carne">Escolha a carne do seu Burger:</label>
          <select @blur="v$.carne.$touch" name="carne" id="carne" v-model="carne">
            <option value="">Selecione o tipo de carne</option>
            <option v-for="carne in carnes" :key="carne.id" :value="carne.tipo">
              {{ carne.tipo }}
            </option>
          </select>
          <div v-for="error of v$.carne.$errors" :key="error.$uid">
            <div class="error">{{ error.$message }}</div>
          </div>
        </div>
        <div id="opcionais-contaier" class="input-container">
          <label id="opcionais-title" for="opcionais">Selecione os opcionais:</label>
          <div class="checkbox-container" v-for="opcional in opcionaisdata" :key="opcional.id">
            <input type="checkbox" name="opcionais" :value="opcional.tipo" v-model="opcionais">
            <span>{{ opcional.tipo }}</span>
          </div>
        </div>
        <div class="input-container">
          <input type="submit" class="submit-btn" value="Criar meu Burger">
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import Message from "../components/Message.vue"
import { useVuelidate } from '@vuelidate/core'
import { required, maxLength, helpers } from '@vuelidate/validators'

const alphaPt = helpers.regex(/^[a-záàâãéèêíïóôõöúçñ ]+$/i)

export default {
  setup() {
    return { v$: useVuelidate() }
  },

  name: "BurgerForm",

  components: {
    Message,
  },

  data() {
    return {
      paes: null,
      carnes: null,
      opcionaisdata: null,
      nome: null,
      pao: null,
      carne: null,
      opcionais: [],
      msg: null,

    }
  },
  validations() {
    return {
      nome: {
        alphaPt: helpers.withMessage('Este campo só aceita letras', alphaPt),
        required: helpers.withMessage('Este campo não pode ser vazio', required),
        maxLength: helpers.withMessage(
          ({
            $invalid,
            $params,
            $model
          }) => `Este campo tem o valor de '${$model}' mas deve ter no máximo ${$params.max} caracteres, então isto é ${$invalid ? 'invalido' : 'valido'}`,
          maxLength(255)
        )
      },
      pao: {
        required: helpers.withMessage('É necessário selecionar um tipo de pão', required),
      },
      carne: {
        required: helpers.withMessage('É necessário selecionar um tipo de pão', required),
      }
    }
  },
  methods: {
    async getIngredientes() {
      const req = await fetch("http://localhost:3000/ingredientes");
      const data = await req.json();

      this.paes = data.paes;
      this.carnes = data.carnes;
      this.opcionaisdata = data.opcionais;
    },
    async createBurger(e) {
      e.preventDefault();

      const result = await this.v$.$validate()

      if (!result)
        return

      const data = {
        nome: this.nome,
        carne: this.carne,
        pao: this.pao,
        opcionais: Array.from(this.opcionais),
        status: "Solicitado"
      }

      const dataJson = JSON.stringify(data);

      const req = await fetch("http://localhost:3000/burgers", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: dataJson,
      });

      const res = await req.json();

      console.log(res);

      //Colocar uma msg de sistema
      this.msg = `Pedido N° ${res.id} realizado com Sucesso!`

      //limpar msg
      setTimeout(() => this.msg = "", 3000)

      //limpar os campos
      this.nome = "";
      this.carne = "";
      this.pao = "";
      this.opcionais = "";

    }
  },
  mounted() {
    this.getIngredientes()
  }
}

</script>

<style scoped>
#burger-form {
  max-width: 400px;
  margin: 0 auto;
}

.input-container {
  display: flex;
  flex-direction: column;
  margin-bottom: 20px;
}

label {
  font-weight: bold;
  margin-bottom: 15px;
  color: #222;
  padding: 5px 10px;
  border-left: 4px solid #FCBA03;
}

input,
select {
  padding: 5px 10px;
  width: 300px;
}

#opcionais-contaier {
  flex-direction: row;
  flex-wrap: wrap;
}

#opcionais-title {
  width: 100%;
}

.checkbox-container {
  display: flex;
  align-items: flex-start;
  width: 50%;
  margin-bottom: 20px;
}

.checkbox-container span,
.checkbox-container input {
  width: auto;
}

.checkbox-container span {
  margin-left: 6px;
  font-weight: bold;
}

.submit-btn {
  background-color: #222;
  color: #FCBA03;
  font-weight: bold;
  border: 2px solid #222;
  padding: 10px;
  font-size: 16px;
  margin: 0 auto;
  cursor: pointer;
  transition: .5s;
}

.submit-btn:hover {
  background-color: transparent;
  color: #222;
}

.error {
  padding: 5px;
}
</style>