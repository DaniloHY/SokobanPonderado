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