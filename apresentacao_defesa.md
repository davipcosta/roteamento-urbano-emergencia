---
marp: true
theme: default
paginate: true
header: 'Defesa Intermediária: Roteamento de Emergência'
footer: 'Davi Costa, Rafael Diniz, Ramon Souza, Thiago Pamplona'
style: |
  section {
    font-size: 26px;
  }
  h1 {
    color: #0056b3;
  }
  h2 {
    color: #0056b3;
    border-bottom: 2px solid #0056b3;
  }
---

# Roteamento Urbano de Emergência em Belém do Pará
### Acompanhamento e Defesa Intermediária

**Equipe:** 
- Davi Costa
- Rafael Diniz
- Ramon Souza
- Thiago Pamplona

**Contexto:** Despacho de ambulâncias saindo do Hospital Unimed Prime para pontos de ocorrência.

---

## 1. Modalidades e Problemas Definidos

Definimos o problema central como o roteamento rápido de veículos de emergência. A partir disso, dividimos nossa avaliação em duas **modalidades (cenários) de restrição**:

1. **Modalidade 1 - Cenários de Curta/Média Distância (ex: UFPA e Aeroporto):**
   - **Problema:** Encontrar o caminho perfeito (ótimo) onde a explosão de memória da árvore de busca não é um impeditivo tão severo.
2. **Modalidade 2 - Cenários de Longa Distância (ex: Icoaraci):**
   - **Problema:** Lidar com a alta complexidade do mapa. Em viagens longas, algoritmos ótimos sobrecarregam a memória. O objetivo passa a ser obter uma rota muito boa em frações de segundo, aceitando um roteamento sub-ótimo.

---

## 2. Representação e Critérios de Avaliação

- **O Espaço de Estados (Grafo):** Malha viária real baixada via OSMnx.
  - **Nós:** Cruzamentos da cidade.
  - **Arestas:** Trechos de ruas conectando cruzamentos.
- **Função de Custo $g(n)$:** Distância física real (em metros) de um cruzamento a outro.
- **Função Heurística $h(n)$:** Fórmula de *Haversine* (distância geográfica em linha reta até o destino). É uma heurística **admissível**, pois a distância nas ruas nunca será menor que o voo de pássaro.
- **Critérios de Avaliação:**
  1. **Custo da Rota:** Metragem total (qualidade do caminho).
  2. **Nós Expandidos:** Impacto em memória e eficiência de varredura.
  3. **Tempo de Execução:** Em segundos.

---

## 3. Justificativa dos Algoritmos

A modelagem como um problema de grafos torna natural o uso de algoritmos de **Busca Heurística**. 

- Como estamos lidando com ruas reais, precisamos de um algoritmo capaz de lidar com custos variáveis de arestas.
- A **Heurística de Haversine** é perfeita para o problema porque guia a busca na direção física do destino final.
- **A* e Gulosa** formam o contraponto ideal: a garantia matemática de otimalidade contra a agressividade de encontrar uma resposta rápida para a ambulância.

---

## 4. Algoritmos Estudados em Sala (Implementados)

Já temos implementados e com execuções 100% funcionais os algoritmos vistos em sala:

- **Busca Gulosa (Greedy BFS):**
  - Guia-se **apenas** pela Heurística $h(n)$.
  - Super rápida e expande pouquíssimos nós.
  - Não avalia o custo real já percorrido, resultando frequentemente em caminhos sub-ótimos.
- **Algoritmo A* (A-Star Clássico):**
  - Soma o custo real com a heurística: $f(n) = g(n) + h(n)$.
  - Garante o caminho mais curto possível.
  - Sofre com altíssima expansão de nós (explosão de memória) em distâncias longas.

---

## 5. Variantes e Estratégias Adicionais

Para combater os pontos fracos dos algoritmos clássicos, implementamos duas variantes robustas:

1. **Weighted A* (A* Ponderado):**
   - Introduz um peso ($w$) na heurística: $f(n) = g(n) + w \times h(n)$.
   - *Como funciona:* Ao multiplicar $h(n)$ por 1.5 ou 2.0, o algoritmo dá mais prioridade a se aproximar do objetivo. Ele perde a garantia matemática do caminho perfeito, mas acelera drasticamente a busca, sendo ideal para a Modalidade 2 (Icoaraci).
2. **A* Bidirecional:**
   - *Como funciona:* Inicia duas buscas simultâneas, uma saindo do Hospital e outra do Local da Ocorrência.
   - O ponto de encontro no meio corta a profundidade máxima da árvore pela metade, economizando recursos consideráveis.

---

## 6. Evidências e Primeiros Resultados

Rodamos os algoritmos para 3 níveis de distância. Aqui estão as evidências iniciais (Resumo):

**Destino: Aeroporto (Médio) ~ 11.7 km**
- **Busca Gulosa:** Apenas 393 nós expandidos, mas custo alto: 19.8 km (Caminho ruim).
- **A* Clássico:** Caminho ótimo de 11.7 km, mas explorou **5.103 nós**!
- **Weighted A* (w=2.0):** 12.3 km (levemente sub-ótimo), mas explorou apenas **375 nós**!

**Destino: Orla de Icoaraci (Longo)**
- A explosão do A* piora (6.586 nós). O Weighted A* e o Bidirecional demonstram ser a solução mais viável para o despacho imediato. A extração no mapa com *Folium* já consegue desenhar essas rotas visualmente.

---

## 7. Planejamento da Comparação

Para a etapa final do trabalho, nosso foco de observação será:

1. **Balanço Otimalidade vs. Custo Computacional (Trade-off):** Avaliar visualmente as rotas para entender quão "pior" o caminho do Weighted A* realmente é na prática.
2. **Avaliação da Árvore de Busca:** Plotar a nuvem de nós expandidos no mapa de Belém para mostrar a diferença geométrica entre o A* Clássico (que varre em formato de "círculo") e o Weighted A* (que varre em formato "elíptico/direcionado").
3. **Melhorias Planejadas:** Considerar o tempo de trânsito como o custo real nas arestas do grafo, saindo do roteamento estático (distância) para um dinâmico.
