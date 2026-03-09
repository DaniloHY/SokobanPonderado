## 🎨 1. Explicação dos Estilos (Cores e Significado)
O uso de cores facilita a identificação visual rápida de cada elemento no diagrama de fluxo.

| Classe | Cor | Significado | Exemplo |
| :--- | :--- | :--- | :--- |
| **estado** | 🟦 Azul claro | Representa um estado do jogo | Estado inicial, intermediários |
| **decisao** | 🟧 Laranja claro | Pontos de decisão/ramificação | Verificações "if/else" |
| **acao** | 🟩 Verde claro | Ações realizadas | Mover, empurrar, criar estado |
| **objetivo** | 🌲 Verde escuro | Estados finais importantes | Objetivo alcançado, solução |

---

## 🤖 2. Representação do Estado
Um **estado** é definido por quatro informações essenciais que descrevem o mundo em um determinado momento.

### Estrutura de Dados
* **Agente:** `(x, y)` — Posição atual do robô.
* **Caixas:** `[(x1, y1, p1), ...]` — Lista de caixas com coordenadas e peso.
* **Alvos:** `[(a1, b1), ...]` — Posições onde as caixas devem ser entregues.
* **Barreiras:** `{...}` — Conjunto de posições bloqueadas.

### Visualização do Grid
Cada estado é representado visualmente por um mini-grid:
* 🤖 = Agente
* 📦(n) = Caixa com peso *n*
* 🎯 = Alvo
* ⚪️ = Espaço vazio
* ✅ = Caixa posicionada corretamente no alvo

---

## ⚙️ 3. Fluxo Lógico e Decisões

### Verificação Inicial
Antes de qualquer ação, o algoritmo verifica se o problema já foi resolvido.
1.  **Objetivo atingido?**
    * **SIM:** Fluxo segue para o estado **Objetivo** (Fim).
    * **NÃO:** Gera-se a lista de ações possíveis para continuar a busca.

### Cadeia de Verificações de Movimento
Para cada direção (Norte, Sul, Leste, Oeste), o agente segue esta ordem lógica:
1.  **Calcula nova posição:** Onde o agente tentaria ir?
2.  **Posição válida?** Está dentro do grid e não é uma barreira?
    * *Se NÃO:* Descarta e tenta a próxima direção.
    * *Se SIM:* Verifica se há uma caixa no local.

---

## 🔄 4. Diferenças de Movimentação

| Aspecto | Movimento Simples | Empurrar Caixa |
| :--- | :--- | :--- |
| **Verificações** | Só posição do agente | Posição do agente + posição da caixa |
| **Custo** | $1$ (fixo) | $1 + \text{peso da caixa}$ (variável) |
| **Mudanças** | Só agente se move | Agente e caixa mudam de posição |

---

## 🧠 5. Os Três Algoritmos de Busca

| Algoritmo | Prioriza | Fórmula | Comportamento |
| :--- | :--- | :--- | :--- |
| **Dijkstra** | Custo acumulado | $g(n)$ | Explora como onda; garante o caminho mais barato. |
| **Ganancioso** | Futuro estimado | $h(n)$ | Vai direto ao objetivo; rápido, mas pode ser arriscado. |
| **A\*** | Combinado | $g(n) + h(n)$ | Balanceado; encontra o caminho ótimo de forma eficiente. |

---

## 🏗️ 6. Organização em Fases (Subgrafos)
Para manter o projeto organizado, o fluxo é dividido em três grandes blocos:

1.  **ENTRADA:** Leitura do arquivo de cenário e configuração inicial.
2.  **PROCESSO:** O loop de busca, verificações de colisões e expansão de estados.
3.  **SAÍDA:** Geração do caminho final e apresentação dos resultados.