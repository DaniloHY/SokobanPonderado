# Sokoban Ponderado - Fluxograma Principal

```mermaid
graph TD
    %% Definição de estilos
    classDef estado fill:#e1f5fe,stroke:#01579b,stroke-width:2px
    classDef decisao fill:#fff3e0,stroke:#ff6f00,stroke-width:2px
    classDef acao fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px
    classDef objetivo fill:#c8e6c9,stroke:#1b5e20,stroke-width:3px
    classDef fallback fill:#ffebee,stroke:#b71c1c,stroke-width:2px

    %% Estado Inicial
    Inicio(["🏁 ESTADO INICIAL<br/>Agente: (x,y)<br/>Caixas: [(x1,y1,p1), ...]<br/>Alvos: [(a1,b1), ...]<br/>Barreiras: {...}"]) 
    Inicio --> VerificarObj

    %% Verificação de objetivo
    VerificarObj{Verificar se é<br/>ESTADO OBJETIVO} 
    VerificarObj -->|SIM| Objetivo["✅ OBJETIVO ALCANÇADO!<br/>Todas as caixas em alvos"] 
    VerificarObj -->|NÃO| GerarAcoes

    %% Geração de ações
    GerarAcoes["GERAR AÇÕES POSSÍVEIS<br/>Para cada direção (cima, baixo, esq, dir)"] 
    GerarAcoes --> DirecaoLoop

    %% Loop de direções
    DirecaoLoop{Direções a<br/>processar?} 
    DirecaoLoop -->|SIM| ProxDirecao["Próxima direção: (dx, dy)"]
    DirecaoLoop -->|NÃO| Retornar["Retornar lista de<br/>estados sucessores"]
    Retornar --> Fim(["FIM DA EXPANSÃO"])

    %% Processamento de cada direção
    ProxDirecao --> CalcNovaPos["Calcular nova posição do agente:<br/>nova_x = x_agente + dx<br/>nova_y = y_agente + dy"]
    CalcNovaPos --> ValidaPos{Posição válida?<br/>(dentro do grid e<br/>não é barreira)}

    ValidaPos -->|NÃO| DirecaoLoop
    ValidaPos -->|SIM| VerificaCaixa{Posição contém<br/>uma caixa?}

    %% Subgrafo para movimento com caixa
    VerificaCaixa -->|SIM, tem caixa| EncontraCaixa["Encontrar caixa na posição<br/>caixa = (nova_x, nova_y, peso)"]
    EncontraCaixa --> CalcNovaPosCaixa["Calcular nova posição da caixa:<br/>caixa_x' = nova_x + dx<br/>caixa_y' = nova_y + dy"]
    CalcNovaPosCaixa --> ValidaPosCaixa{Posição da caixa<br/>é válida?<br/>(dentro do grid,<br/>não é barreira,<br/>não tem outra caixa)}
    
    ValidaPosCaixa -->|NÃO| DirecaoLoop
    ValidaPosCaixa -->|SIM| CriaEstadoEmpurrar["CRIAR NOVO ESTADO (EMPURRAR)<br/>1. Agente vai para (nova_x, nova_y)<br/>2. Caixa vai para (caixa_x', caixa_y')<br/>3. Demais caixas permanecem<br/>4. Custo = 1 + peso_caixa"]
    
    CriaEstadoEmpurrar --> AddSucessor["Adicionar sucessor à lista<br/>tipo: 'empurrar', peso: peso_caixa"]
    AddSucessor --> DirecaoLoop

    %% Subgrafo para movimento simples
    VerificaCaixa -->|NÃO, não tem caixa| CriaEstadoMover["CRIAR NOVO ESTADO (MOVER)<br/>1. Agente vai para (nova_x, nova_y)<br/>2. Todas as caixas permanecem<br/>3. Custo = 1"]
    CriaEstadoMover --> AddSucessorMover["Adicionar sucessor à lista<br/>tipo: 'mover', peso: 0"]
    AddSucessorMover --> DirecaoLoop

    %% Subgrafo de busca
    subgraph Busca ["ALGORITMOS DE BUSCA"]
        EscolhaAlg{Escolha do<br/>Algoritmo}
        
        EscolhaAlg --> Dijkstra["DIJKSTRA<br/>Prioriza: menor g(n)<br/>(custo acumulado)"]
        EscolhaAlg --> Ganancioso["GANANCIOSO<br/>Prioriza: menor h(n)<br/>(heurística)"]
        EscolhaAlg --> AEstrela["A*<br/>Prioriza: menor f(n) = g(n) + h(n)"]
        
        Dijkstra --> Expande["Expandir estado<br/>com menor custo"]
        Ganancioso --> Expande
        AEstrela --> Expande
        
        Expande --> VerificaVisitado{Já visitado<br/>com custo menor?}
        VerificaVisitado -->|SIM| Ignora["Ignora estado"]
        VerificaVisitado -->|NÃO| MarcaVisitado["Marca como visitado<br/>e adiciona à fronteira"]
        
        MarcaVisitado --> GeraSucessoresBusca["Gerar sucessores<br/>(usando função acima)"]
        GeraSucessoresBusca --> AtualizaFronteira["Atualiza fronteira<br/>com novos sucessores"]
        AtualizaFronteira --> ProximoEstado["Pega próximo estado<br/>da fronteira"]
        ProximoEstado --> VerificaObjBusca{É estado<br/>objetivo?}
        VerificaObjBusca -->|SIM| Solucao["🎉 SOLUÇÃO ENCONTRADA!"]
        VerificaObjBusca -->|NÃO| Expande
    end

    %% Conexões entre busca e expansão
    GeraSucessoresBusca -.-> GerarAcoes

    %% Exemplo de transição visual
    subgraph Exemplo ["EXEMPLO DE TRANSIÇÃO"]
        E1["Estado 1<br/>🤖 ⚪️ ⚪️<br/>⚪️ 📦(2) ⚪️<br/>⚪️ ⚪️ 🎯"]
        A1["Ação: Mover Direita<br/>Custo: 1"]
        E2["Estado 2<br/>⚪️ 🤖 ⚪️<br/>⚪️ 📦(2) ⚪️<br/>⚪️ ⚪️ 🎯"]
        A2["Ação: Mover Baixo<br/>Custo: 1"]
        E3["Estado 3<br/>⚪️ ⚪️ ⚪️<br/>⚪️ 🤖 📦(2)<br/>⚪️ ⚪️ 🎯"]
        A3["Ação: Empurrar Direita<br/>Custo: 1+2=3"]
        E4["Estado 4 (OBJETIVO)<br/>⚪️ ⚪️ ⚪️<br/>⚪️ ⚪️ 🤖<br/>⚪️ ⚪️ ✅"]
        
        E1 --> A1 --> E2
        E2 --> A2 --> E3
        E3 --> A3 --> E4
    end

    %% Aplicando estilos
    class Inicio,E1,E2,E3 estado
    class Objetivo,E4 objetivo
    class VerificarObj,DirecaoLoop,VerificaVisitado decisao
    class CriaEstadoMover,CriaEstadoEmpurrar acao
    class Solucao objetivo

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