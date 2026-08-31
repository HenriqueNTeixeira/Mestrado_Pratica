# Decisões do projeto

Registro das decisões técnicas e metodológicas da parte prática da dissertação,
com a justificativa de cada uma. Atualizar sempre que uma pendência for fechada.

---

## Ambiente

| Item | Valor | Justificativa |
|---|---|---|
| ROS | Noetic | imagem `lar-gazebo:noetic`, já validada |
| Gazebo | Classic 11.11 | vem com a imagem do `lar_gazebo` |
| Build | `catkin build` | a imagem foi construída com catkin_tools; misturar com `catkin_make` quebra o build space |
| Workspace | host montado em `/ws` via docker compose | arquivos ficam no host, versionáveis; container é descartável |
| Mapa | `lab_robotica_06mai2019` (2,5 cm/célula, 1024x1024) | oficial do LAR, resolução mais fina que o mapa próprio via GMapping (5 cm) |
| Área livre total | 84,19 m² | calculado sobre o `.pgm` oficial |
| Área navegável | 37,13 m² (folga de 0,35 m ≈ raio do Husky) | erosão binária do espaço livre |
| Extensão do espaço livre | x: -12,79 a +6,84 m / y: -8,79 a +1,99 m | frame `map` |

---

## Sensor

| Item | Valor | Fonte |
|---|---|---|
| Modelo | SICK LMS1xx | Carvalho et al. (2023), Fig. 13 |
| Nº de feixes | 541 | extraído das bags reais |
| FOV | 270° (-135° a +135°) | extraído das bags reais |
| Resolução angular | 0,5°/feixe | derivado de FOV / feixes |
| Alcance | 0,01 – 20 m | extraído das bags reais |

---

## Geração da base sintética

| Item | Valor | Fonte / justificativa |
|---|---|---|
| Espaçamento entre poses | 0,25 m (a validar) | Zhao et al. (MoLi-PoseNet): intervalo ≈ acurácia esperada |
| Intervalo angular | 10° (36 orientações por posição) | Zhao et al. |
| Tamanho estimado do dataset | ~21.400 scans (0,25 m x 36 orient.) | calculado sobre a área navegável |
| Ruído de treino | uniforme ±0,05–0,1 m na posição, ±1° na orientação | Zhao et al.: evita que a rede decore as poses discretas da grade |
| Leituras inválidas | preencher com zero | Zhao et al.: espelha como o LiDAR real trata retorno de baixa intensidade |
| Poses válidas | apenas em células livres, com folga do raio do robô | Carvalho et al.: hipóteses em célula ocupada recebem score zero |
| Células desconhecidas no raycasting | **A DECIDIR empiricamente** | comparar histograma de alcances sintéticos vs. bags reais nas duas variantes (desconhecido = livre / = ocupado) |

---

## Rede neural

| Item | Valor | Fonte / justificativa |
|---|---|---|
| T (janela temporal) | Etapa A: T = 1 / Etapa B: T = 10 | reduz risco; a comparação T=1 vs T=10 vira ablação e testa a hipótese central do trabalho |
| K (hipóteses) | 10 | Filotheou (CBGL) usa k=10; ~77% das hipóteses bottom-k ficam abaixo de 0,5 m |
| Codificação angular | sin θ / cos θ | Zhou et al. (2019): evita a descontinuidade em ±180° |
| Split validação | 25% do treino | Zhao et al. |
| Otimizador | a definir (Adam como candidato) | IR-MCL usa Adam; RPROP permanece apenas dentro do PM |

---

## Refinamento e avaliação

| Item | Valor | Fonte |
|---|---|---|
| L_c do Perfect Match | 1 m | Pinto et al. (2013): garante 0,5 ≤ E_i ≤ 1 quando d_i ≥ 1 m |
| Otimizador do PM | RPROP | Lauer et al. (2006), ~10 iterações |
| Pré-computação para o PM | mapa de distância + gradientes x/y (Sobel) | Lauer et al.; mesmo módulo de processamento de mapa serve ao raycasting |
| **Sucesso — rede isolada** | erro ≤ 12 cm e ≤ 20° | Carvalho et al.: L_dev/A_dev, tolerância do PM medida em 10.000 inicializações |
| **Sucesso — rede + PM** | erro < 10 cm e < 5°, por ≥ 5 iterações consecutivas | Carvalho et al. |
| Ground truth em dados reais | rastreamento por PM | Carvalho et al.: não há ground truth real disponível no laboratório |
| Métricas | erro de posição, erro angular, taxa de sucesso, tempo de inferência | projeto de dissertação, Seção 4.5 |

---

## Comparação experimental principal

**Rede isolada vs. rede + Perfect Match.**
AG, PSO, MCL e AMCL entram apenas na fundamentação teórica e nos trabalhos
relacionados, não como baseline experimental.

Enquadramento: o sistema de Carvalho et al. (2023) é `meta-heurística multi-hipótese → PM`.
Esta dissertação substitui a meta-heurística por uma rede neural, mantendo a
mesma arquitetura de dois estágios.

---

## Convenções de escrita

- Terminologia fixa: "LiDAR", "localização global", "sequestro do robô", "hipóteses de pose"
- Evitar: descrever a abordagem como MCL ou filtro de partículas; chamar dados de raycasting de "dados reais"
- Citações ABNT autor-data, vírgula decimal

---

## Pendências com os orientadores

1. As bags de 07/05/2019 são do Husky físico ou de uma sessão em simulação?
   (evidência forte de que são reais: padrão de intensidades e quedas de alcance
   repetidas nos mesmos índices; Carvalho et al. usaram Husky + LMS1xx nesse mesmo laboratório)
2. Autorização para usar o truque do DSOM: adicionar obstáculos aleatórios ao gerar
   as observações de treino, mas alimentar a rede com o mapa limpo, para robustez
   a objetos não mapeados.
3. O código-fonte do Perfect Match do trabalho AG/PSO pode ser reaproveitado?
   (o artigo indica que Carvalho desenvolveu o código dos métodos propostos)

---

## Histórico de decisões em aberto

- Arquitetura final da rede (codificador por leitura + módulo temporal + cabeça MDN)
- Biblioteca de deep learning
- Proporção treino/validação/teste no nível de **poses**, não de scans
- Mapas públicos adicionais para a avaliação de generalização
