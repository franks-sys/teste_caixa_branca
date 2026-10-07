# 🧪 Teste de Caixa Branca — Loja SENAI

| | |
|---|---|
| **Instituição** | SENAI |
| **Curso** | Técnico em Desenvolvimento de Sistemas |
| **Unidade curricular** | SESI CE 356 |
| **Atividade** | Teste de Caixa Branca |
| **Aluno** | Francisco de Paula Souza |
| **Turma** | 3A |
| **Professores** | Robson, Reenye e Wellington |
| **Data** | 02/10/2026 |

> 📝 **Minhas anotações gerais**
>
> _fiz um teste de caixa branca e tive que analisar onde tinha inconsisências_

---

## 1. O que é teste de caixa branca

No teste de caixa branca eu não olho só para o que entra e o que sai: eu abro o código e acompanho o que acontece dentro dele. Passo pelos `if`, pelas comparações e pelos caminhos que o programa pode seguir, para ver se a lógica faz o que deveria. Isso ajuda a achar erros que o uso comum não mostra, principalmente nos **valores-limite** (quando o número está exatamente na borda de uma regra).

### O sistema

Uma página de loja onde o usuário escolhe **produto**, **quantidade**, **cupom** e **frete**, clica em *Calcular pedido* e vê subtotal, desconto, frete e total.

- `index.html`: estrutura da página
- `style.css`: visual
- `script.js`: toda a lógica e os cálculos (onde estavam os erros)

<details>
<summary><b>Código original (script.js)</b></summary>

```js
const precos = {
  notebook: 3000,
  mouse: 80,
  teclado: 150
};

const estoque = {
  notebook: 5,
  mouse: 20,
  teclado: 10
};

const produto = document.getElementById("produto");
const quantidade = document.getElementById("quantidade");
const cupom = document.getElementById("cupom");
const frete = document.getElementById("frete");
const calcular = document.getElementById("calcular");
const resultado = document.getElementById("resultado");

function calcularDesconto(subtotal, codigo) {
  if (codigo === "SENAI10") {
    return subtotal * 0.10;
  }

  if (codigo === "SENAI20" && subtotal >= 1000) {
    return subtotal * 0.20;
  }

  return 0;
}

function calcularFrete(tipo, subtotal) {
  if (tipo === "retirada") {
    return 0;
  }

  if (tipo === "expresso") {
    return 60;
  }

  if (subtotal >= 500) {
    return 0;
  }

  return 30;
}

function finalizarPedido() {
  const produtoSelecionado = produto.value;
  const qtd = Number(quantidade.value);
  const codigo = cupom.value.trim().toUpperCase();

  if (qtd < 0) {
    resultado.innerHTML = "<p>Quantidade inválida.</p>";
    return;
  }

  if (qtd >= estoque[produtoSelecionado]) {
    resultado.innerHTML = "<p>Quantidade indisponível em estoque.</p>";
    return;
  }

  const subtotal = precos[produtoSelecionado] * qtd;
  const desconto = calcularDesconto(subtotal, codigo);
  const valorFrete = calcularFrete(frete.value, subtotal);

  let total = subtotal - desconto + valorFrete;

  if (qtd > 5) {
    total = total - subtotal * 0.05;
  }

  if (total > 3000) {
    total = total * 0.95;
  }

  let mensagem = "Pedido calculado com sucesso.";

  if (total <= 0) {
    mensagem = "Valor do pedido inválido.";
  } else if (total >= 3000) {
    mensagem = "Pedido de alto valor.";
  }

  resultado.innerHTML = `
    <p>${mensagem}</p>
    <p>Subtotal: R$ ${subtotal.toFixed(2)}</p>
    <p>Desconto: R$ ${desconto.toFixed(2)}</p>
    <p>Frete: R$ ${valorFrete.toFixed(2)}</p>
    <p class="total">Total: R$ ${total.toFixed(2)}</p>
  `;
}

calcular.addEventListener("click", finalizarPedido);
```
</details>

---

## 2. Decisões do código

O código só usa `if` (sem `switch`, ternário ou laço). O único operador lógico é o `&&` do cupom SENAI20.

| # | Função | Condição | Verdadeiro / Falso |
|---|---|---|---|
| D1 | `calcularDesconto` | `codigo === "SENAI10"` | 10% / vai para D2 |
| D2 | `calcularDesconto` | `codigo === "SENAI20" && subtotal >= 1000` | 20% / sem desconto |
| D3 | `calcularFrete` | `tipo === "retirada"` | R$ 0 / vai para D4 |
| D4 | `calcularFrete` | `tipo === "expresso"` | R$ 60 / vai para D5 |
| D5 | `calcularFrete` | `subtotal >= 500` | R$ 0 / R$ 30 |
| D6 | `finalizarPedido` | `qtd < 0` | "Quantidade inválida" / vai para D7 |
| D7 | `finalizarPedido` | `qtd >= estoque` | "Indisponível" / segue o cálculo |
| D8 | `finalizarPedido` | `qtd > 5` | aplica 5% / não aplica |
| D9 | `finalizarPedido` | `total > 3000` | 5% extra / não aplica |
| D10 | `finalizarPedido` | `total <= 0` | "Valor inválido" / vai para D11 |
| D11 | `finalizarPedido` | `total >= 3000` | "Alto valor" / "Sucesso" |

---

## 3. Fluxograma geral

```mermaid
flowchart TD
    classDef inicio fill:#1f2937,stroke:#111827,color:#ffffff,font-weight:bold
    classDef dado fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef calc fill:#f3f4f6,stroke:#6b7280,color:#111827
    classDef decisao fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef erro fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef ok fill:#dcfce7,stroke:#16a34a,color:#14532d

    I([▶ Clique em Calcular]):::inicio --> L[/Lê produto, qtd, cupom e frete/]:::dado
    L --> D6{Qtd inválida?}:::decisao
    D6 -- Sim --> E1[Quantidade inválida]:::erro --> F
    D6 -- Não --> D7{Passa do estoque?}:::decisao
    D7 -- Sim --> E2[Indisponível em estoque]:::erro --> F
    D7 -- Não --> C1[Calcula subtotal, desconto e frete]:::calc
    C1 --> D8{Qtd ≥ 5?}:::decisao
    D8 -- Sim --> C2[Soma 5% ao desconto]:::calc --> C3
    D8 -- Não --> C3[Total parcial = subtotal − desconto + frete]:::calc
    C3 --> D9{Total parcial ≥ 3000?}:::decisao
    D9 -- Sim --> C4[Alto valor: 5% extra]:::calc --> M
    D9 -- Não --> M[Define a mensagem]:::calc
    M --> R[/Exibe resultado/]:::ok --> F([⏹ Fim]):::inicio
```

---

## 4. Casos de teste

| ID | Entrada | Condição / Caminho | Resultado esperado |
|---|---|---|---|
| CT01 | Mouse, qtd 0, sem cupom, normal | D6 no limite | "Quantidade inválida" |
| CT02 | Teclado, qtd 10, sem cupom, retirada | D7 no limite | Aceito, total R$ 1.425,00 |
| CT03 | Mouse, qtd 5, sem cupom, retirada | D8 no limite | Desconto R$ 20,00, total R$ 380,00 |
| CT04 | Mouse, qtd 10, SENAI10, retirada | D1 e D8 verdadeiros | Desconto R$ 120,00, total R$ 680,00 |
| CT05 | Notebook, qtd 1, sem cupom, retirada | D9 e D11 com total = 3000 | Alto valor, total R$ 2.850,00 |
| CT06 | Notebook, qtd 1, sem cupom, expresso | D9 verdadeiro e depois D11 | Alto valor, total R$ 2.907,00 |

## 5. Resultados

| Teste | Obtido (original) | Situação | Obtido (corrigido) | Situação |
|---|---|---|---|---|
| CT01 | "Sucesso", total R$ 30,00 | ❌ | "Quantidade inválida." | ✅ |
| CT02 | "Indisponível em estoque" | ❌ | Total R$ 1.425,00 | ✅ |
| CT03 | Total R$ 400,00, sem desconto | ❌ | Desconto R$ 20,00, total R$ 380,00 | ✅ |
| CT04 | Desconto R$ 80,00, total R$ 680,00 | ❌ | Desconto R$ 120,00, total R$ 680,00 | ✅ |
| CT05 | "Alto valor", total R$ 3.000,00 | ❌ | "Alto valor", total R$ 2.850,00 | ✅ |
| CT06 | "Sucesso", total R$ 2.907,00 | ❌ | "Alto valor", total R$ 2.907,00 | ✅ |

---

## 6. Análise dos erros

### 🔴 ERRO 1 · Fácil · valores-limite

**Trecho:** `if (qtd < 0)`
**Esperado:** quantidade 0, vazia ou decimal deve ser recusada.
**Teste:** mouse, qtd 0, sem cupom, frete normal.
**Caminho:** D6 `0 < 0` falso → D7 `0 >= 20` falso → subtotal 0 → D5 `0 >= 500` falso → frete 30 → total 30 → "Sucesso".
**Obtido:** pedido aceito com frete de R$ 30,00 para zero itens.
**Erro:** o limite está errado; o zero é o primeiro valor inválido e passa.
**Correção:**
```js
if (!Number.isInteger(qtd) || qtd <= 0) {
```
**Depois:** "Quantidade inválida."

```mermaid
flowchart LR
    classDef dado fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef decisao fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef calc fill:#f3f4f6,stroke:#6b7280,color:#111827
    classDef erro fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef ok fill:#dcfce7,stroke:#16a34a,color:#14532d

    A[/qtd = 0 · mouse · frete normal/]:::dado --> B{"qtd < 0 ?<br/>0 < 0"}:::decisao
    B -- "Falso ⚠ passou" --> C[subtotal = 0<br/>frete = 30]:::calc
    C --> D[total = 30]:::erro
    D --> E[Exibe: Sucesso]:::erro
    B -. "Esperado: Verdadeiro" .-> F[Quantidade inválida]:::ok
```

> 📝 **Minhas anotações — Erro 1:** 1 = qtd = 0 com frete ainda funciona

---

### 🔴 ERRO 2 · Fácil · valores-limite

**Trecho:** `if (qtd >= estoque[produtoSelecionado])`
**Esperado:** se há 10 teclados, dá para comprar os 10.
**Teste:** teclado (estoque 10), qtd 10, sem cupom, retirada.
**Caminho:** D6 falso → D7 `10 >= 10` verdadeiro → mostra "indisponível" e dá `return`.
**Obtido:** "Quantidade indisponível em estoque."
**Erro:** o `>=` bloqueia o pedido que usa todo o estoque.
**Correção:**
```js
if (qtd > estoque[produtoSelecionado]) {
```
**Depois:** pedido aceito, total R$ 1.425,00. Com qtd 11 continua bloqueando.

```mermaid
flowchart LR
    classDef dado fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef decisao fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef erro fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef ok fill:#dcfce7,stroke:#16a34a,color:#14532d

    A[/qtd = 10 · estoque = 10/]:::dado --> B{"qtd >= estoque ?<br/>10 >= 10"}:::decisao
    B -- "Verdadeiro ⚠" --> C[Indisponível em estoque]:::erro --> D([return]):::erro
    B -. "Esperado: Falso" .-> E[Segue o cálculo → total 1425]:::ok
```

> 📝 **Minhas anotações — Erro 2:** 2 = qtd tem 10 no estoque não da pra comprar

---

### 🟠 ERRO 3 · Médio · valores-limite e cobertura de decisões

**Trecho:** `if (qtd > 5)`
**Esperado:** 5% de desconto a partir de 5 unidades.
**Teste:** mouse, qtd 5, sem cupom, retirada.
**Caminho:** subtotal 400 → desconto 0 → frete 0 → total 400 → D8 `5 > 5` falso → sem desconto.
**Obtido:** total R$ 400,00.
**Erro:** o `>` deixa de fora a quantidade que abre a faixa.
**Correção:**
```js
const QTD_MINIMA_DESCONTO = 5;
if (qtd >= QTD_MINIMA_DESCONTO) {
  desconto += subtotal * 0.05;
}
```
**Depois:** total R$ 380,00. Com 4 unidades continua sem desconto.

```mermaid
flowchart LR
    classDef dado fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef decisao fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef calc fill:#f3f4f6,stroke:#6b7280,color:#111827
    classDef erro fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef ok fill:#dcfce7,stroke:#16a34a,color:#14532d

    A[/qtd = 5 · mouse · retirada/]:::dado --> B[subtotal = 400<br/>total = 400]:::calc
    B --> C{"qtd > 5 ?<br/>5 > 5"}:::decisao
    C -- "Falso ⚠" --> D[Total R$ 400,00]:::erro
    C -. "Esperado: aplicar" .-> E[400 − 20 = R$ 380,00]:::ok
```

> 📝 **Minhas anotações — Erro 3:** 3 = qtd tinha ser descontada o 0.05% quando e maior ou igual a 5

---

### 🟠 ERRO 4 · Médio · rastreamento de variáveis

**Trecho:**
```js
const desconto = calcularDesconto(subtotal, codigo);
let total = subtotal - desconto + valorFrete;
if (qtd > 5) {
  total = total - subtotal * 0.05;
}
```
**Esperado:** o desconto exibido deve somar cupom + quantidade, para a conta da tela fechar.
**Teste:** mouse, qtd 10, SENAI10, retirada.
**Caminho:** subtotal 800 → D1 verdadeiro, `desconto = 80` → total 720 → D8 verdadeiro, `total = 680` → `desconto` continua 80.
**Obtido:** desconto R$ 80,00 e total R$ 680,00 (800 − 80 = 720, não fecha).
**Erro:** o desconto por quantidade mexe direto em `total` e nunca entra na variável `desconto`.
**Correção:**
```js
let desconto = calcularDesconto(subtotal, codigo);
if (qtd >= QTD_MINIMA_DESCONTO) {
  desconto += subtotal * 0.05;
}
const totalParcial = subtotal - desconto + valorFrete;
```
**Depois:** desconto R$ 120,00 e total R$ 680,00.

```mermaid
flowchart LR
    classDef dado fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef decisao fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef calc fill:#f3f4f6,stroke:#6b7280,color:#111827
    classDef erro fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef ok fill:#dcfce7,stroke:#16a34a,color:#14532d

    A[/mouse · qtd 10 · SENAI10/]:::dado --> B[subtotal = 800<br/>desconto = 80]:::calc
    B --> C[total = 720]:::calc
    C --> D{"qtd > 5 ?"}:::decisao
    D -- Sim --> E["total = 680<br/>desconto segue 80 ⚠"]:::erro
    E --> F[Tela: 800 − 80 ≠ 680]:::erro
    D -. "Correção" .-> G[desconto = 80 + 40 = 120]:::ok
```

> 📝 **Minhas anotações — Erro 4:** 4 = era para somar o desconto por quantidade com o desconto do cupom e mostrar o desconto correto pro usuário porem aparece so o desconto do cupom

---

### 🔴 ERRO 5 · Difícil · condições dependentes

**Trecho:**
```js
if (total > 3000) { total = total * 0.95; }
...
} else if (total >= 3000) { mensagem = "Pedido de alto valor."; }
```
**Esperado:** o mesmo limite para o desconto extra e para a mensagem (adotei "a partir de R$ 3.000").
**Teste:** notebook, qtd 1, sem cupom, retirada (total = 3000).
**Caminho:** D9 `3000 > 3000` falso (sem 5%) → D10 falso → D11 `3000 >= 3000` verdadeiro → "Alto valor".
**Obtido:** "Pedido de alto valor." com total R$ 3.000,00 (sem o desconto).
**Erro:** duas decisões testam o mesmo limite com operadores diferentes (`>` e `>=`).
**Correção:**
```js
const LIMITE_ALTO_VALOR = 3000;
const altoValor = totalParcial >= LIMITE_ALTO_VALOR;
```
**Depois:** "Pedido de alto valor." com total R$ 2.850,00.

```mermaid
flowchart LR
    classDef dado fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef decisao fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef calc fill:#f3f4f6,stroke:#6b7280,color:#111827
    classDef erro fill:#fee2e2,stroke:#dc2626,color:#7f1d1d

    A[/notebook · qtd 1 · retirada/]:::dado --> B[total = 3000]:::calc
    B --> C{"total > 3000 ?<br/>3000 > 3000"}:::decisao
    C -- "Falso: sem 5%" --> D{"total >= 3000 ?<br/>3000 >= 3000"}:::decisao
    D -- "Verdadeiro" --> E["Alto valor, mas R$ 3.000,00 ⚠<br/>(regras divergentes)"]:::erro
```

> 📝 **Minhas anotações — Erro 5:** 5 = quando coloca 3000 ele exibe que e um pedido de alto valor porem não recebe o desconto desejado pois pedido de alto valor > 3000

---

### 🔴 ERRO 6 · Difícil · análise de caminhos

**Trecho:** os mesmos `if (total > 3000)` e `else if (total >= 3000)` do erro 5.
**Esperado:** a classificação de alto valor olha o valor *antes* do desconto que ele mesmo gera.
**Teste:** notebook, qtd 1, sem cupom, frete expresso.
**Caminho:** subtotal 3000 → frete 60 → total 3060 → D9 `3060 > 3000` verdadeiro → total vira 2907 → D11 `2907 >= 3000` falso → "Sucesso".
**Obtido:** "Pedido calculado com sucesso." com total R$ 2.907,00.
**Erro:** ordem das operações. O desconto reduz `total` e logo depois a mesma variável classifica o pedido. Vale para qualquer pedido entre R$ 3.000,00 e ~R$ 3.157,89.
**Correção:** decidir se é alto valor *antes* de alterar o total.
```js
const altoValor = totalParcial >= LIMITE_ALTO_VALOR;
const descontoAltoValor = altoValor ? totalParcial * 0.05 : 0;
const total = totalParcial - descontoAltoValor;

if (total <= 0) {
  mensagem = "Valor do pedido inválido.";
} else if (altoValor) {
  mensagem = "Pedido de alto valor.";
}
```
**Depois:** "Pedido de alto valor." com total R$ 2.907,00.

```mermaid
flowchart LR
    classDef dado fill:#dbeafe,stroke:#2563eb,color:#1e3a8a
    classDef decisao fill:#fef3c7,stroke:#d97706,color:#78350f
    classDef calc fill:#f3f4f6,stroke:#6b7280,color:#111827
    classDef erro fill:#fee2e2,stroke:#dc2626,color:#7f1d1d
    classDef ok fill:#dcfce7,stroke:#16a34a,color:#14532d

    A[/notebook · qtd 1 · expresso/]:::dado --> B[subtotal 3000 + frete 60<br/>total = 3060]:::calc
    B --> C{"total > 3000 ?<br/>3060 > 3000"}:::decisao
    C -- Sim --> D[total = 3060 × 0,95 = 2907]:::calc
    D --> E{"total >= 3000 ?<br/>2907 >= 3000"}:::decisao
    E -- "Falso ⚠" --> F[Mensagem: Sucesso]:::erro
    E -. "Esperado" .-> G[Mensagem: Alto valor]:::ok
```

> 📝 **Minhas anotações — Erro 6:** 6 = produto quando recebe frete ele se torna um pedido de alto valor, porem no sistema não é informado um pedido de alto valor apenas "pedido calculado com sucesso"

---

## 7. Antes e depois

| Erro | Nível | Antes | Depois |
|---|---|---|---|
| 1 | Fácil | Aceita qtd 0 e cobra frete | Recusa 0, vazio e decimais |
| 2 | Fácil | Bloqueia pedido igual ao estoque | Aceita até o estoque |
| 3 | Médio | 5 unidades sem desconto | Desconto a partir de 5 |
| 4 | Médio | Desconto da tela não fecha com o total | Desconto soma cupom + quantidade |
| 5 | Difícil | `>` e `>=` divergem em 3000 | Mesmo limite nas duas decisões |
| 6 | Difícil | Perde "alto valor" após o desconto | Classificação feita antes do desconto |

**Cobertura:** os seis casos passam por D1, D6, D7, D8, D9, D10 (lado falso), D11 (os dois lados) e parte da condição composta D2. Ainda vale testar SENAI20 acima e abaixo de R$ 1.000, frete normal acima e abaixo de R$ 500 e cupom inexistente.

### Código corrigido (`finalizarPedido`)

O arquivo completo está em [`scriptnovo.js`](./scriptnovo.js).

```js
const LIMITE_ALTO_VALOR = 3000;
const QTD_MINIMA_DESCONTO = 5;

function finalizarPedido() {
  const produtoSelecionado = produto.value;
  const qtd = Number(quantidade.value);
  const codigo = cupom.value.trim().toUpperCase();

  if (!Number.isInteger(qtd) || qtd <= 0) {
    resultado.innerHTML = "<p>Quantidade inválida.</p>";
    return;
  }

  if (qtd > estoque[produtoSelecionado]) {
    resultado.innerHTML = "<p>Quantidade indisponível em estoque.</p>";
    return;
  }

  const subtotal = precos[produtoSelecionado] * qtd;
  let desconto = calcularDesconto(subtotal, codigo);

  if (qtd >= QTD_MINIMA_DESCONTO) {
    desconto += subtotal * 0.05;
  }

  const valorFrete = calcularFrete(frete.value, subtotal);
  const totalParcial = subtotal - desconto + valorFrete;

  const altoValor = totalParcial >= LIMITE_ALTO_VALOR;
  const descontoAltoValor = altoValor ? totalParcial * 0.05 : 0;
  const total = totalParcial - descontoAltoValor;

  let mensagem = "Pedido calculado com sucesso.";

  if (total <= 0) {
    mensagem = "Valor do pedido inválido.";
  } else if (altoValor) {
    mensagem = "Pedido de alto valor.";
  }

  resultado.innerHTML = `
    <p>${mensagem}</p>
    <p>Subtotal: R$ ${subtotal.toFixed(2)}</p>
    <p>Desconto: R$ ${desconto.toFixed(2)}</p>
    ${altoValor ? `<p>Desconto alto valor: R$ ${descontoAltoValor.toFixed(2)}</p>` : ""}
    <p>Frete: R$ ${valorFrete.toFixed(2)}</p>
    <p class="total">Total: R$ ${total.toFixed(2)}</p>
  `;
}
```

---

## 8. Conclusão
>
> - Um código pode rodar sem travar e mesmo assim estar errado: os 6 erros sempre mostravam um resultado com cara de normal.
> - Quatro erros foram de valor-limite (`<`, `>=`, `>`), um foi de inconsistência entre o desconto mostrado e o aplicado, e um foi de ordem das operações.
> - O fluxograma ajudou a enxergar o erro 6, em que uma decisão muda o valor usado pela decisão seguinte.
> - Todos os 6 casos falharam antes da correção e passaram depois.
